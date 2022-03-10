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

[image-spec-index]: https://github.com/opencontainers/image-spec/blob/main/image-index.md
