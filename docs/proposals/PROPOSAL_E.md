# Proposal E

Cherry Pick

## Description

This proposal combines ideas from proposals A, B, and D to create a solution that provides an ability to use it on existing registries, while giving a value add as registries upgrade in the future.

It consists of the following changes:

- It adds a new media type for future use cases.
- It defines a "refers" field on both the new media type and existing image manifests.
- It defines annotations for filtering artifacts.
- It defines both an API and a digest tag syntax for querying referrers.

The result is the following upgrade path:

- Registries without new API or media type support can use the image manifest plus digest tags to create artifacts that refer to other manifests.
- As registries are upgraded, the new API can be used for more efficient queries and less tag clutter.
  The extended Image manifest would still be used for portability to older registries.
- When enough registries support the new media types, a transition can happen to those media types for artifacts.

## Links

| Description      | Link                        |
| ---------------- | --------------------------- |
| Proposal A       | [View](./PROPOSAL_A.md)     |
| Proposal B       | [View](./PROPOSAL_B.md)     |
| Proposal D       | [View](./PROPOSAL_D.md)     |

## Modifications

### JSON Schema

#### Annotations

These annotations would be added for artifacts:

- `org.opencontainers.artifact.type`: type of artifact (sig, sbom, etc)
- `org.opencontainers.artifact.description`: human readable description for the artifact
- `org.opencontainers.artifact.created`: creation time for a manifest

Additional annotations should be considered for filtering various artifact types, e.g. signature public key hash, attestation type, and sbom schema.

These annotations should be considered from the current config schema:

- `org.opencontainers.platform.architecture`: CPU architecture for binaries
- `org.opencontainers.platform.os`: operating system for binaries
- `org.opencontainers.platform.os.version`: operating system version for binaries
- `org.opencontainers.platform.variant`: variant of the CPU architecture for binaries

Existing `org.opencontainers.image.*` annotations should be reviewed to consider more generic names (e.g. replacing `image` with `manifest` or `artifact`).

#### Artifact Spec

Create a new artifact media type to support future use cases where a separate config blob, and an ordered list of blobs, may not match an artifact's use case:

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.artifact.manifest.v1+json",
  "blobs": [
    // optional list, ordering is not enforced by the spec
    // but may be required for specific artifact types
  ],
  "refers": { // optional
    // include refers here
  },
  "annotations": [ // optional
    // annotations for this artifact here
  ]
}
```

#### Image Spec

Extend the Image Manifest with a refers field (existing registries should ignore this per OCI's extensibility requirement):

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": { ... },
  "layers": [ ... ],
  "refers": { // new field
    "mediaType": "application/vnd.oci.image.manifest.v1+json", // any manifest media type
    "size": 1234,
    "digest": "sha256:a1a1a1...",
    "annotations": []
  },
  "annotations": [
    // annotations for this artifact here
  ]
}
```

### Registry HTTP API

#### Referrers API

Add the following `referrers` API (`<ref>` is the digest in the `refers` schema field):

Current working group testing API:

```text
GET /v2/<name>/_oci/artifacts/referrers?digest=<ref>
```

Proposed API for distribution-spec:

```text
GET /v2/<name>/referrers/<ref>
```

The response is an Index of descriptors:

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1234,
      "digest": "sha256:a1a1a1...",
      "annotations": [
        // annotations pulled up from manifest
        "org.opencontainers.artifact.type": "example/icecream",
        "org.opencontainers.artifact.created": "2022-01-01T14:42:55Z",
        "org.example.icecream.flavor": "chocolate"
      ]
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 1234,
      "digest": "sha256:a2a2a2...",
      "annotations": [
        "org.opencontainers.artifact.type": "example/icecream",
        "org.opencontainers.artifact.created": "2022-01-01T15:24:30Z",
        "org.example.icecream.flavor": "vanilla"
      ]
    }
  ],
  "annotations": [
    // reserved for future use
  ]
}
```

If a registry does not implement the `referrers` API, it MUST return a 404.
If a query results in no referrers found, an empty manifest list MUST be returned.

#### Ordering and Pagination

- The registry MUST order the results in a way that allows pagination.
- Adding the `n` query parameter is used to limit the number of entries per index returned.
- The registry SHOULD return fewer than `n` results when the generated Index would exceed the recommended maximum manifest size.
- The `Link` HTTP header is included in the response when additional results are available and is set to the URL for the next page of results.

#### Registry Upgrade Expectations

When upgrading existing registries, the following manifests MUST be scanned for a `refers` field:

1. Any existing `application/vnd.oci.image.manifest.v1+json` manifest referenced by a [digest tag](#digest-tags)
2. Any newly uploaded `application/vnd.oci.image.manifest.v1+json` and `application/vnd.oci.artifact.manifest.v1+json` manifests

Clients SHALL NOT expect manifests uploaded before the [referrers API](#referrers-api) is available on that registry, without using a [digest tag](#digest-tags), will be included in future API responses.

#### Digest Tags

For registries that do not support the `referrers` API, a tag MUST be pushed for any manifest containing a `refers` descriptor with the following syntax:

```text
ref.<alg>-<ref>.<hash>.<type>
```

- E.g. `registry.example.org/project:ref.sha256-0000000000000000000000000000000000000000000000000000000000000000.0404040404040404.sbom`
- `<alg>`: the digest algorithm
- `<ref>`: the digest from the `refers` field (limit of 64 characters)
- `<hash>`: the digest of this artifact (limit of 16 characters)
- `<type>`: type of artifact for filtering (limit of 5 characters)
- Querying for referrers requires the client to get the tag listing, and filter for matching `<alg>`, `<ref>`, and `<type>` entries.
- Adding a `<hash>` of the artifact allows multiple artifacts of the same type to exist with little risk of collision or race conditions.
- Periodic garbage collection may be performed by clients pushing new referrers, deleting stale referrers that have been replaced with newer versions, and tags that no longer point to an accessible manifest.
- Clients can verify the registry does not support the `referrers` API by querying the API and checking for a 404.

## Requirements

### Filtering

1. As a user, I want to query a registry for a stored artifact by its digest or tag.
   - Yes, artifacts may be pushed and pulled by tag.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
   - Yes, queries to the API work with a digest, and registries without the API can be queried by listing the tags and searching for the appropriate tag digest.
1. As a user, I want to query a registry for all stored artifacts of a particular type that reference a given artifact by its digest or tag.
   - Yes, digest tags include a type, and the API returns a descriptor list containing annotations that can be filtered by the client.
1. As a user, I want to query a registry for all stored artifacts based on annotations that reference a given artifact by its digest or tag.
   - Partial, registries that add the API returns a descriptor list containing annotations that can be filtered by the client.
1. As a user, I want to fetch the most up-to-date artifact, collection of artifacts, or application.
   - Partial, registries that add the API returns a descriptor list containing annotations that can be filtered by the client.
1. As an artifact producer, I want to reduce the number of tags that reference an artifact.
   - Yes, registries that add the API can associate artifacts with manifests without pushing digest tags.

### Backwards Compatibility

1. As a user, I want to be sure that existing container runtimes are not affected by any other type of registry artifact.
   - Yes, pulling existing images is not impacted by these changes.
1. As a user, I want to move container images to and from registries that do not support reference types.
   - Yes, if Image manifests are used, artifacts can be pushed both to and from OCI compatible registries without changing the digest of the artifact.
     Digest tags are used on registries without the new API.
1. As an artifact producer, I want to tag artifacts that I can pull by said tag, even if they contain references to other artifacts.
   - Yes, artifacts may be pushed and pulled by tag.
1. As an artifact producer, I want be sure that pushing an artifact to a repository will not affect a copy of the same artifact previously created and referenced by a manifest list existing in another repository on the same registry.
   - Yes, artifacts with referrers are pushed separately from the manifests they refer to.
1. As a tool writer, I want to identify whether a registry supports reference types or not.
   - Yes, this can be provided by either the OCI extensions discovery interface, or querying the referrers API and checking for an error.
     When a query to the referrers API fails, the client should push a digest tag.
1. As a tool writer, I would like the option to perform a server side blob mount when copying a large artifact between repositories.
   - Yes, blob APIs are not impacted by this proposal.
1. As a tool writer, I want to be able to include reference types within the Image Layout filesystem format.
   - Yes, digest tags may be used within the Image Layout.

### Content Management

1. As a user, I want to store one or more artifacts in a registry with a reference to another related artifact.
   - Yes, referrers are created by the `refers` field and, for registries without the `referrers` API, a digest tag.
1. As a user, when I delete an artifact, I want the option to delete one or more artifacts referencing it.
   - Yes, referrers could be queried first and explicitly deleted.
     Digest tags can be explicitly deleted.
     Garbage collection should delete any untagged artifact if the `refers` field points to a non-existent manifest.
1. As a user, I want to push an artifact that references another artifact that doesn't exist on the registry yet.
   - Partial, digest tags do not require the target manifest to exist on that registry, but would require clients to query for that tag even if the registry supports the `referrers` API.
     Alternatively, a registry with the `referrers` API would need to disable validation of the `refers` field and garbage collection of untagged artifacts, which is valid under OCI, but not expected to be a default configuration for most registries.
     The `referrers` API would also need to support querying against a digest that doesn't exist.
1. As a user, I want to move an artifact and one or more artifacts referencing it from one registry to another.
   - Yes, after copying the artifact, the referrers should be recursively queried and copied.
1. As a registry operator, I want to help users understand how they can manage the lifecycle of their artifacts.
   - Yes, registry operators can configure their retention policies to support the `refers` field.
     Registries without the `referrers` API would continue to use tag based retention policies.
1. As a registry operator, I want to allow users to "lock" the tags to their artifacts.
   - Yes, registries that enforce tag locking can still have artifacts pushed that refer to that locked artifact.
     Digest tags include a unique hash to allow multiple referrers to the same artifact with minimal risk of a hash collision.
1. As an artifact producer, I want to update an existing artifact with a newer artifact.
   - Yes, artifacts may be pushed with unique creation annotations, and older artifacts can be explicitly deleted by the client.
1. As an artifact producer, I want to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors.
   - Yes, artifacts pushed either without a tag, or with a digest tag, are idempotent.
1. As an artifact author, I want to document other artifacts in one or more registries that my artifact requires and/or provides.
   - Partial, cross repository or registry referrers are not supported with the `referrers` API.
     Digest tags may be used to point to digests that do not exist in the current repository.
     Additional annotations could be defined to direct the client to search for content in another repository or registry.
     The behavior of descriptors that include a `urls` field is unmodified by this proposal which potentially could define a `refers` to an external resource, but this is likely to create more issues than it would solve.
1. As a user, I want assurances that the object I found really did come from the claimed supplier.
   - Yes, artifacts may be signed, and those signatures may be pushed as a referrer to the artifact.
1. As an artifact author, I want to add assurances that the artifact originated from me.
   - Yes, artifacts may be signed, and those signatures may be pushed as a referrer to the artifact.
1. As a registry operator, I want to provide users with retention policies for their artifacts.
   - Yes, registry operators can configure their retention policies to support the `refers` field.
     Registries without the `referrers` API would continue to use tag based retention policies.
1. As a user, I want to apply timestamp or numerical ranges on my artifacts so I can apply retention policies on them.
   - Partial, an artifact may have an annotation added with a timestamp, version, or similar value.
     These annotations could be queried and a retention policy could be created either by the client or a registry server.
     However, this proposal doesn't introduce a specific capability, and leaves retention policies undefined since they are not currently in scope within OCI.
