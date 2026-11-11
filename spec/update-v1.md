# Signed catalogue update, v1

This document describes the intended release contract. No production update release or signing key is provisioned by this repository. Publication waits for compatible update code and maintainer-controlled signing.

## Location and files

Canonical root: `https://github.com/saiko-app/catalogue/releases/`.

Release tags use `catalogue-N`, where N is a positive canonical decimal UInt64 sequence, without leading zeroes. Never reuse a sequence or replace published signed bytes. Corrections receive a higher sequence.

Required update assets:

- `manifest.json`: exact UTF-8 JSON bytes, at most 64 KiB.
- `manifest.sig`: raw 64-byte detached Ed25519 signature, not Base64 or hex.
- `catalogue.sqlite`: exact database bytes identified by the manifest, at most 64 MiB.

Additional attribution, licence, provenance and reproduction files accompany the release where applicable. **The v1 signature does not authenticate these sidecars or companion datasets.** It binds one database through its signed byte count and hash. TripSit and other separately licensed datasets retain separate packaging; multi-file atomic updates need a compatible future contract.

## Manifest fields

| Field | Meaning |
| --- | --- |
| `format` | Exactly `saiko.catalogue.update.v1`. |
| `sequence` | UInt64 release counter, strictly greater than the client's authenticated replay floor for a new candidate. |
| `catalogueVersion` | Nonempty string equal to the database `meta.version`. |
| `schema` | Supported catalogue database schema integer, currently 1. |
| `byteCount` | Positive integer equal to the exact database byte count and within the client's size ceiling. |
| `sha256` | Exactly 64 lowercase hexadecimal characters, SHA-256 of the complete database file. |

Illustration only; this is not a valid release, hash or allocated sequence:

```json
{"format":"saiko.catalogue.update.v1","sequence":1,"catalogueVersion":"EXAMPLE-ONLY","schema":1,"byteCount":123,"sha256":"0000000000000000000000000000000000000000000000000000000000000000"}
```

Sign the exact manifest bytes, including any final newline. Do not parse, normalize whitespace or re-encode JSON before verifying its signature. Ed25519 public keys are raw 32-byte values trusted by the application. Repository ownership, HTTPS, a GitHub hash or a downloaded public key cannot substitute for that trust.

## Verification and activation

Authenticate the manifest before treating its contents as trusted. Refuse unsupported schema, replay/non-increasing sequence, invalid byte count or whole-file hash. Check the database format/schema/version and recompute its row content hash as documented in the [database schema](catalogue-schema-v1.md).

Staging does not authorize activation. Activation must recheck current trust, authenticated sequence floor, database content and composed identity continuity inside the installation transaction. It must not open or rewrite a personal journal. Existing readers remain immutable snapshots; the app must publish refreshed readers and invalidate derived caches deliberately.

An authorized previous-version selection revalidates the old signed bytes and composition while preserving the newest authenticated sequence floor and retained identity references. It is not permission to download a lower-sequence candidate. Current trusted keys authorize new candidates; explicitly trusted, non-revoked historical keys are limited to stored-release verification. Key provisioning, custody, rotation and revocation are maintained separately from hosting.

## Download contract

Resolve `latest/download/manifest.json` once to a validated same-repository tag, then fetch all required assets from that same tag. Require HTTPS, the exact repository/path and an explicitly reviewed asset CDN host such as `release-assets.githubusercontent.com`. Reject unrelated repositories, mixed tags and arbitrary wildcard hosts.

The intended bounded transport permits at most three redirects per resource, a shared 120-second attempt deadline, 15-second idle bounds and decoded chunks no larger than 64 KiB. No automatic retries, cookies, credentials, persistent cache, journal payload or app-specific identifiers. Every response must close on completion, refusal, failure or cancellation. These are requirements for the app adapter, not evidence that live traffic has been validated.

An unavailable endpoint leaves local reference data usable. Sequence numbers prevent replay below the locally authenticated floor; v1 has no signed expiry or freshness guarantee. It also does not protect against deletion/restoration of the entire installation store. Signature validity is not a scientific review or a licence grant.
