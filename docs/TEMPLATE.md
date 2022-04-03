# Proposal \<LETTER\>

Short description here that will fit neatly in the README table

## Description

Longer description here in 100 words or less. Cras imperdiet nisl odio, sed fermentum urna consequat eu. Integer sed condimentum turpis. Vestibulum at ultrices nunc. Suspendisse semper lectus ut dolor maximus ornare.

Etiam eu dolor tellus. Suspendisse sed eros vulputate, cursus urna eget, interdum nunc. Morbi sagittis ante sed eros hendrerit placerat. Aliquam sagittis blandit velit eget ullamcorper. Vivamus hendrerit egestas dui, non fringilla risus.

Vestibulum blandit nec ex in fringilla. Nam lacus diam, interdum id quam sed, sollicitudin tincidunt quam. Integer eget magna erat. Sed vel efficitur quam, id viverra lacus. Curabitur sapien nisi, placerat quis dapibus ut, malesuada.

## Links

| Description                                 | Link                        |
| ------------------------------------------- | --------------------------- |
| GitHub issue where this was first proposed  | [View](https://example.com) |
| Sample code of a working proof-of-concept   | [View](https://example.com) |
| Blog post which proves the point            | [View](https://example.com) |

## Modifications

### JSON Schema

In this section, explain any modifications which may be needed in the existing schema of
JSON manifests, indexes, etc.

If no modifications are required, describe how you propose to use the existing schema
to achieve the goals of this working group.

Include code blocks of JSON with comments to illustrate your proposal:

```jsonc
{
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  ...
  "icecream": "r.example.com/flavors/chocolate:0.1.0" // <-- Remote ref here
}
```

The field `icecream` above is used to link a container image to an ice cream cone.

### Registry HTTP API

In this section, explain any modifications which may be needed in HTTP API used to
interact with registries.

If no modifications are required, describe how you propose to use the existing API
to achieve the goals of this working group.

Include code blocks of sample HTTP requests to illustrate your proposal:

```text
GET /v2/<name>/manifests/<ref>/icecream
```

Response:

```jsonc
{
  "flavors": {
    "chocolate": {
      "providers": [
        "r.example.com/flavors/chocolate:0.1.0"
      ]
    }
  }
}
```

## Requirements

In this section, copy the distilled user stories from [REQUIREMENTS.md](REQUIREMENTS.md) and include a description of how each is handled by this proposal.

### Filtering

1. As a user, I want to query a registry for a stored artifact by its digest or tag.
   - Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
1. As a user, I want to query the registry for all stored artifacts that reference a given artifact by its digest or tag.
   - Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
...
