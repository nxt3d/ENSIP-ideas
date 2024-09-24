---
ensip: TBD
title: Data URL and URI Contenthash
status: Idea
type: ENSRC
author: Prem Makeig (premm.eth) <premm@unruggable.com>, raffy.eth <raffy@unruggable.com>
created: 2024-6-7
---

# Abstract 

This ENSIP extends the `contenthash` field to support two additional content types: data URL and URI.

# Motivation

The `contenthash` field has become the standard for using ENS names for decentralized websites and dapps. With ENSIP-10 and CCIP-Read (EIP-3668), resolving ENS records from L2s and offchain is now possible, reducing the cost of using the `contenthash` field. This makes adopting the [data URL](https://datatracker.ietf.org/doc/html/rfc2397) standard feasible, allowing content like webapps, images, and videos to be stored onchain or offchain. While ENS names are traditionally linked with decentralization, CCIP-Read has increased their flexibility, enabling use cases like centralized offchain names. However, the `contenthash` field still supports only decentralized storage. This ENSIP also introduces a new URL content type for the `contenthash` field, allowing browsers to redirect to a standard URI when loading an ENS name.

# Specification

ENSIP-7 introduced the `contenthash` field for resolving ENS names to content hosted on distributed systems such as IPFS and Swarm. The value returned by `contenthash` is represented as a machine-readable multicodec, which permits a wide range of protocols to be supported by ENS names. The format is specified as follows:

```
<protoCode uvarint><value []byte>
```

protoCodes and their meanings are specified in the [multiformats/multicodec](https://github.com/multiformats/multicodec) repository.

This ENSIP intruduces two new types of new multicodecs, uri and data-url.  

>[!WARNING] 
>These protoCodes are not approved yet!.
>https://github.com/multiformats/multicodec/pull/353

uri: 0xf2

data-uri: 0xf3

Until the protoCodes are approved the "Private Use Area" temporary codes should be used.

uri: 0x3000f2

data-uri: 0x3000f3

## New Formats 
**URI**

Format: `uvarint(codec1) + <URI as utf8 bytes>`

**Data URL**

Format: `uvarint(codec2) + byte(length(MIME)) + <MIME bytes as ascii> + <DATA as bytes>`

>[!Note] 
>`MIME` cannot exceed 255 bytes

Comment: check on the syntax, to unifiy it with other ENSIPs. 

## Web Application View 

**URI:** `$URI` (literal)

e.g. https://domain.com/a/b/c

**Data URL:** `data:$MIME;base64,${base64_encode($DATA)}`

e.g. data:text/plain;base64,SGVsbG8sIFdvcmxkIQ==	

## Web Gateway Resolution (e.g. .limo)

**URI:** 

* The HTTP response MUST be a `HTTP 307` Temporary Redirect.
	
* The response `Location` MUST be `$URI` eg. https://domain.com/a/b.c?d=e.

If the URI is a data URL the web gateway will not resolve the data URL and instead will redirect the browser to the data URL. 

**Data URL:**

* The HTTP reponse MUST be a `HTTP 200` OK.

* The HTTP response MUST be of `Content-type: $MIME`.

When resolving Data URLs, the URL of the request to the gateway is only used to determine the ENS name. Any path or query data of the request URL is ignored. For example `https://name.eth.limo` returns the same data URL as `https://name.eth.limo/a/b/c`.


# Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).


