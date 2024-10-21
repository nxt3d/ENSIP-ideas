---
ensip: TBD
title: ENS Resolver Storage Interface
status: Idea
type: ENSRC
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
created: 2024-10-21
---

## Abstract

An ENS Resolver Storage interface that allows clients to create and update ENS records regardless of the storage medium, such as Layer 1 (L1) Ethereum, a Layer 2 (L2) solution, or an off-chain database. This interface extends the `IExtendedResolver` to provide additional functionality for writing records.

## Motivation

ENS (Ethereum Name Service) allows name records to be stored either natively in a resolver, such as the Public Resolver on L1 Ethereum, or externally. For example, using CCIP-Read and ENSIP-10, it is possible to resolve (read) an ENS subname whose records are on a Layer 2 or stored off-chain using a database. However, there is currently no standard for writing records.

This ENSIP introduces a standard interface that allows clients to write records of an ENS name in a standardized way, whether the names store their records on Layer 1 Ethereum, on a Layer 2, off-chain using a database, or via an API connected to any other storage method. This standard enables clients and applications to write to any ENS name or subname using this protocol.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

### Writing to an ENS Name

Writing to any ENS name that follows this ENSIP involves the following steps:

1. **Resolve the ENS Name**:
   - Use an ENS name resolution method (such as ENSIP-1 or ENSIP-10) to discover the resolver of the name.
   - The resolver may be the actual resolver record of the name or a parent name, such as with wildcard resolution.

2. **Set the resolver of the ENS name as the `storage interface`**:
   - The `storage interface` **MUST** support the `EnsNameStorageInterface` defined below and **MUST** extend the `IExtendedResolver` interface.
   - Contracts that implement this interface **MUST** support ERC-165 and **MUST** return `true` when queried with the interface ID of `EnsNameStorageInterface`.

3. **Call the `write()` function** on the `storage interface`:
   - **a.** Write directly to the `storage interface` smart contract.
   - **b.** If writing to the current `storage interface` is not possible, **revert** with the `WriteToAnotherBlockchain` error containing information to write to another blockchain's `storage interface`.
   - **c.** If writing on-chain is not feasible, **revert** with the `WriteToOffchainAPI` error containing information to write to off-chain storage using an API.

4. **Handle the response**:
   - **If a**, the write operation was successful on the current `storage interface`. No further action is required.
   - **If b**, extract the information from the revert reason and:
     - Change the `storage interface` to the new `storage interface` on the other chain.
     - Return to step 3 to attempt writing again.
   - **If c**, extract the information from the revert reason and:
     - Hash and sign the data using [ERC-712](https://eips.ethereum.org/EIPS/eip-712).
     - The data to be written **MUST** be either:
       - The ABI-encoded function call to a standard ENS write operation (e.g., `setAddress`, `setContenthash`), **or**
       - The ABI-encoded function call to `multicall(bytes[])`, where each `bytes` element is an ABI-encoded function call to standard ENS write operations.
     - Use the provided API to write the signed data.

### ENS Name Storage Interface

To facilitate seamless interaction with alternative storage mediums, the `write()` function **MUST** revert with standardized errors containing the necessary information when it cannot perform the write operation on the current ENS Name Storage Interface. The `EnsNameStorageInterface` is defined below. 

```
import "./IExtendedResolver.sol";

/**
 * @title EnsNameStorageInterface
 * @dev Interface for writing ENS records, extending the IExtendedResolver for resolution capabilities.
 */
interface EnsNameStorageInterface is IExtendedResolver {
    /**
     * @dev Writes a record to the ENS name storage.
     *      This function can handle either a single ENS write operation or a multicall.
     * @param name The name of the ENS record.
     * @param data The ABI-encoded data to be written to the ENS record, which can be a single call or a multicall.
     */
    function write(bytes memory name, bytes memory data) external;

    /**
     * @dev Emitted when a record is written successfully on-chain.
     * @param name The name of the ENS record that was written.
     * @param data The data that was written.
     */
    event OnChainWriteSuccess(bytes indexed name, bytes data);

    /**
     * @dev Reverts with information to write to another blockchain's storage interface.
     * @param targetChainId The Chain ID of the target blockchain.
     * @param targetStorageInterface The address of the storage interface on the target blockchain.
     */
    error WriteToAnotherBlockchain(uint256 targetChainId, address targetStorageInterface);

    /**
     * @dev Reverts with information to write to off-chain storage using an API.
     * @param apiEndpoint The API endpoint to use for off-chain storage.
     * @param payload The ABI-encoded multicall data required by the API to perform the write operation.
     */
    error WriteToOffchainAPI(string apiEndpoint, bytes payload);
}
```

### Using ERC-712 for Data Signing

When writing to off-chain storage using an API, the data **MUST** be signed using [ERC-712](https://eips.ethereum.org/EIPS/eip-712) to ensure integrity and authenticity. 

#### EIP-712 Typed Data Structure

To comply with ERC-712, the data to be signed must follow a specific structure consisting of a domain, types, and the message. Below is the detailed breakdown:

1. **Domain Separator**

   Defines the context of the signing, including the contract name, version, chain ID, and the verifying contract's address. The `verifyingContract` **MUST** be the address of the current `storage interface`, and the `chainId` **MUST** correspond to the chain where this `storage interface` is deployed. This ensures that signatures are bound to the specific `storage interface` and blockchain, preventing replay attacks across different chains or interfaces.

```
{
  "name": "ENS Resolver Storage",
  "version": "1",
  "chainId": 1,
  "verifyingContract": "0xYourStorageInterfaceAddress"
}
```

2. **Types**

   Defines the structure of the data to be signed. In this case, it's a `write` containing the `name` and `data`.

```
{
  "write": [
    { "name": "name", "type": "bytes" },
    { "name": "data", "type": "bytes" }
  ]
}
```

3. **Message**

   The actual data that needs to be signed, including the ENS name and the corresponding data (either a single ENS write operation or a multicall).

```
{
  "name": "0xNameBytes",
  "data": "0xDataBytes"
}
```

4. **Full Typed Data**

   Combines the domain, types, and message into a single object.

```
{
  "types": {
    "EIP712Domain": [
      { "name": "name", "type": "string" },
      { "name": "version", "type": "string" },
      { "name": "chainId", "type": "uint256" },
      { "name": "verifyingContract", "type": "address" }
    ],
    "write": [
      { "name": "name", "type": "bytes" },
      { "name": "data", "type": "bytes" }
    ]
  },
  "domain": {
    "name": "ENS Resolver Storage",
    "version": "1",
    "chainId": 1,
    "verifyingContract": "0xYourStorageInterfaceAddress"
  },
  "primaryType": "write",
  "message": {
    "name": "0xNameBytes",
    "data": "0xDataBytes"
  }
}
```

#### Signing Process

1. **Construct the Typed Data**

   Assemble the domain, types, and message as shown above.

2. **Hash the Typed Data**

   Utilize the EIP-712 hashing mechanism to hash the constructed typed data.

3. **Sign the Hash**

   Sign the hashed data using the client's private key. This can be done using libraries like [ethers.js](https://docs.ethers.io/v5/) or [web3.js](https://web3js.readthedocs.io/).

```
const { ethers } = require("ethers");

// Example data
const domain = {
  name: "ENS Resolver Storage",
  version: "1",
  chainId: 1,
  verifyingContract: "0xYourStorageInterfaceAddress"
};

const types = {
  write: [
    { name: "name", type: "bytes" },
    { name: "data", type: "bytes" }
  ]
};

const message = {
  name: "0xNameBytes",
  data: "0xDataBytes"
};

// Sign the typed data
const signer = new ethers.Wallet(privateKey);
const signature = await signer._signTypedData(domain, types, message);
```

4. **Transmit the Signed Data**

   Send the signed data to the off-chain API for processing.

#### Example of ABI-Encoded Multicall

The `data` parameter **MUST** be either:

- A single ABI-encoded function call to a standard ENS write operation, **or**
- An ABI-encoded function call to `multicall(bytes[])`, where each `bytes` element is an ABI-encoded function call to standard ENS write operations.

**Single Write Operation Example:**

```
bytes memory data = abi.encodeWithSignature("setAddress(bytes32,address)", node, newAddress);
```

**Multicall Example:**

```
bytes[] memory calls = new bytes[](2);
calls[0] = abi.encodeWithSignature("setAddress(bytes32,address)", node, newAddress);
calls[1] = abi.encodeWithSignature("setContenthash(bytes32,bytes)", node, newContenthash);
bytes memory data = abi.encodeWithSignature("multicall(bytes[])", calls);
```

#### JSON Structure for Signing

Below is an example JSON structure that a client would use to sign the `write` using ERC-712.

```
{
  "types": {
    "EIP712Domain": [
      { "name": "name", "type": "string" },
      { "name": "version", "type": "string" },
      { "name": "chainId", "type": "uint256" },
      { "name": "verifyingContract", "type": "address" }
    ],
    "write": [
      { "name": "name", "type": "bytes" },
      { "name": "data", "type": "bytes" }
    ]
  },
  "domain": {
    "name": "ENS Resolver Storage",
    "version": "1",
    "chainId": 1,
    "verifyingContract": "0xYourStorageInterfaceAddress"
  },
  "primaryType": "write",
  "message": {
    "name": "0xNameBytes",
    "data": "0xDataBytes"
  }
}
```

#### Verifying the Signature

On the server-side or within the off-chain API, verify the signature to ensure that it was signed by an authorized party.

```
const { ethers } = require("ethers");

// Reconstruct the typed data
const domain = { /* as above */ };
const types = { /* as above */ };
const message = { /* as above */ };

// Recover the signer address
const recoveredAddress = ethers.utils.verifyTypedData(domain, types, message, signature);

// Compare with expected signer
if (recoveredAddress === expectedSignerAddress) {
  // Proceed with writing data
} else {
  // Reject the request
}
```

### Summary

By adhering to the ERC-712 standard for data signing, this ENSIP ensures that write operations to ENS records are secure, authenticated, and can be reliably processed either on-chain or off-chain. The structured approach to signing and verifying data, combined with precise usage of the `storage interface` and `chainId`, ensures that the ENS storage is accurate and protected across various storage mediums and blockchains.
