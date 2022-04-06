# Proposal E

Cherry Pick

## Description

This proposal combines ideas from proposals A, B, and D to create a solution that provides an ability to use it on existing registries, while giving a value add as registries upgrade in the future.

It consists of the following changes:

- It adds a new media type for future use cases.
- It defines a "reference type" field on both the new media type and existing image manifests.
- It defines annotations for filtering artifacts.
- It defines both an API and a digest tag syntax for querying references.

The result is the following upgrade path:

- Registries without new API or media type support can use the image manifest plus digest tags to create artifacts that reference other manifests.
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

### Annotations

These annotations would be added for artifacts:

- `org.opencontainers.artifact.type`: type of artifact (sig, sbom, etc)
- `org.opencontainers.artifact.description`: human readable description for the artifact
- `org.opencontainers.artifact.created`: creation time for a manifest

Additional annotations should be considered for filtering various artifact types, e.g. signature public key hash, attestation type, and sbom schema.

These annotations should be considered from the current config schema:

- `org.opencontainers.platform.architecture`: CPU architecture for binaries
- `org.opencontainers.platform.os`: operating system for binaries
- `org.opencontainers.platform.variant`: variant of the CPU architecture for binaries

Existing `org.opencontainers.image.*` annotations should be reviewed to consider more generic names (e.g. replacing `image` with `manifest` or `artifact`).

### JSON Schema

Extend the Image Manifest with a reference field (existing registries should ignore this per OCI's extensibility requirement):

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": { ... },
  "layers": [ ... ],
  "reference": { // new field
    "mediaType": "application/vnd.oci.image.manifest.v1+json", // any manifest media type
    "size": 1234,
    "digest": "sha256:a1a1a1...",
    "annotations": [
      // pull up annotations from referenced manifest here
    ]
  },
  "annotations": [
    // annotations for this artifact here
  ]
}
```

Create a new artifact media type to support future use cases where a separate config blob, and an ordered list of blobs, may not match an artifact's use case:

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.artifact.manifest.v1+json",
  "blobs": [
    // optional list, ordering is not enforced by the spec
    // but may be required for specific artifact types
  ],
  "reference": { // optional
    // include reference here
  },
  "annotations": [ // optional
    // annotations for this artifact here
  ]
}
```

### Registry HTTP API

Add the following `references` API:

```text
GET /v2/<name>/references/<ref>
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
    // optional annotations indicating pagination, sort, and filter settings
  ]
}
```

#### Ordering and Pagination

- The registry MUST order the results in a way that allows pagination.
- Adding the `n` query parameter is used to limit the number of entries per index returned.
- The `Link` HTTP header is included in the response when additional results are available and is set to the URL for the next page of results.

#### Filtering and Sorting

- The registry SHOULD support filtering by annotation using the `filter=<url encoded "name=value">` parameter.
- Multiple filter parameters may be set on a request, results must match all filters.
- TBD: More advanced query semantics (e.g. glob, regex, prefix, lists, inverse query).
- The registry SHOULD support sorting by annotation using the `sort=name` parameter.
- Artifacts without a specified sorting annotation are sorted last.
- TBD: Reversing the sort order is needed (prepend a `-` to the name, `sortDesc` parameter, align with how others do this).
- TBD: an HTTP header or an Index annotation is needed for clients to discover when the returned result supported custom sorting and/or filtering.

### Digest Tags

For registries that do not support the `references` API, digest tags MUST be pushed for any manifest containing a `reference` descriptor with the following syntax:

```text
<repo>:<alg>-<ref>.<hash>.<type>
```

- E.g. `registry.example.org/project-e:sha256-0000000000000000000000000000000000000000000000000000000000000000.0404040404040404.sbom`
- `<alg>`: the digest algorithm
- `<ref>`: the referenced digest (limit of 64 characters)
- `<hash>`: hash of this artifact (limit of 16 characters)
- `<type>`: type of artifact for filtering
- Adding a short hash of the artifact allows multiple artifacts of the same type with little risk of collision or race conditions.
- Periodic garbage collection may be performed by clients pushing new references, deleting stale references that have been replaced with newer versions, and tags that no longer point to an accessible manifest.

## Requirements

In this section, copy the distilled user stories from [REQUIREMENTS.md](REQUIREMENTS.md) and include a description of how each is handled by this proposal.

### Filtering

... TBD ...

### Backwards Compatibility

... TBD ...

### Content Management

... TBD ...
