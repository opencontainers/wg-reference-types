# Proposal B

Add "reference" field to existing schemas

## Description

The following proposal describes adding a new field to the existing JSON schemas to enable the creation of relationships between objects stored in an OCI registry.

Additionally it describes a new read-only endpoint on the registry HTTP API to allow listing of all objects which reference a given object.

## Links

| Description                        | Link                                                            |
| ---------------------------------- | --------------------------------------------------------------- |
| Original proposal (image-spec)     | [View](https://github.com/opencontainers/image-spec/issues/827) |
| PR to implement (image-spec)       | [View](https://github.com/opencontainers/image-spec/pull/828)   |
| Fork of distribution with support  | [View](https://github.com/dlorenc/distribution/tree/references) |
| Diagram of usage pattern           | [View](https://user-images.githubusercontent.com/1007786/121441279-5c61ca00-c93e-11eb-8b18-40b312542044.png) |

## Modifications

### JSON Schema

All existing supported JSON schemas (index/manifest/descriptor) will be allowed to include a new field, `reference`, which is itself a [descriptor](https://github.com/opencontainers/image-spec/blob/main/specs-go/v1/descriptor.go#L22) object:

```jsonc
{
  "mediaType": "icecream/scoops.vanilla.v1.json",
  "size": 2345,
  "digest": "sha256:b2b2b2...",
  "reference": { // <----- New field
    "mediaType": "icecream/cone.v1.json",
    "size": 1234,
    "digest": "sha256:a1a1a1..."
  }
}
```

### Registry HTTP API

A single, read-only endpoint will be added to the registry HTTP API:

```
GET /v2/<name>/manifests/<ref>/references
```

The response will be a valid [index](https://github.com/opencontainers/image-spec/blob/main/specs-go/v1/index.go#L21), containing a `manifests` array, which is the complete list of descriptor objects referencing a given `<ref>` (tag or digest) on a given `<name>` (repo).

```jsonc
{
  "manifests": [
    <descriptor1>,
    <descriptor2>,
    ...
  ]
}
```

For example:

```
GET /v2/products/cones/manifests/neapolitan/references
```

```jsonc
{
  "manifests": [
    {
      "mediaType": "icecream/scoops.vanilla.v1.json",
      "size": 2345,
      "digest": "sha256:b2b2b2..."
    },
    {
      "mediaType": "icecream/scoops.chocolate.v1.json",
      "size": 2345,
      "digest": "sha256:c3c3c3..."
    },
    {
      "mediaType": "icecream/scoops.strawberry.v1.json",
      "size": 2345,
      "digest": "sha256:d4d4d4..."
    }
  ]
}
```
