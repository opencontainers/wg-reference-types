# Proposal F

OCI Index references it all

## Description

This proposal cherry picks ideas from [Proposal C](https://github.com/opencontainers/wg-reference-types/blob/main/docs/proposals/PROPOSAL_C.md) and [Proposal D](https://github.com/opencontainers/wg-reference-types/blob/main/docs/proposals/PROPOSAL_D.md).
A **nested** [OCI Index](https://github.com/opencontainers/image-spec/blob/main/image-index.md) (an OCI Index referencing another OCI index, allowed by the Image Specification) is used as a top-level object to reference all the artifacts attached to an image.
No change to the reference is expected, but a **well-known annotations** section is added.

## Links

| Description    | Link                                                 |
| -------------- | ---------------------------------------------------- |
| OCI Artifacts  | [View](https://github.com/opencontainers/artifacts)  |
| Image Spec     | [View](https://github.com/opencontainers/image-spec) |

## Modifications

### JSON Schema

The image-spec json schema is left unchanged.

The OCI Index object references OCI image manifests and OCI artifacts attached to them, using OCI descriptors.
OCI Artifacts descriptors **SHOULD** contain **reference** annotations, to expose the reference type and the image referenced.
OCI Manifest descriptors **SHOULD** contain **well-known** annotations, to expose metadata attached to the image.

#### Pre-defined annotations

Reference annotations are added to the [Annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md) section.
* `org.opencontainers.reference.digest`: digest referencing an image. The digest **SHOULD** be referenced as OCI Descriptor in the root OCI index **OR** in the nested OCI index to avoid a weak reference.
* `org.opencontainers.reference.type`: type of the artifact referenced by the descriptor.
* `org.opencontainers.reference.description`: Human-readable description of the reference

Example of a reference OCI descriptor, pointing to a SBOM referencing an alpine image on a specific platform:
```json
{
   "mediaType": "application/vnd.oci.image.manifest.v1+json",
   "digest": "sha256:5b0044a1244...",
   "size": 1024,
   "annotations": {
       "org.opencontainers.reference.type": "sbom",
       "org.opencontainers.reference.digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
       "org.opencontainers.reference.description": "Software Bill of Materials of alpine:3.17 linux/arm64 image"
   }
}
```

#### Well-known annotations
A **well-known** annotations section is added to the [Annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md) section.
Those annotations **SHOULD** be added to the referenced image descriptor.
An encountered annotation that is unknown to the implementation **MUST** be ignored.
The annotations **MAY** store small data.
The list will evolve depending adoption.

Example of well-known annotations, adding cosign signatures to an image:
```json
{
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
   "size": 528,
   "platform": {
      "architecture": "arm64",
      "os": "linux"
   },
   "annotations": {
      "org.favorite.icecream": "rocky-road"
   }
}
```

### Scenarii

The current proposal doesn't change the workflow for [the Refrigerated Truck Driver 🚚](https://github.com/opencontainers/wg-reference-types/blob/main/docs/PERSONAS.md#3-the-refrigerated-truck-driver-), [the Bodega Owner 🏪](https://github.com/opencontainers/wg-reference-types/blob/main/docs/PERSONAS.md#4-the-bodega-owner-) and the [The Ice Cream Lover 😍](https://github.com/opencontainers/wg-reference-types/blob/main/docs/PERSONAS.md#5-the-ice-cream-lover-) personas.
Two scenarii are identified for the [Ice Cream Factory Worker 🏭](https://github.com/opencontainers/wg-reference-types/blob/main/docs/PERSONAS.md#2-the-ice-cream-factory-worker-) and [Health Inspector 🕵️‍♀️](https://github.com/opencontainers/wg-reference-types/blob/main/docs/PERSONAS.md#2-the-ice-cream-factory-worker-) personas.

1/ The 🏭 wants to enhance an existing image without changing its digest, already consumed by 😍, so 🕵️ can validate it.
2/ The 🏭 pushes an image with all the pieces needed by 🕵 without changing 😍 workflow.

#### Enhance existing image

An image is already pushed and used. To add artifacts and annotations to it without re-pushin and changing the known digest, a new OCI index is pushed on a new tag, referencing the original image.
The tag **SHOULD** have the following format:
`<repo>:<alg>-<ref>`:
- E.g. `registry.example.org/project-d:sha256-0000000000000000000000000000000000000000000000000000000000000000`
- `<alg>`: the digest algorithm
- `<ref>`: the referenced digest (limit of 64 characters)

If the referenced image is an OCI Index, the manifests **SHOULD** be inserted first, for backward compatibility with runtimes.
Runtimes select the first matching manifest when they do not understand the artifact annotations.
This is specified in the [OCI image-spec index definition](image-spec-index):

> If multiple manifests match a client or runtime's requirements, the first matching entry SHOULD be used.

Example:
```jsonc
// Original image, pushed to registry.example.org/project-d:v1
{
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "schemaVersion": 2,
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
         "size": 528,
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253",
         "size": 528,
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      }
   ]
}
// Reference OCI Index, pushed to  registry.example.org/project-d:sha256-0000000000000000000000000000000000000000000000000000000000000000.0404040404040404
{
   "mediaType": "application/vnd.oci.image.index.v1+json",
   "schemaVersion": 2,
   "manifests": [
      // Original OCI index referenced with signatures on the whole index, can be used by runtimes supporting nested indexes
      {
         "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
         "digest": "sha256:0000000000000000000000000000000000000000000000000000000000000000",
         "size": 1024,
         "annotations": {
            "org.favorite.icecream": "mint-chocolate"
         }
      },
      // Original manifests, for old runtimes, as fallback
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
         "size": 528,
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         },
         // signatures added to the linux/arm64
         "annotations": {
            "org.favorite.icecream": "banana"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253",
         "size": 528,
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         },
         // signatures added to the linux/amd64
         "annotations": {
            "dev.cosignproject.cosign/signature.e59879c": "...",
            "dev.sigstore.cosign/bundle.6436749": "...",
            "dev.sigstore.cosign/certificate.776a3f1": "...",
            "dev.sigstore.cosign/chain.8adf60e": "...",
            "dev.cosignproject.cosign/signature.de25bf6": "...",
            "dev.sigstore.cosign/bundle.37d34ff": "...",
            "dev.sigstore.cosign/certificate.e8adf60": "...",
            "dev.sigstore.cosign/chain.b1c64a3": "...",
            "org.opencontainers.signatures.index":"dev.cosignproject.cosign/signature.e59879c,dev.cosignproject.cosign/signature.de25bf6,dev.sigstore.cosign/bundle.37d34ff,dev.sigstore.cosign/certificate.e8adf60,dev.sigstore.cosign/chain.b1c64a3,dev.sigstore.cosign/bundle.6436749,dev.sigstore.cosign/certificate.776a3f1,dev.sigstore.cosign/chain.8adf60e"
         }
      },
      // artifacts referencing the image manifests
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:5b0044a1244...", // arm64 SBOM
         "size": 1024,
         "annotations": {
            "org.opencontainers.reference.type": "sbom",
            "org.opencontainers.reference.digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
            "org.opencontainers.reference.description": "Software Bill of Materials of alpine:3.17 linux/arm64 image"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:5b0044a44d...", // amd64 SBOM
         "size": 1024,
         "annotations": {
            "org.opencontainers.reference.type": "sbom",
            "org.opencontainers.reference.digest": "sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253",
            "org.opencontainers.reference.description": "Software Bill of Materials of alpine:3.17 linux/amd64 image"
         }
      }
   ]
}
```

#### Enhanced image

An enhanced image can be pushed directly, with all metadata annotations and references.
There is no need to nest index in this scenario.

```jsonc
{
   "mediaType": "application/vnd.oci.image.index.v1+json",
   "schemaVersion": 2,
   "manifests": [
      // Image manifests with signature annotations
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
         "size": 528,
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         },
         // signatures added to the linux/arm64
         "annotations": {
            "dev.cosignproject.cosign/signature.e59879c": "...",
            "dev.sigstore.cosign/bundle.6436749": "...",
            "dev.sigstore.cosign/certificate.776a3f1": "...",
            "dev.sigstore.cosign/chain.8adf60e": "...",
            "dev.cosignproject.cosign/signature.de25bf6": "...",
            "dev.sigstore.cosign/bundle.37d34ff": "...",
            "dev.sigstore.cosign/certificate.e8adf60": "...",
            "dev.sigstore.cosign/chain.b1c64a3": "...",
            "org.opencontainers.signatures.index":"dev.cosignproject.cosign/signature.e59879c,dev.cosignproject.cosign/signature.de25bf6,dev.sigstore.cosign/bundle.37d34ff,dev.sigstore.cosign/certificate.e8adf60,dev.sigstore.cosign/chain.b1c64a3,dev.sigstore.cosign/bundle.6436749,dev.sigstore.cosign/certificate.776a3f1,dev.sigstore.cosign/chain.8adf60e"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253",
         "size": 528,
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         },
         // signatures added to the linux/amd64
         "annotations": {
            "dev.cosignproject.cosign/signature.e59879c": "...",
            "dev.sigstore.cosign/bundle.6436749": "...",
            "dev.sigstore.cosign/certificate.776a3f1": "...",
            "dev.sigstore.cosign/chain.8adf60e": "...",
            "dev.cosignproject.cosign/signature.de25bf6": "...",
            "dev.sigstore.cosign/bundle.37d34ff": "...",
            "dev.sigstore.cosign/certificate.e8adf60": "...",
            "dev.sigstore.cosign/chain.b1c64a3": "...",
            "org.opencontainers.signatures.index":"dev.cosignproject.cosign/signature.e59879c,dev.cosignproject.cosign/signature.de25bf6,dev.sigstore.cosign/bundle.37d34ff,dev.sigstore.cosign/certificate.e8adf60,dev.sigstore.cosign/chain.b1c64a3,dev.sigstore.cosign/bundle.6436749,dev.sigstore.cosign/certificate.776a3f1,dev.sigstore.cosign/chain.8adf60e"
         }
      },
      // artifacts referencing the image manifests
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:5b0044a1244...", // arm64 SBOM
         "size": 1024,
         "annotations": {
            "org.opencontainers.reference.type": "sbom",
            "org.opencontainers.reference.digest": "sha256:c3c58223e2af75154c4a7852d6924b4cc51a00c821553bbd9b3319481131b2e0",
            "org.opencontainers.reference.description": "Software Bill of Materials of alpine:3.17 linux/arm64 image"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "digest": "sha256:5b0044a44d...", // amd64 SBOM
         "size": 1024,
         "annotations": {
            "org.opencontainers.reference.type": "sbom",
            "org.opencontainers.reference.digest": "sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253",
            "org.opencontainers.reference.description": "Software Bill of Materials of alpine:3.17 linux/amd64 image"
         }
      }
   ]
}
```

### Registry HTTP API

> The distribution-spec **SHOULD** solve the race condition occurring when [multiple parties modify an OCI Index in parallel](https://github.com/opencontainers/distribution-spec/pull/251).
Querying for a reference involves pulling either existing tags pointing to an index that includes the artifact and image, or pulling new tags that use the digest of the referenced image in the tag value.

> Registries are expected to handle nested OCI Index correctly.

### Runtime

> A runtime (clients) **SHOULD** handle a nested OCI index correctly.

## Requirements

### Filtering

1. As a user, I want to query a registry for a stored artifact by its digest or tag.
   - Yes: Artifacts can be pushed and pulled by tag.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
   - Partial: The index stores all the artifacts referencing the given tag or digest.
     It is not possible to query artifacts referencing a digest pointing to an image manifest without having a reference to the parent index.
1. As a user, I want to query a registry for all stored artifacts of a particular type that reference a given artifact by its digest or tag.
   - Partial: The index stores all the artifacts referencing the given tag or digest, the client can filter per type.
     It is not possible to query artifacts referencing a digest pointing to an image manifest without having a reference to the parent index.
1. As a user, I want to query a registry for all stored artifacts based on annotations that reference a given artifact by its digest or tag.
   - Partial: The index stores all the artifacts referencing the given tag or digest.
     The client must pull manifests for each artifact to filter by annotation.
     It is not possible to query artifacts referencing a digest pointing to an image manifest without having a reference to the parent index.
1. As a user, I want to fetch the most up-to-date artifact, collection of artifacts, or application.
   - Yes: The tagged index returned by the registry stores all the artifacts referencing the given tag.
1. As an artifact producer, I want to reduce the number of tags that reference an artifact.
   - Partial: The image originator may push a single tag.
     Extending that without changing the digest requires an additional tag using a digest schema.

### Backwards Compatibility

1. As a user, I want to be sure that existing container runtimes are not affected by any other type of registry artifact.
   - TBD: Testing is needed to verify that an index with entries without a platform or with an unknown platform will not break processing of other entries.
1. As a user, I want to move container images to and from registries that do not support reference types.
   - Partial: Some registries do not support a nested OCI Index.
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

### Content Management

1. As a user, I want to store one or more artifacts in a registry with a reference to another related artifact.
   - Yes: References may be created using the unique tag syntax.
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
   - Partial: Once the original tag has been pushed, a single digest schema tag can be pushed with any added artifacts.
     It is not possible to push additional digest schema tags to add additional referrers to an index without changing an existing tag.
1. As an artifact producer, I want to update an existing artifact with a newer artifact.
   - Yes: New artifacts are associated by pushing the artifact and the unique tag referencing the target manifest.
1. As an artifact producer, I want to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors.
   - No: If two manifests with the same tag are pushed, one push will be overwritten by the other. This is similar to the issue experienced creating multi-platform manifests today.
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
