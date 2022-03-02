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
  ]
}
```

Attaching an artifact to another object in the registry may be done with an OCI Index.
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
    { // the following is an artifact for the above linux/amd64 image
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:a0a0a0...",
      "size": 1234,
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "annotations": {
        "vnd.oci.artifact": "true", // this descriptor points to an artifact
        "vnd.oci.artifact.type": "sig" // additional annotations may be used for filtering queries
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
    { // the following is an artifact for the above linux/arm64 image
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:b0b0b0...",
      "size": 1234,
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      },
      "annotations": {
        "vnd.oci.artifact": "true",
        "vnd.oci.artifact.type": "sbom"
      }
    },
    { // the following is an image with the artifact directly embedded as base64
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:c0c0c0...",
      "size": 1152,
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v7"
      },
      "annotations": {
        // there is no `vnd.oci.artifact` annotation since this is an image descriptor
        "vnd.oci.artifact.sig.data": "<base64-encoded-artifact>" // embedding small artifacts
      }
    }
  ]
}
```

Runtimes select the first matching manifest when they do not understand the artifact annotations.
This is specified in the [OCI image-spec index definition][image-spec-index]:

> If multiple manifests match a client or runtime's requirements, the first matching entry SHOULD be used.

### Registry HTTP API

The distribution-spec is unmodified.
Querying for a reference involves pulling either existing tags pointing to an index that includes the artifact and image, or pulling new tags that use the digest of the referenced image in the tag value.

The index with the embedded artifact references may be pushed to one of three tags.
The following assumes the original index digest was `sha256:000000...` before the artifacts were added:

1. `<repo>:<tag>`:
   - Pushing directly to the target `<tag>` is useful when artifacts are added by the image originator.
   - Altering an existing tag will change the digest, and should be avoided on copies of the referenced artifact.
1. `<repo>:sha256-000000....<type>`:
   - Type indicates the type of artifact (sig, sbom, etc). 
   - This may point directly to the artifact instead of the index.
   - Updating this tag with additional artifacts requires the use of an index and has potential race conditions.
1. `<repo>:sha256-000000....<hash>.<type>`:
   - Adding a short hash of the artifact allows multiple artifacts of the same type with little risk of collision or race conditions.
   - This may point directly to the artifact instead of the index.
   - This increases query complexity since a tag listing must be retrieved and parsed.

[image-spec-index]: https://github.com/opencontainers/image-spec/blob/main/image-index.md
