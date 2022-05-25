# Upgrading

This document is a place to outline the upgrade path for
both clients and vendors, considering that not all will adopt
the processes designed by this WG at once (or perhaps ever).

## Proposal E

The following section describes the upgrade path for
[Proposal E](./proposals/PROPOSAL_E.md).

### Component defintions

The following icons and definitions will be used to illustate various scenarios
in the context of this document.

#### Producer

A producer of images and/or artifacts (a client that "pushes")

| Icon              | State        | Description                          |
| ----------------- | ------------ | ------------------------------------ |
| 🌱  (leaf)        | Current      | Build tooling that does not push references     |
| 🌿  (tree branch) | Intermediate | Pushes image-spec manifests with "refers" field, includes tags if refers API missing           |
| 🌲  (whole tree)  | Complete     | Pushes references using new manifest |

#### Registry

A service that hosts images and/or artifacts

| Icon            | State        | Description |
| --------------  | ------------ | -- |
| 🚲 (bicycle)    | Current      | References supported via tags/"refers" field in existing image-spec manifest |
| ~~🛵 (moped)~~*      | ~~Intermediate~~ | ~~N/A~~ |
| 🏍 (motocycle)  | Complete     | Supports new API and manifest |

*\* It has been determined that there is not a valid "Intermediate" registry state.*

#### Consumer

A consumer of images and/or artifacts (a client that "pulls")

| Icon          | State        | Description |
| ------------- | ------------ | -- |
| 🐀 (rat)      | Current      |  Existing runtime that is unaware of refers |
| 🐿 (squirrel) | Intermediate | Discovers image-spec manifest references via refers API, falls back to tags if refers API is missing |
| 🦫 (beaver)   | Complete     | Discovers referrers via new API and manifest  |

### Scenarios

There are a total of 18 component combinations listed below.
Note: supported scenarios have descriptions in **bold**.

|   | Producer  | Registry | Consumer | Description |
| - | ----------|----------|----------|-------------|
| 1 | 🌱  | 🚲  | 🐀  | **Present state, no refers** |
| 2 | 🌱  | 🚲  | 🐿  | **Consumer looking for refers that don't exist** |
| 3 | 🌱  | 🚲  | 🦫  | **Consumer looking for refers that don't exist** |
| 4 | 🌱  | 🏍   | 🐀  | **Present state, no refers** |
| 5 | 🌱  | 🏍   | 🐿  | **Consumer looking for refers that don't exist** |
| 6 | 🌱  | 🏍   | 🦫  | **Consumer looking for refers that don't exist** |
| 7 | 🌿  | 🚲   | 🐀  | **Producer creating refers, consumer isn't using refers and is not impacted** |
| 8 | 🌿  | 🚲   | 🐿  | **Producer / consumer working in compatibility mode with tags** |
| 9 | 🌿  | 🚲   | 🦫  | **Consumer wants artifact-spec, but can fall back to image-spec refers using tags** |
| 10 | 🌿  | 🏍   | 🐀  | **Producer creating refers, consumer isn't using refers and is not impacted** |
| 11 | 🌿  | 🏍   | 🐿  | **Producer / consumer using image-spec manifests and referrers API** |
| 12 | 🌿  | 🏍   | 🦫  | **Producer pushes image-spec manifest, consumer discovers refers via new API and would prefer new manifest** |
| 13 | 🌲  | 🚲  | 🐀  | Producer attempts to push new manifest, registry rejects as new manifest is unknown |
| 14 | 🌲  | 🚲  | 🐿  | Producer attempts to push new manifest, registry rejects as new manifest is unknown |
| 15 | 🌲  | 🚲  | 🦫  | Producer attempts to push new manifest, registry rejects as new manifest is unknown |
| 16 | 🌲  | 🏍  | 🐀  | **Producer creates new manifest, consumer isn't using refers and is not impacted** |
| 17 | 🌲  | 🏍  | 🐿  | Producer pushes new manifest, consumer finds refers via new API but cannot parse new manifest |
| 18 | 🌲  | 🏍  | 🦫  | **Producer and consumer both use new manifest and API** |

### Registry Transition

1. Registry in current state, referrers pushed as tags. Registries should not block refer field in manifests, but do not need to parse it.
2. Registry indexes refer field in new manifests, reindexes old manifests with a digest tag.
3. Registry enables referrers API and artifact manifest, clients stop pushing or querying digest tags.

## Extensions API for Distribution

Beginning in OCI Distribution Spec v1.1.0 (not yet released), the
[Extensions API for Distribution](https://github.com/opencontainers/distribution-spec/tree/main/extensions)
will provide a method for asking a registry (or individual repository) for which extensions are supported:

```HTTP
GET /v2/_oci/ext/discover
```

```HTTP
GET /v2/{name}/_oci/ext/discover
```

```HTTP
200 OK
Content-Length: <length>
Content-Type: application/json

{
    "extensions": [
        {
            "name": "_<extension>",
            "description": "",
            "url": "..."
        }
    ]
}
```

This may provide a useful upgrade path as new APIs designed by this WG are
proposed but not yet accepted by other parts of the OCI and wider community.
