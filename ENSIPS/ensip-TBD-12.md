---
title: Draft Number Assignment for ENSIPs  
author: Prem Makeig (premm.eth) <premm@unruggable.com>  
discussions-to: <URL>
status: Idea  
created: 2025-05-17  
---

## Abstract

This ENSIP defines a minimal and consistent process for assigning draft numbers to new ENSIPs. Its goal is to simplify the early stages of ENSIP development by ensuring that contributors can reference and track work-in-progress proposals using clearly defined identifiers.

## Motivation

The ENS ecosystem has grown to include a wide variety of contributors in different domains. Having a standard for assigning ENSIP draft numbers makes it easier for authors to share and build momentum around their proposals, helping them gain visibility and community support early in the process. A simple and predictable mechanism is needed to support the early development and discussion of new ENSIPs.

## Specification

When a contributor wishes to start a new ENSIP, they may open a pull request in the ENSIPs repository with a title that accurately describes the content. This file name will be updated once an ENSIP number is assigned. The ENSIP editors will review the proposal and, if accepted as a draft ENSIP, assign it a canonical number by renaming the file to `ensip-XXX.md`, where `XXX` is the next available ENSIP number.

The assigned number MUST be unique and MUST remain unchanged once issued. The editors MAY reject duplicate numbers or proposals that do not meet the basic requirements of an ENSIP.

Editors are responsible for managing the ENSIP registry, including maintaining the sequence of assigned numbers. ENSIP numbers should be allocated in strict ascending order.

## Rationale

This process enables contributors to begin writing and referencing proposals without waiting for final approval, while also allowing the ENSIP editors to maintain control over the namespace. It balances flexibility during the drafting phase with clarity and order as proposals mature.

## Copyright

Copyright and related rights waived via CC0.
