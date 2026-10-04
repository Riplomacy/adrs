# INFRA-010: Private Web Bucket Served Through CloudFront with Prefix-Scoped Access

**Date:** 2026-10-03\
**Status:** Accepted\
**Deciders:** Jean-Sébastien Dominique

## Context

Ban appeal tickets needed a help animation showing players where to find their Steam ID. Discord animates an image inline when it is given a direct link to a gif file. The animation originally lived on a third-party image host, which can remove or change it at any time. It needed to be hosted somewhere we control.

Every existing bucket in this account blocks all public access, and an automated invariant enforces that on every bucket. They also hold private data (transcripts, staff and Patreon datasets, whitelists, logs), so none of them can be opened up, or reused for public content.

## Decision

A single dedicated web bucket holds everything the bot publishes to the web. It keeps full public-access blocking and enforced TLS like every other bucket -- it is never itself public.

A CloudFront distribution is the only reader, authenticated with Origin Access Control. The bucket policy grants CloudFront read access to the `public/` prefix only, and the distribution's origin path is that same prefix. Anything stored under another prefix cannot be reached through this distribution, even by mistake.

Public static files are checked into the repository and deployed with the infrastructure, so the bucket's public content is reproducible from source.

The distribution is served on its default CloudFront domain; no custom domain is used.

## Consequences

### Positive

- The bucket never has a public policy, so the existing "blocks all public access" invariant applies to it without a waiver
- Prefix scoping makes "what is public" a property of the bucket policy, not of what happens to be stored there
- Discord gets a stable direct URL with the correct content type, so the animation renders inline
- Other content can share the bucket under separate prefixes without becoming public
- Cost is negligible at this traffic volume

### Negative

- First CloudFront use in this account, so a new kind of resource to maintain, with a several-minute deploy time for changes to the distribution itself
- The convenience construct CDK provides for this pattern grants read on the whole bucket and cannot be narrowed, so the origin access wiring is written by hand
- The default CloudFront domain is opaque and not tied to a domain we own

### Neutral

- Replaced public files can be served stale until the cache is invalidated; the deployment invalidates on every change
- There is no hosted zone to attach a custom domain to; the previous one was retired (see INFRA-008)

## Alternatives Considered

- **Public-read bucket with no CloudFront:** Rejected -- needs a public bucket policy, which conflicts with the account-wide public-access invariant, and the bucket would be one mistake away from exposing non-public content
- **Reuse an existing bucket:** Rejected -- every existing bucket holds private data; carving out a public prefix there weakens a bucket whose whole purpose is being private
- **Keep hosting on Imgur or another image host:** Rejected -- the file would stay under a third party's control and could disappear, breaking part of our ticket flow
- **Serve the file from a Lambda Function URL:** Rejected -- pays per request and runs code to serve a static file, and is a poor fit for a static file

## Implementation Notes

- The distribution's price class is the cheapest tier; traffic is a handful of requests per ticket
