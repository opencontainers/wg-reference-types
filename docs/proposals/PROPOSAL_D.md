# Proposal D

No Changes

## Description

The following proposal describes how existing OCI components can be used to provide features of reference types without introducing new manifest schemas or distribution APIs.

## Links

| Description    | Link                                                 |
| -------------- | ---------------------------------------------------- |
| OCI Artifacts  | [View](https://github.com/opencontainers/artifacts)  |
| Image Spec     | [View](https://github.com/opencontainers/image-spec) |

## Modifications

### JSON Schema

The image-spec json schema is unchanged.

The following annotations would be reserved:

- "org.opencontainers.reference": this annotation is defined on an descriptor to an artifact and is set to the digest of another manifest that this artifact references.
- "org.opencontainers.reference.type": this annotation is defined on descriptors and artifact manifests to indicate the type of artifact.

The artifact is defined using the existing OCI Artifact definition (image-spec to a custom config media type):

```jsonc
{
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "schemaVersion": 2,
  "config": {
    "mediaType": "<artifact-config-media-type>",
    "size": 1234,
    "digest": "sha256:a1a1a1..."
  },
  "layers": [
    {
      "mediaType": "<artifact-layer-media-type>",
      "size": 1234,
      "digest": "sha256:b2b2b2...",
      "annotations": {
        "com.example.metadata": "value"
      }
    }
  ],
  "annotations": {
    // "org.opencontainers.reference" is not included here since one artifact could reference multiple objects
    "org.opencontainers.reference.type": "sig"
    // additional annotations may be added for filtering queries (to avoid fetching blobs)
  }
}
```

Attaching an artifact to another object in the registry may be done with an OCI Index.
This should only be done by the image originator since changes to this index would modify the digest.
For this example, the above artifact manifest is assumed to be `sha256:a0a0a0...`:

```jsonc
{
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:010101...",
      "size": 1234,
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:020202...",
      "size": 1234,
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:a0a0a0...",
      "size": 1234,
      "platform": {
        "os": "unknown" // platform that shouldn't be used by image runtimes
      },
      "annotations": {
        "org.opencontainers.reference": "sha256:010101...", // this is a reference to another descriptor
        "org.opencontainers.reference.type": "sig" // type is recommended for client filtering
        // additional annotations may be added for filtering queries
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:b0b0b0...",
      "size": 1234,
      "platform": {
        "os": "unknown"
      },
      "data": "<base64-encoded-artifact>", // artifact data may be inlined
      "annotations": {
        "org.opencontainers.reference": "sha256:020202...",
        "org.opencontainers.reference.type": "sbom"
      }
    }
  ]
}
```

Later examples assume the digest of the index above is `sha256:000000...`.

Runtimes select the first matching manifest when they do not understand the artifact annotations.
This is specified in the [OCI image-spec index definition][image-spec-index]:

> If multiple manifests match a client or runtime's requirements, the first matching entry SHOULD be used.

### Registry HTTP API

The distribution-spec is unmodified.
Querying for a reference involves pulling either existing tags pointing to an index that includes the artifact and image, or pulling new tags that use the digest of the referenced image in the tag value.

References may be pushed to the following tags.

1. `<repo>:<tag>`:
   - Pushing directly to the target `<tag>` is useful when artifacts are added by the image originator.
   - Altering an existing index after it is released is not recommended since it will change the digest.
1. `<repo>:<alg>-<ref>.<hash>.<type>`:
   - E.g. `registry.example.org/project-d:sha256-0000000000000000000000000000000000000000000000000000000000000000.0404040404040404.sbom`
   - `<alg>`: the digest algorithm
   - `<ref>`: the referenced digest (limit of 64 characters)
   - `<hash>`: hash of this artifact (limit of 16 characters)
   - `<type>`: type of artifact for filtering
   - Adding a short hash of the artifact allows multiple artifacts of the same type with little risk of collision or race conditions.
   - This may point directly to an artifact instead of the index.
   - Image originators should include these tags for every referenced image inside their index, pointing to the same digest as the original index.
     E.g. `sha256-010101...a0a0a0.sig` and `sha256-020202...b0b0b0.sbom` would be created and both point to `sha256:000000...`.
   - Periodic garbage collection may be performed by clients pushing new references, deleting stale references that have been replaced with newer versions, and tags that no longer point to an accessible manifest.

## Requirements

### Filtering

1. As a user, I want to query a registry for a stored artifact by its digest or tag.
   - Yes: Artifacts can be pushed and pulled by tag.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
   - Yes: The tag list is checked for all digest entries matching the digest of the referenced manifest.
1. As a user, I want to query a registry for all stored artifacts of a particular type that reference a given artifact by its digest or tag.
   - Yes: The tag list is checked for all digest entries matching the digest of the referenced manifest and the desired type for the artifact.
1. As a user, I want to query a registry for all stored artifacts based on annotations that reference a given artifact by its digest or tag.
   - Partial: Each manifest matching the digest of the referenced manifest and type of the desired artifact must be fetched and filtered by the client.
1. As a user, I want to fetch the most up-to-date artifact, collection of artifacts, or application.
   - Partial: Each manifest matching the digest of the referenced manifest and type of the desired artifact must be fetched and filtered by the client.
1. As an artifact producer, I want to reduce the number of tags that reference an artifact.
   - No: This proposal relies on creating tags to track the references.

### Backwards Compatibility

1. As a user, I want to be sure that existing container runtimes are not affected by any other type of registry artifact.
   - Yes: Testing is needed to verify that an index with entries without a platform or with an unknown platform will not break processing of other entries.
1. As a user, I want to move container images to and from registries that do not support reference types.
   - Yes: This proposal does not require changes to the registry.
1. As an artifact producer, I want to tag artifacts that I can pull by said tag, even if they contain references to other artifacts.
   - Yes: Artifacts can be pulled by tag.
1. As an artifact producer, I want be sure that pushing an artifact to a repository will not affect a copy of the same artifact previously created and referenced by a manifest list existing in another repository on the same registry.
   - Yes: Access to artifacts are queried and pulled from their repository using the existing access controls.
1. As a tool writer, I want to identify whether a registry supports reference types or not.
   - Yes: This proposal doesn't require changes to the registry.
1. As a tool writer, I would like the option to perform a server side blob mount when copying a large artifact between repositories.
   - Yes: Artifacts can be copied with existing APIs.
1. As a tool writer, I want to be able to include reference types within the Image Layout filesystem format.
   - Yes: Artifacts may be added to the Image Layout in addition to the referenced manifests.
     Descriptors are added to the `index.json` with the `org.opencontainers.image.ref.name` annotation for each digest tag.

### Content Management

1. As a user, I want to store one or more artifacts in a registry with a reference to another related artifact.
   - Yes: References are created using the unique tag syntax.
1. As a user, when I delete an artifact, I want the option to delete one or more artifacts referencing it.
   - Partial: Clients that delete a manifest should also delete tags with a digest referencing that manifest.
     A client-side GC could also be created to prune digest tags for digests that are no longer available in that repository.
     Client-side GC should not be performed when the user wants to push artifacts before the referenced image, or when creating references to images that are not stored in this repository.
1. As a user, I want to push an artifact that references another artifact that doesn't exist on the registry yet.
   - Yes: Digest tags can be created that point to manifests that do not exist yet.
     Client-side GC should not be performed in these environments.
1. As a user, I want to move an artifact and one or more artifacts referencing it from one registry to another.
   - Yes: Artifacts may be copied between registries.
1. As a registry operator, I want to help users understand how they can manage the lifecycle of their artifacts.
   - Partial: Lifecycle management depends on client-side processing to remove tags to artifacts that should be deleted.
1. As a registry operator, I want to allow users to "lock" the tags to their artifacts.
   - Yes: Tag locking can be used with the unique tags to extend existing manifests with new artifact references.
     There is no need to mutate existing tags, and the originator can package their index in advance to push once with the images and all reference artifacts together.
1. As an artifact producer, I want to update an existing artifact with a newer artifact.
   - Yes: New artifacts are associated by pushing the artifact and the unique tag referencing the target manifest.
1. As an artifact producer, I want to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors.
   - Yes: Each artifact may be pushed separately with a unique artifact manifest and tag that references the target manifest.
1. As an artifact author, I want to document other artifacts in one or more registries that my artifact requires and/or provides.
   - Yes: Unique tags could reference the digest of a manifest in a different registry.
1. As a user, I want assurances that the object I found really did come from the claimed supplier.
   - Yes: Artifacts can be signed, and the signature can be associated with the artifact using the same tag schema used to reference any other manifest.
1. As an artifact author, I want to add assurances that the artifact originated from me.
   - Yes: Artifacts can be signed, and the signature can be associated with the artifact using the same tag schema used to reference any other manifest.
1. As a registry operator, I want to provide users with retention policies for their artifacts.
   - Yes: Artifacts are retained for as long as a tag to that artifact exists.
1. As a user, I want to apply timestamp or numerical ranges on my artifacts so I can apply retention policies on them.
   - Partial: Client-side processing is required to pull and check annotations on each artifact manifest.

[image-spec-index]: https://github.com/opencontainers/image-spec/blob/main/image-index.md
