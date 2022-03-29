# Requirements

This document contains a list of requirements identified
to be considered in all proposals originating from this WG.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119) (Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997).

## Member-submitted user stories (Distilled)

### Top level definitions
- An OCI artifact is defined as any artifact that is to be stored in an OCI registry.
- A container image is a specialized OCI artifact as defined by the OCI [image](https://github.com/opencontainers/image-spec) and [runtime](https://github.com/opencontainers/runtime-spec) specifications.

### User stories

The following are categorized user stories 

#### Filtering
1. As a user, I want to query a registry for a stored artifact by its digest or tag.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
1. As a user, I want to query a registry for all stored artifacts of a particular type that reference a given artifact by its digest or tag.
1. As a user, I want to query a registry for all stored artifacts based on annotations that reference a given artifact by its digest or tag.
1. As a user, I want to fetch the most up-to-date artifact, collection of artifacts, or application.
1. As an artifact producer, I want to reduce the number of tags that reference an artifact.

#### Backwards Compatibility
1. As a user, I want to be sure that existing container runtimes are not affected by any other type of registry artifact.
1. As a user, I want to move container images to and from registries that do not support reference types. 
1. As an artifact producer, I want to tag artifacts that I can pull by said tag, even if they contain references to other artifacts.
1. As an artifact producer, I want be sure that pushing an artifact to a repository will not affect a copy of the same artifact previously created and referenced by a manifest list existing in another repository on the same registry.
1. As a tool writer, I want to identify whether a registry supports reference types or not.
1. As a tool writer, I would like the option to perform a server side blob mount when copying a large artifact between repositories.
1. As a tool writer, I want to be able to include reference types within the Image Layout filesystem format.

#### Content Management
1. As a user, I want to store one or more artifacts in a registry with a reference to another related artifact.
1. As a user, when I delete an artifact, I want the option to delete one or more artifacts referencing it.
1. As a user, I want to push an artifact that references another artifact that doesn't exist on the registry yet.
1. As a user, I want to move an artifact and one or more artifacts referencing it from one registry to another.
1. As a registry operator, I want to help users understand how they can manage the lifecycle of their artifacts.
1. As a registry operator, I want to allow users to "lock" the tags to their artifacts.
1. As an artifact producer, I want to update an existing artifact with a newer artifact.
1. As an artifact producer, I want to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors.
1. As an artifact author, I want to document other artifacts in one or more registres that my artifact requires and/or provides.
1. As a user, I want assurances that the object I found really did come from the claimed supplier.
1. As an artifact author, I want to add assurances that the artifact originated from me.
1. As a registry operator, I want to provide users with retention policies for their artifacts.
1. As a user, I want to apply timestamp or numerical ranges on my artifacts so I can apply retention policies on them. 

## Member-submitted user stories (Raw)

The following user stories have been submitted by members of
the WG, and MAY be converted to official requirements.

Stated more clearly, just because something is listed below does
not necessarily mean this WG has reached consensus to support
the use case.

Titles of subsections below are the name of the member who
originally submitted them.

*General note: a lot of these say "container image" to ground them
in a non-controversial real world scenario; in general a lot of
these apply to Artifacts too*

### Lachlan

- As a user, I want to store a signature/SBOM/etc in a registry along with a reference to the container image <by digest?> it is for
- As a user, I want to query the registry the signatures for a container image by name and tag
- As a user, I want to query the registry for all stored objects that reference a container image by name and tag
- As a user, I want to query the registry for stored objects that reference a container image filtering by type (eg. Signature, SBOM, etc) or by annotation (I want to see all signatures from this identity)
- As a user, I want to store a signature for an SBOM in a registry along with a reference to SBOM it is for
- As a user, I want to store a signature for an SBOM in a registry along with a reference to SBOM it is for
- As a user, I want to query the registry for the most recent stored objects that reference a container image
- As a user, I want all stored objects that reference a container image to be deleted when a container image is deleted
- As a user, I want to be able to store a SBOM or signature in a registry without the container image it’s for
- As a user, I want to be able to copy a container image and all it’s references to another registry

### Steve

- As a user, I want to copy a container image, and a subset of its references to another registry
- As a user, I want to view the top-level container images in a registry through tags, but only see the references as a selected detail
- As a registry operator, I want to help users understand how they can manage the lifecycle of their container images and their references
- As a registry operator, I want to provide tag locking features that don’t preclude users from adding signatures or SBOMs to a tagged artifact
- As an artifact author, I want to use multiple blobs/layers for reference types
- As an artifact author, I want to push annotations (without blob data) as additional metadata, as reference types
- As a security admin, I want to assure non-container artifacts, pushed to a registry doesn't add security risks to container runtime services that may pull the reference type 
- As a container image author, I want to support promotion of container images, and their references, across registries that may not yet support references
- As a security scanner project/product, I want to know the type of each artifact so I know how to scan it for vulnerabilities, or if I should scan it. (example, scanners evaluate signatures, but don’t "scan" them)

### Nisha

- As a user, I want to identify the most updated artifact in a registry
- As a user, I would like to be able to map monotonically increasing product versions to container images so I have an idea of deployment progression
- As a developer/devopser, I would like to bisect builds based on container images I have deployed

### Josh

- As a user, I want to store images in one registry, and signatures/SBOMs/attestations in a separate registry

### Brandon

- As an artifact producer, I want to be able to create multiple artifacts referring to the same manifest, and upload them separately
- As an artifact producer, I would like to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors
- As an artifact producer, I would like to be able to push an artifact to a registry, associated with another manifest, and not need to tag that artifact (reducing tag clutter)
- As an artifact producer, I want to push an artifact to a registry with a tag so it can be pulled directly, in addition to having references to other manifests in the registry
- As an artifact producer, I would like an efficient way to update an existing artifact with a newer version (replacing an expiring signature)
- As an artifact producer, pushing an artifact to one repository should not impact the associated artifact list on a copy of the manifest previously created in another repository on the same registry
- As a tool writer, I would like to be able to efficiently query artifacts of different types attached to a given digest
- As a tool writer, I would like to be able to query for a specific artifact based on artifact type and other user defined annotations on the artifact
- As a tool writer, I would like to be able to identify when a registry does not support the new APIs
- As a tool writer, I want to be able to include reference types within the Image Layout filesystem format and have a way to lookup those reference types without reading the blob for every manifest
- As a tool writer, I would like to perform a server side blob mount when copying a large artifact between repositories
- As a user, I would like to walk the CAS tree in reverse. This would allow a query of all manifests pointing to a blob digest, and all manifest lists pointing to a manifest digest
- As a registry operator, I want attempts to push artifacts to an older registry to have a minimal impact (e.g. artifacts are not missed by GC causing bloating of storage, and a tagged artifact does not break tag listings)
- As a user, I would like graceful degradation if the registry does not support the new ref types. This would either be some kind of limited fall back support or a clear error message from tooling
- As a user, I want to be sure existing runtimes are not affected by any of the new reference types added to images
