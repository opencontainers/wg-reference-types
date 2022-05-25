# Proposal C

Create a Node manifest.

## Description

The proposal introduces a "Node" manifest used to organize container images with related artifacts or to group container images together into a higher order artifact. It is based on the typical "Node" data structure.

A detailed description of this proposal can be found [here](https://docs.google.com/document/d/1KYJaCm5Z-BJ7hVOI_lDh4vEvrmyEAMwUp2nJBELstGw/edit).

## Links

| Description                                 | Link                        |
| ------------------------------------------- | --------------------------- |
| A talk that introduces the idea   	      | [View](https://youtu.be/U_Q5bG-G_Pw) |
| A Detailed description with pictures        | [View](https://docs.google.com/document/d/1KYJaCm5Z-BJ7hVOI_lDh4vEvrmyEAMwUp2nJBELstGw/edit) |

## Modifications

### JSON Schema

A new manifest called "node" is introduced.

The JSON Schema for a "node" manifest looks like this:
```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "This is a generic node object",
  "objects": [
    ... list of blob descriptors ...
  ],
  "reference": {
    ... one blob descriptor
    OR pointer to an external reference
    OR none ...
  }
}
```
Restrictions on this manifest schema to prevent loops and reduce load on server side processing:

- "objects" CANNOT contain descriptors to another Node manifest.
- "reference" has a cardinality of 0..1 i.e. it can either contain 0 or 1 descriptor.

This is an example of using the node manifest to refer multiple ice cream ingredients to a specific ice cream:

```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Chocolate Sprinkles Ice Cream",
  "objects": [
    {
      "mediaType": "flavor/vnd.glenandlarry.v0.1.0+batch",
      "size": 10,
      "digest": "sha256:c0c0abea5"
    },
    {
      "mediaType": "toppings/vnd.sprinklesgalore.v1.0.1+batch",
      "size": 12,
      "digest": "sha256:512c17e5"
    }
  ]
  "reference": {
    "mediaType": "icecream/vnd.glenandlarry.chocolatesprinkles.v2.0.0+package",
    "size": 1024,
    "digest": "sha256:1cec12ea77"
  }
}
```

This is an example of multiple ice creams grouped together to make one ice cream product:

```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Neopolitan Ice Cream",
  "objects": [
    {
      "mediaType": "icecream/vnd.glenandlarry.chocolate.v0.0.2+batch",
      "size": 10,
      "digest": "sha256:c0c01a7e"
    },
    {
      "mediaType": "icecream/vnd.glenandlarry.vanilla.v2.0.1+batch",
      "size": 12,
      "digest": "sha256:4a77111a"
    },
    {
      "mediaType": "icecream/vnd.onlyberry.strawberry.v0.0.1+batch",
      "size": 12,
      "digest": "sha256:5d12abe12121"
    }
  ]
  "reference": {
    "mediaType": "icecream/vnd.glenandlarry.neopolitan.v2022+package",
    "size": 3096,
    "digest": "sha256:bec12a1c12a1"
  }
}
```

_NOTE_:The Node manifest looks very similar to the [index](https://github.com/opencontainers/image-spec/blob/main/image-index.md). Further experiments need to be done in order to assert that the index schema can accept OCI descriptors as well as image manifests.

This is an example of using a Node manifest to describe a single ingredient:

```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Ice Cream Cone",
  "objects": [
    {
      "mediaType": "cones/vnd.waffleconeinc.v0.1.0+cone",
      "size": 10,
      "digest": "sha256:114fff13c0"
    }
  ]
  "reference": {}
}
```

### Registry HTTP API

No modifications to the existing API are needed. The proposal includes the following new APIs:

#### References to a tag or digest

```
GET /v3/<name>/referrers/<reference>
```

Response:

```jsonc
{
  "referrers":
    [list of descriptors]
}
```

Example:

```
GET /v3/glenandlarry/referrers/sha256:bec12a1c12a1
```

Gives response:

```jsonc
{
  "referrers": [
    {
      "mediaType": "icecream/vnd.glenandlarry.chocolate.v0.0.2+batch",
      "size": 10,
      "digest": "sha256:c0c01a7e"
    },
    {
      "mediaType": "icecream/vnd.glenandlarry.vanilla.v2.0.1+batch",
      "size": 12,
      "digest": "sha256:4a77111a"
    },
    {
      "mediaType": "icecream/vnd.onlyberry.strawberry.v0.0.1+batch",
      "size": 12,
      "digest": "sha256:5d12abe12121"
    },
  ]

```

#### Reference of a tag or digest

```
GET /v3/<name>/reference/<reference>
```

Response:

```jsonc
{
  descriptor OR empty
}
```

Example:

```
GET /v3/glenandlarry/reference/sha256:abcdef123450
```

Gives response:

```jsonc
{
  "mediaType": "icecream/vnd.glenandlarry.chocolatesprinkles.v2.0.0+package",
  "size": 1024,
  "digest": "sha256:1cec12ea77"
  
}
```

#### Objects contained in a node manifest

```
GET /v3/<name>/objects/<reference>
```

Response:

```jsonc
{
  "objects": [ list of descriptors ]
}
```

Example:

```
GET /v3/glenandlarry/objects/sha256:abcdef123450
```

Gives response:

```jsonc
{
  "objects": [
    {
      "mediaType": "flavor/vnd.glenandlarry.v0.1.0+batch",
      "size": 10,
      "digest": "sha256:c0c0abea5"
    },
    {
      "mediaType": "toppings/vnd.sprinklesgalore.v1.0.1+batch",
      "size": 12,
      "digest": "sha256:512c17e5"
    }
  ]
}
```

#### The Description of a node manifest

```
GET /v3/<name>/description/<reference>
```

Response:

```jsonc
{
  description
}
```

Example:

```
GET /v3/glenandlarry/description/sha256:abcdef123450
```

Gives response:

```jsonc
{
  "Chocolate Sprinkles Ice Cream"  
}
```

## Requirements

#### Filtering

1. As a user, I want to query a registry for a stored artifact by its digest or tag.
  - Yes: use `GET /v3/<name>/objects/<reference>` to get a list of objects.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
  - Yes: use `GET /v3/<name>/referrers/<reference>`.
1. As a user, I want to query a registry for all stored artifacts of a particular type that reference a given artifact by its digest or tag.
  - Partial: use `GET /v3/<name>/description/<reference>`, then use the client to filter based on `description` content. This is to allow artifact producers to include specific keywords rather than relying on IANA artifact types.
1. As a user, I want to query a registry for all stored artifacts based on annotations that reference a given artifact by its digest or tag.
  - Yes: In this proposal, the existing image manifest layout does not change and can still be filtered by annotations.
1. As a user, I want to fetch the most up-to-date artifact, collection of artifacts, or application.
  - Partial: If Node manifests are used to combine artifacts of a certain type, and a client follows a pattern where the most up-to-date artifact is appended to the beginning of the list, then the most up-to-date artifact is the one at the beginning of the list. However, there isn't any strong protection against not following these rules. Clients will have to rely on some indicator in the description that points to the "latest" artifact. Another way to implement this is to have a Node reference pointing to an older Node reference. Then the first Node manifest represents the most recent artifact. The drawback of this mechanism is long Node chains which registries will have to garbage collect.
1. As an artifact producer, I want to reduce the number of tags that reference an artifact.
  - Yes: Node manifests pointing to a list of artifacts removes the requirement to tag the artifacts themselves if needed.

#### Backwards Compatibility

1. As a user, I want to be sure that existing container runtimes are not affected by any other type of registry artifact.
  - Yes: Existing manifests such as container or index manifests are not affected.
1. As a user, I want to move container images to and from registries that do not support reference types.
  - Partial: Registries that do not support node manifests require the client to create tags for each of the embedded artifacts before pushing them. All the information in description and reference will be lost. Registries that support node manifests have no effect on artifacts that require tagging.
1. As an artifact producer, I want to tag artifacts that I can pull by said tag, even if they contain references to other artifacts.
  - Yes: Node manifests will not affect tags on other artifacts.
1. As an artifact producer, I want be sure that pushing an artifact to a repository will not affect a copy of the same artifact previously created and referenced by a manifest list existing in another repository on the same registry.
  - Yes: Node manifests do not modify existing artifacts nor do they create duplicates of the same artifact, as is the case with existing registries.
1. As a tool writer, I want to identify whether a registry supports reference types or not.
  - Yes: use `GET /v3/`. Catalog listing is not prescribed in this proposal.
1. As a tool writer, I would like the option to perform a server side blob mount when copying a large artifact between repositories.
  - Yes: If a large blob is pushed to the registry and then referenced by a Node manifest, a server side mount should be possible.
1. As a tool writer, I want to be able to include reference types within the Image Layout filesystem format.
  - Yes: The existing image layout will be unaffected.

#### Content Management

1. As a user, I want to store one or more artifacts in a registry with a reference to another related artifact.
  - Yes: Node manifests can group one or more artifacts with a reference to another manifest.
1. As a user, when I delete an artifact, I want the option to delete one or more artifacts referencing it.
  - Partial: use `GET /v3/<name>/referrers/<reference>` to get a list of referring artifacts, use the client to delete specific ones, then push a new Node manifest containing the updated object list. To delete the whole manifest, use `DELETE /v3/<name>/manifests/<reference>`.
1. As a user, I want to push an artifact that references another artifact that doesn't exist on the registry yet.
  - Yes: This is should be possible if registries don't perform a check that the digest exists.
1. As a user, I want to move an artifact and one or more artifacts referencing it from one registry to another.
  - Partial: The client is responsible for selection of the artifacts and pushing to the target registries, including checks if the registry supports node manifests.
1. As a registry operator, I want to help users understand how they can manage the lifecycle of their artifacts.
  - Yes: Node manifests allow for lifecycle policies to be applied to the list of objects.
1. As a registry operator, I want to allow users to "lock" the tags to their artifacts.
  - Yes: Collections of different types of artifacts may be referenced by a long lived tag.
1. As an artifact producer, I want to update an existing artifact with a newer artifact.
  - Partial: use `GET /v3/<name>/manifests/<reference>` to fetch a node manifest, then update the objects list with a new artifact.
1. As an artifact producer, I want to push multiple artifacts concurrently (possibly from separate systems) without encountering race conditions or other errors.
  - No: If two manifests with the same digest are pushed to the same registry path, a race condition will occur. However, this problem exists in current registries.
1. As an artifact author, I want to document other artifacts in one or more registries that my artifact requires and/or provides.
  - Partial: Use the description to indicate the reference is a "requires" or "provides" relationship.
1. As a user, I want assurances that the object I found really did come from the claimed supplier.
  - Yes: Node manifests can include attestation, signature, and SBOM artifacts, which can be linked using references.
1. As an artifact author, I want to add assurances that the artifact originated from me.
  - Yes: Node manifests can include supplier information with attestations.
1. As a registry operator, I want to provide users with retention policies for their artifacts.
  - Yes: Server side policies on node manifest size or length of the objects list can be enforced. Timestamp data may be embedded in the description. A registry provider would then have to require that every node manifest have timestamp data in the description which it can read.  
1. As a user, I want to apply timestamp or numerical ranges on my artifacts so I can apply retention policies on them.
  - Yes: Clients can enforce ranges on objects and description in accordance with update and retention policies.
