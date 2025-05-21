---
ensip: TBD  
title: Data URL Multicodec for ENS Records  
status: Draft  
type: ENSRC  
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>  
created: 2024-06-07 (originally proposed in ENSIP-TBD-2)  
---

# Abstract

This ENSIP introduces a new multicodec type, `data-url`, for ENS records. This codec allows ENS names to resolve to content embedded directly in a Data URL format. A corresponding multicodec proposal has been submitted to the multiformats/multicodec repository.

Data URLs are particularly useful for storing multimodal data for AI applications. Because Data URLs are self-descriptive, it's possible for example for LLMs to interpret the data without first needing to know the data type. ENSIP-TBD-9 defines how an unlimited number of arbitrary `bytes` type records can be stored with a single ENS name. ENSIP-TBD-11 specifies a `root-context` text record that can be used as the entry point for AI applications using ENS. Using the `root-context` as the starting point, it is possible to define an unlimited number of Data URL resources that can be used in the context of prompts, tools, or resources — for example, for use with chatbot interfaces, agents, and MCP servers.

# Specification

This ENSIP proposes the addition of the following multicodec:

- `data-url`: `0xf3` (pending approval)  
- Temporary "Private Use Area" codec: `0x3000f3`

A pull request has been submitted to the [multiformats/multicodec repository](https://github.com/multiformats/multicodec/pull/353) for approval.

## Format

**Data URL**

```
uvarint(codec2) + byte(length(MIME)) + &lt;MIME bytes as ASCII&gt; + &lt;DATA as bytes&gt;
```

- The MIME type must not exceed 255 bytes.  
- Data is stored in its raw (not base64-encoded) form.

## Example (Base64 Representation for Readability)

```
data:text/plain;base64,SGVsbG8sIFdvcmxkIQ==
```

# Security Considerations

Onchain Data URLs can improve security by eliminating dependencies on external servers. Content embedded this way is immutable and transparent, reducing risks of tampering.

# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
