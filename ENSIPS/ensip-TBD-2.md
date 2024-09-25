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

# Rationale 

[ENSIP-7](https://github.com/ensdomains/ensips/blob/master/ensips/7.md) makes it possible to resolve contenthash records, allowing decentralized websites using decentralized storage such as IPFS and Swarm to be resolved using ENS names. Many users, however, would prefer to simply redirect their ENS name to a URI. It is currently possible to use the text record 'url'; however, this has traditionally been used as a profile record to link to a website of the user, for example, to a blog or homepage. This ENSIP makes it possible to redirect the ENS name to a URI using the contenthash field. In some cases, users want to be able to store entire single-page websites or images onchain. With the addition of the Data URL address type, it is possible to resolve a decentralized website that is fully onchain, avoiding the need to worry about pinning data, for example, using IPFS.

An ENSIP was previously proposed by NameSys on the ENS DAO forum, [[Draft] ENSIP-17: DataURI Format in Contenthash](https://discuss.ens.domains/t/draft-ensip-17-datauri-format-in-contenthash/18048/7). Several methods for encoding that Data URL were discussed, including bypassing the multicodec and using the IPFS multicodec format among other methods. Adding two new protoCodes was also discussed, and this ENSIP takes that approach in order not to overload the top-level IPFS codec with other subtypes that aren’t necessarily related to IPFS.

# Security Considerations

Data URLs and URIs are intended for use in web browsers or other user-facing clients, so their security considerations are similar to any web application. However, onchain Data URLs can be safer than a traditional DNS website because the content can be stored entirely onchain, preventing attackers from altering or compromising the website.
  
# Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).


