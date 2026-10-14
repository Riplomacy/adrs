# 0032: Private Web Bucket Served Through CloudFront with Prefix-Scoped Access

**Status:** Accepted\
**Date:** 2026-10-03\
**Deciders:** Jean-Sébastien Dominique\
**Scope:** `riplomacy_bot`\
**Converted from:** INFRA-010

## Decision

A single dedicated web bucket holds everything the bot publishes to the web. The bucket itself is never public. A CloudFront distribution is its only reader, and can only read a dedicated public prefix, which is also the distribution's origin path. Public files are kept in source and deployed with the infrastructure. The distribution uses its default CloudFront domain.

## Context

Ban appeal tickets needed a help animation, and Discord only animates an image inline from a direct link to the file. The animation lived on a third-party image host that could remove it at any time.

Every existing bucket blocks all public access, an automated check enforces that, and they all hold private data, so none could be opened up or reused for public content.

## Options Considered

- **A public-read bucket with no CloudFront:** conflicts with the account-wide public-access rule, and is one mistake away from exposing private content.
- **Reuse an existing bucket:** weakens a bucket whose purpose is being private.
- **Keep the third-party image host:** the file stays under someone else's control.
- **Serve the file from a Lambda Function URL:** runs code and pays per request to serve a static file.

## Consequences

- The bucket never needs a public policy, so the public-access rule applies without exception.
- What's public is defined by the bucket policy, not by what happens to be stored; other content can share the bucket under other prefixes without becoming public.
- Discord gets a stable direct link, and cost is negligible.
- The first CloudFront use in the account: a new kind of resource to maintain, with slow distribution deploys.
- The standard CDK construct for this grants read on the whole bucket, so the origin access is wired by hand.
- The default domain is opaque; there's no hosted zone to attach a custom one to (ADR-0024).
- Replaced files can be served stale until invalidated; deployments invalidate on every change.
