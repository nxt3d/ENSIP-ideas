---
title: Root‑Context Text Record  
author: Prem Makeig (premm.eth) <premm@unruggable.com>  
discussions-to: <URL>
status: Idea  
created: 2025-05-17  
---

## Abstract

This ENSIP extends **ENSIP‑5: Text Records** by standardizing a single global text record key, **`root-context`**. The key acts as the “home page” for an ENS name when viewed by large language model (LLM) systems: it points to a manifest that tells an AI client *what* the name represents (data source, chatbot, autonomous agent, etc.) and *how* to interact with it. By keeping that manifest onchain, ENS gives any app (chat front‑ends, wallet UIs, MCP middleware, crawlers) one reliable place to look it up.

## Motivation

ENS is a globally recognized, tokenized naming system whose ownership is verifiable on-chain. This makes it a natural anchor for agents and AI-focused datasets. A single `root-context` text record lets any client, whether a chat app, wallet, crawler or MCP middleware, load the initial instructions it needs before deciding how to behave. This newly introduced text record supplies the context an LLM requires, signals which interface to present (chat, agent or data), and keeps the pointer verifiable and reproducible because it is stored on-chain.

## Specification

### Text Record Key

* **Key**: `root-context`
* **Value**: Text in UTF-8 format for a manifest that outlines one or more interfaces (for example, `chat`, `agent`, `data`) plus any supporting metadata.
* **Expected Format**: Plain text, YAML, JSON, or other standard formats designed for LLM context.

The key **MUST** be published via `text(bytes32,string)` as defined in ENSIP‑5. Implementers MAY embed multiple interfaces in a single manifest; each interface can be described in natural language or structured metadata.

### Semantic Role *(informative)*

Think of `root-context` as the landing page of a web application, except that it is optimized for AI, especially LLMs:

* **Chat interface.** A manifest might instruct: “You are the official support bot for my blog. Greet users and answer FAQ from the dataset at `ipfs://…`.” Chat clients that recognize this cue can preload the context before opening a conversation.
* **Agent interface.** The same manifest could include a second section: “When invoked as an *agent*, the ENS name can be loaded by MCP middleware, which exposes tools to the LLM. The tools themselves may be defined using dynamic text record keys or smart contract addresses and interfaces. Additional resources such as an API endpoint for an LLM or a memory service may be supplied to the MCP middleware to support the agent's behavior. This provides all the necessary components to "enliven" the ENS name as a fully functional AI agent.”
* **Data-only mode.** In read-only scenarios, a manifest could simply declare: “This ENS name hosts the latest quarterly report PDFs.” These files could be stored as the contenthash record of the name, referenced using [ENSIP-TBD-9](https://github.com/nxt3d/ENSIP-ideas/blob/main/ENSIPS/ensip-TBD-9.md), or linked using `ipfs://`, `ar://`, or standard URLs. Indexers would treat it as a static data source.

Each interface is *discoverable* through the manifest, so no additional ENS text records are required.

### Client Resolution Flow *(informative)*

1. Resolve `root-context` for the target ENS name.
2. Retrieve the manifest.
3. Parse the manifest to select an interface that matches the client’s capabilities (for example, chat or agent).
4. Proceed according to the manifest’s instructions: initializing an LLM, loading tools, displaying UI hints, or fetching datasets.

### ENS Tool Calling *(example)*
The root-context record can specify executable tools and functions that AI systems can invoke when interacting with the name. These tools are defined using a standardized JSON format:
```json
{
    "systemPrompt": "This AI can check the ENS balance of an Ethereum address",
    "tools": [
        {
            "name": "getEnsBalanceTool",
            "description": "Fetch the ENS token balance for a given Ethereum address.",
            "parameters": {
                "type": "object",
                "properties": {
                    "address": {
                        "type": "string",
                        "description": "Ethereum address to check ENS token balance for (0x...)."
                    }
                },
                "required": [
                    "address"
                ]
            },
            "execution": "import { createPublicClient, http, parseAbi, formatUnits } from 'viem';\nimport { mainnet } from 'viem/chains';\n\nconst ENS_TOKEN_CONTRACT = '0xC18360217D8F7Ab5e7c516566761Ea12Ce7F9D72';\nconst erc20Abi = parseAbi([\n  'function balanceOf(address owner) view returns (uint256)'\n]);\n\nexport async function main(args) {\n  const client = createPublicClient({\n    chain: mainnet,\n    transport: http('https://eth.drpc.org')\n  });\n\n  const balance = await client.readContract({\n    address: ENS_TOKEN_CONTRACT,\n    abi: erc20Abi,\n    functionName: 'balanceOf',\n    args: [args.address]\n  });\n\n  const formattedBalance = formatUnits(balance, 18);\n  return { ensBalance: formattedBalance };\n}"
        }
    ]
}
```
These tool definitions are executed server-side in a sandboxed environment as standalone isolated functions to minimize attack vectors. The sandbox environment explicitly limits library access, exposing only a curated set of permitted dependencies. Specifically:

- Purpose-built blockchain libraries (like `viem`, `ethers.js`, ENS-specific SDKs)
- General data handling utilities and formatters
- Controlled network access for data fetching with rate limiting
- No access to file system, operating system commands, or arbitrary package installation

When an LLM client encounters this functionality in the `root-context`, it can:

1. Parse the tool definitions
2. Present appropriate UI for invoking the tools
3. Execute the tools in a secure runtime
4. Return results to the LLM for further processing

This approach extends ENS names beyond static identifiers into functional endpoints that AI systems can interact with programmatically, while maintaining security boundaries between execution environments.

### Backwards Compatibility

Unaware clients will simply ignore the new key; existing behavior is unaffected.

### Example

A simple example using Ethers.js:

```js
const resolver = await provider.getResolver("example.eth");
const rootContext = await resolver.getText("root-context");
console.log(rootContext);
```

This command returns the raw content that defines the ENS name’s machine-readable context.

### Security Considerations

There are no security considerations specific to this ENSIP.

## Rationale *(informative)*

The purpose of this ENSIP is to introduce a standardized way for clients to locate and interpret AI-relevant context for any ENS name. By associating a plain text pointer with the name itself, developers and applications can uniformly initialize context-aware behavior in a uniform way.

## Copyright

CC0‑1.0


