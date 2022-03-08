# Proposal C

Create a Node manifest.

## Description

The proposal introduces a "Node" manifest used to organize container images with related artifacts or to group container images together into a higher order artifact. It is based on the typical "Node" data structure used for creating linked lists.

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
    ... list of blob descriptors...
  ],
  "reference": {
    ... one blob descriptor
    OR pointer to an external reference
    OR none ...
  }
}
```

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

_NOTE_:The Node manifest looks very similar to the [index](https://github.com/opencontainers/image-spec/blob/main/image-index.md). It may be possible to modify the index manifest to incorporate these features.

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

This is an example of creating a tree structure with the node manifest:

Existing ice creams:

```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Vanilla Ice Cream - created two months ago",
  "objects": [
    {
      "mediaType": "icecream/vnd.glenandlarry.vanilla.v2.0.1+batch",
      "size": 12,
      "digest": "sha256:4a77111a"
    }
  ]
  "reference": {}
}

{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Chocolate Ice Cream - created one month ago",
  "objects": [
    {
      "mediaType": "icecream/vnd.glenandlarry.chocolate.v0.0.2+batch",
      "size": 10,
      "digest": "sha256:c0c01a7e"
    }
  ]
  "reference": {}
}
```

New ice cream:

```jsonc
{
  "schemaVersion": 3,
  "mediaType": "application/vnd.oci.node.v1+json",
  "description": "Vanilla Chocolate Swirl - created two days ago",
  "objects": [
    {
      "mediaType": "application/vnd.oci.node.v1+json",
      "size": 15,
      "digest": "sha256:b1ab1ab1a"
    },
    {
      "mediaType": "application/vnd.oci.node.v1+json",
      "size": 12,
      "digest": "sha256:cabba9ebeef"
    }
  ]
  "reference": {}
}

```

### Registry HTTP API

No modifications to the existing API are needed. The proposal includes the following new APIs

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
