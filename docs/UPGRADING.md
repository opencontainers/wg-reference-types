# Upgrading

This document is a place to outline the upgrade path for
both clients and vendors, considering that not all will adopt
the processes designed by this WG at once (or perhaps ever).

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
