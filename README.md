# OCI Working Group: Reference Types

## Mission Statement

Propose how to describe and query relationships between objects stored in an OCI registry.

## Background

Here is the [link](https://github.com/opencontainers/tob/blob/main/proposals/wg-reference-types.md)
to the original proposal to create this working group.

## Governance

Link to [governance](./GOVERNANCE.md) document.

## Meetings

* WG Meeting: [Tuesdays at 11:00am PT (Pacific Time)](https://zoom.us/j/92128676364) (weekly). [Convert to your timezone](https://dateful.com/convert/pt-pacific-time?t=11am).
* [Meeting notes and Agenda](https://hackmd.io/bGIxKAxPROi8KlwZMQioXQ?edit).
* [Past Meetings](https://github.com/opencontainers/wg-reference-types/tree/main/minutes).
* [Initial Google Doc](https://docs.google.com/document/d/1SVOWQTowigXzbYdorzfa7tMmrcm91yK12LvSONqziJY/edit)

## In Progress

### Upstream Changes

The following changesets are being prepared to be submitted upstream based on
[Proposal E](./docs/proposals/PROPOSAL_E.md).

These forks are currently found in the [oci-playground](https://github.com/oci-playground)
GitHub org (an extension of this working group).

| Upstream repo                                                                            | Changeset                                                                                                      |
| ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| [openconatiners/distribution-spec](https://github.com/opencontainers/distribution-spec)  | [View](https://github.com/opencontainers/distribution-spec/compare/main...oci-playground:distribution-spec:pr) |
| [openconatiners/image-spec](https://github.com/opencontainers/image-spec)                | [View](https://github.com/opencontainers/image-spec/compare/main...oci-playground:image-spec:pr)               |

### Proposals

The following proposal ("E") is actively under revision and represents the
current direction of the WG:

| ID | Link                                   | Description  |
| -- | -------------------------------------- | ------------ |
| E  | [View](./docs/proposals/PROPOSAL_E.md) | Cherry pick  |

### Inactive Proposals

After much discussion, the following proposals are no longer being evaluated by the WG:

| ID | Link                                   | Description                                                               |
| -- | -------------------------------------- | ------------------------------------------------------------------------- |
| A  | [View](./docs/proposals/PROPOSAL_A.md) | Defines a new artifact manifest and corresponding referrers extension API |
| B  | [View](./docs/proposals/PROPOSAL_B.md) | Add "reference" field to existing schemas                                 |
| C  | [View](./docs/proposals/PROPOSAL_C.md) | Create Node manifest                                                      |
| D  | [View](./docs/proposals/PROPOSAL_D.md) | No Changes                                                                |
| F  | [View](./docs/proposals/PROPOSAL_F.md) | OCI Index references it all                                               |

### Documents

The following documents are actively being updated by the WG:

| Document                                | Description                                                      |
| --------------------------------------- | ---------------------------------------------------------------- |
| [Personas](./docs/PERSONAS.md)          | A friendly frame of reference to characterize our design goals   |
| [Requirements](./docs/REQUIREMENTS.md)  | A list of requirements identified by the WG                      |
| [Upgrading](./docs/UPGRADING.md)        | A description of upgrade scenarios at various stages of adoption |
| [Proposal Template](./docs/TEMPLATE.md) | A template for submitting a new reference types proposal         |
| [Evaluation Rubric](./docs/RUBRIC.md) | A comparison matrix of proposals to use cases         |

## Organizers

* Lachlan Evenson (@lachie83)
* Justin Cormack (@justincormack)
* Michael Brown (@michaelb990)
* Derek McGowan (@dmcgowan)
* Jon Johnson (@jonjohnsonjr)

## Contact

* Slack: [#wg-reference-types](https://opencontainers.slack.com/messages/wg-api-expression)
* [GitHub Discussions](https://github.com/opencontainers/wg-reference-types/discussions)
