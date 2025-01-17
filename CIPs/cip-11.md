---
cip: 11
title: Separate Status and Heartbeat Messages
author: Ming Wu <ming@confluxnetwork.org>
discussions-to: Peilun Li <peilun.li@confluxnetwork.org>, Peter Garamvolgyi <peter@confluxnetwork.org>
status: Final
type: Protocol Breaking
created: 2020-07-25
requires: cip-4
---

<!--You can leave these HTML comments in your merged CIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new CIPs. Note that a CIP number will be assigned by an editor. When opening a pull request to submit your CIP, please use an abbreviated title in the filename, `CIP-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the CIP.-->
This CIP proposes to separate the Status message and Heartbeat message.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Currently, Status message is also used for heartbeat. However, in the message, not all the fields should be sent periodically as heartbeat. This causes unnecessary data transfer and network bandwidth consumption. Separating Status message and Heartbeat message will avoid such redundant data transfer.

## Motivation
<!--The motivation is critical for CIPs that want to change the Conflux protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the CIP solves. CIP submissions without sufficient motivation may be rejected outright.-->
The current Status message has the following fields:
```
pub chain_id: ChainIdParams,
pub genesis_hash: H256,
pub best_epoch: u64,
pub node_type: NodeType,
pub terminal_block_hashes: Vec<H256>,
```
The field *chain_id*, *genesis_hash*, and *node_type* do not need to be sent periodically, since we assume these values would not change during the running time of a node. Therefore, we may have a Status message containing these fields and a Heartbeat message containing the rest fields.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Conflux platforms ([conflux-rust](https://github.com/Conflux-Chain/conflux-rust)).-->
TBD

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
TBD

## Backwards Compatibility
<!--All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->
We use message versioning to address the backwards compatibility.

## Test Cases
<!--Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.-->
TBD

## Implementation
<!--The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
See [conflux-rust pull request #1715](https://github.com/Conflux-Chain/conflux-rust/pull/1715).

## Security Considerations
<!--All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. a CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.-->
TBD

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
