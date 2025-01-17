---
cip: 90
title: A Space Fully EVM compatible
author: Fan Long <longfan@confluxnetwork.org> Chenxing Li <chenxing@confluxnetwork.org>
status: Final
type: Spec Breaking
created: 2022-01-10
replaces: 72, 80
---


## Simple Summary

Introduce a new space that is fully EVM compatible. 



## Abstract

This CIP aims to introduce a new fully EVM-compatible space. The new space is called **EVM Space,** and the current space is called **Native Space**. The EVM Space follows the same rule as EVM and supports eth rpc like `eth_getBalance`, so the tools from ethereum economics can be used on Conflux directly. 

The accounts in two spaces are separated. A native space account can only interact with the other accounts in native space with the original conflux transaction type. An EVM space account can only interact with the other accounts in EVM space with Ethereum transaction type EIP-155. A new internal contract will be deployed for assets and data cross-space. Unlike cross-chain operations, the cross-space operations are atomic and with layer 1 security. 



## Motivation

Conflux has a virtual machine that is similar to EVM. However, there are still some differences between Conflux VM and Ethereum VM. Conflux has a different transaction format and a different rule for generating addresses from public keys. This impedes the EVM compatible dApps porting to Conflux. The previous CIP-72 and CIP-80 try to solve these obstacles. But that will influence the current applications. So this CIP introduces a new space to construct a space that is fully EVM-compatible without changing the existing accounts and transactions.



## Specification

### Addresses

At the user level, the native space still uses base32 format address like `cfx:aa...` and the EVM space uses hex checksum address like `0xaAD8...` . At the system level, both the native space and the EVM space are in the format of 160 bits. One address in two spaces is two different accounts. Their balances and nonces will be counted separately. 

### Storage Key

The Conflux uses **Delta MPT** as the underlying authenticated data structure for the ledger state. Delta MPT provides a key-value interface to the virtual machine. 

In computing the *storage key* for accounts in EVM space, we calculate the storage key for the account with the same address in the native space and insert byte `0x81` at position 20 (index started at 0). 

In computing the *delta set storage key* (see section 3.2.2 for details) for accounts in EVM space, we compute the storage key for the account with the same address in the native space and insert byte `0x81` at position 32 (index started at 0). 

### Virtual Machine

- Support the precompiled contracts on Ethereum from address  `0x00..01` to address `0x00..08`.
- The `NUMBER` opcode will return the epoch number.
- The `BLOCKHASH` opcode can only take `NUMBER-1` as input. (Unlike Ethereum, which takes any integer in `NUMBER-256` to `NUMBER-1` as input)
- No gas refund in `SSTORE` opcode and `SUICIDE` opcode.
- The operations which occupy storage have a different gas cost.
    - `SSTORE` costs 40000 gas (instead of 20000 gas in Ethereum) when changing a storage entry from zero to non-zero. 
    - When deploying a new contract, each byte costs 400 gas (instead of 200 gas in Ethereum).
    - When creating a new account by `CALL` or `SUICIDE`, it consumes 50000 gas (instead of 25000 gas in Ethereum).


### Cross Space Operations

#### Mapped Account

Each the native space account has a mapped account in the EVM space. The address of the mapped account is the last 160 bits of the keccak hash value of the original account. With the assumption for the security of the keccak hash function, the mapped address will never collide with the addresses generated by Ethereum tools. The native space account can withdraw the balance from the mapped account. 

#### Internal Contract

A new internal contract called the `CrossSpace` contract will be deployed at the address `0x0888000000000000000000000000000000000006` with the following interfaces. The native space user/contract can interact with the accounts in the EVM space and process the return value in the same transaction. So the cross-space operations can be atomic. 

When making a cross-space call, only 1/10 of available gas can be passed in the EVM space. This is aimed to limit gas usage in a cross-space call. 

```solidity
pragma solidity >=0.5.0;

interface CrossSpace {

    event Call(bytes20 indexed sender, bytes20 indexed receiver, uint256 value, uint256 nonce, bytes data);

    event Create(bytes20 indexed sender, bytes20 indexed contract_address, uint256 value, uint256 nonce, bytes init);

    event Withdraw(bytes20 indexed sender, address indexed receiver, uint256 value);

    function createEVM(bytes calldata init) external payable returns (bytes20);

    function transferEVM(bytes20 to) external payable returns (bytes memory output);

    function callEVM(bytes20 to, bytes calldata data) external payable returns (bytes memory output);

    function staticCallEVM(bytes20 to, bytes calldata data) external view returns (bytes memory output);

    function withdrawFromMapped(uint256 value) external;

    function mappedBalance(address addr) external view returns (uint256);

    function mappedNonce(address addr) external view returns (uint256);
}
```

### Block gas limit

Only the block whose block height is a multiple of 5 can pack Ethereum type transaction. The total gas limit of these transaction cannot exceed half of the block gas limit.



## Rationale

<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

### About the storage key

In the current implementation, the 21-th byte of encoded storage key is a valid ascii charachter. So set the highest bit of the 21-th byte can distinguish new storage keys from the existing keys. 

### About the interface of the internal contracts

The internal contracts use type `bytes20` to indicate an address in the EVM space. Because an EVM space address may be invalid in Conflux Space and rejected by some interfaces in RPC.  

## Backwards Compatibility

<!--All CIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CIP must explain how the author proposes to deal with these incompatibilities. CIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

This CIP is spec breaking. 



## Test Cases

<!--Test cases for an implementation are mandatory for CIPs that are affecting consensus changes. Other CIPs can choose to include links to test cases if applicable.-->

TBA.



## Implementation

<!--The implementations must be completed before any CIP is given status "Final", but it need not be completed before the CIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

TBA.



## Security Considerations

<!--All CIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. CIP submissions missing the "Security Considerations" section will be rejected. a CIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.-->

TBA.



## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).