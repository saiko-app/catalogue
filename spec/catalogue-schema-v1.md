# Catalogue SQLite schema, v1

The release database format is `saiko.catalogue`, schema 1. This document describes reference data only. There are no personal journal tables. No database release is included in this repository yet.

## Tables

| Table | Purpose |
| --- | --- |
| `meta` | Format, schema, version, synthetic flag, input digest and canonical row content hash. |
| `source` | Publisher, edition/retrieval/version, licence, URL and primary/derivative/synthetic kind. |
| `substance` | Stable identifier and display name. Names are unique case-insensitively within one built component. |
| `alias` | Attributed identity search names with common, brand, street or systematic kind. |
| `claim` | Source-linked statement, exact applicability, numeric range or text, review state and locator. |
| `not_modeled` | Explicit dose/duration omission with a reason. Absence of data is not zero effect or safety. |

`meta` includes `format`, `schema`, `version`, `synthetic` (`0` or `1`), `inputs_sha256` and `content_sha256`. The input digest identifies the frozen build inputs; it is not the whole-file release hash.

## Applicability and review

Each claim retains its substance ID, route, salt form, isomer, release form, material, population and endpoint. A null form means unspecified, not freebase, racemate or immediate release. Do not convert units or infer applicability across routes/forms without an explicit supported contract.

Review states are `unreviewed`, `reviewed`, `disputed` and `withdrawn`. Withdrawal must remain distinguishable from missing data and withheld values must not become displayable merely through export. A source labelled `derivative` never becomes independent corroboration because it appears in another compilation.

Claim kinds include `dose.threshold`, `dose.light`, `dose.common`, `dose.strong`, `dose.heavy`; `duration.onset`, `duration.comeup`, `duration.peak`, `duration.offset`, `duration.total`, `duration.after`; `note`; and separately gated `model.*` parameters. A schema capable of storing a model claim does not authorize importing one. Numeric units, locators, unknown lineage and scientific review gates remain mandatory.

## Table definitions

The definitions below reflect the prepared primary catalogue's schema. They describe the file format; schema conformance alone is not a scientific, rights or activation check.

```sql
CREATE TABLE alias (
    substance_id TEXT NOT NULL REFERENCES substance (id), text TEXT NOT NULL,
    kind TEXT NOT NULL CHECK (kind IN ('common', 'brand', 'street', 'systematic')),
    PRIMARY KEY (substance_id, text)
) STRICT;

CREATE TABLE claim (
    id TEXT PRIMARY KEY NOT NULL,
    substance_id TEXT NOT NULL REFERENCES substance (id),
    route TEXT, salt_form TEXT, isomer TEXT, release_form TEXT, material TEXT,
    kind TEXT NOT NULL, low REAL, high REAL, unit TEXT, text TEXT, population TEXT, endpoint TEXT,
    review TEXT NOT NULL CHECK (review IN ('unreviewed', 'reviewed', 'disputed', 'withdrawn')),
    source_id TEXT NOT NULL REFERENCES source (id),
    locator TEXT,
    CHECK (low IS NULL OR (high >= low AND low > 0))
) STRICT;

CREATE TABLE meta (key TEXT PRIMARY KEY NOT NULL, value TEXT NOT NULL) STRICT;

CREATE TABLE not_modeled (
    substance_id TEXT NOT NULL REFERENCES substance (id), route TEXT,
    family TEXT NOT NULL CHECK (family IN ('dose', 'duration')), reason TEXT NOT NULL
) STRICT;

CREATE TABLE source (
    id TEXT PRIMARY KEY NOT NULL, title TEXT NOT NULL, publisher TEXT, version TEXT NOT NULL,
    licence TEXT NOT NULL, url TEXT, kind TEXT NOT NULL CHECK (kind IN ('primary', 'derivative', 'synthetic'))
) STRICT;

CREATE TABLE substance (id TEXT PRIMARY KEY NOT NULL, name TEXT NOT NULL UNIQUE COLLATE NOCASE) STRICT;

CREATE INDEX claim_scope ON claim (substance_id, route, kind);
```

## Content verification

The signed update manifest's `sha256` hashes the **whole SQLite file**. `meta.content_sha256` hashes a deterministic representation of stored rows, excluding the hash's own metadata row. Both checks are required; they are different hashes.

Canonical traversal is:

1. `meta`: key, value, excluding `content_sha256`, ordered by key.
2. `source`: id, title, publisher, version, licence, url, kind, ordered by id.
3. `substance`: id, name, ordered by id.
4. `alias`: substance_id, text, kind, ordered by substance_id then text.
5. `claim`: id, substance_id, route, salt_form, isomer, release_form, material, kind, low, high, unit, text, population, endpoint, review, source_id, locator, ordered by id.
6. `not_modeled`: substance_id, route, family, reason, ordered by substance_id, route, family.

A `#table_name` line precedes each table. Each row is serialized as a compact JSON array in column order, preserving nulls, text and REAL numbers. Join lines with a single newline, without an additional trailing newline, and SHA-256 the UTF-8 result. Use the compatible application's canonical JSON/number encoding; this prose is not a replacement serializer or an independent conformance implementation. A later public validator must pin numeric and escaping test vectors before claiming compatibility.

Repeated builds with the same inputs and SQLite library can be byte-identical. The row-content hash supplies the broader platform-independent content contract; do not assume different SQLite versions produce identical file bytes.

## Composition and continuity

The primary catalogue and separately licensed companions are distinct artifacts. Source or claim removal must not reinterpret a recorded substance ID. Identity-only retention preserves references without restoring withdrawn claims or aliases. Journal records retain their own recorded ID and name snapshots.

Companion metadata, source IDs, claims and identity names must pass composition checks, and every historical baseline ID must continue to resolve under the app's approved continuity contract. A new database needs the private authoring pipeline's complete bundle validation before distribution. A public checksum or schema check cannot replace it.

The [v1 signed update](update-v1.md) replaces one primary file. It does not specify independent download/activation of TripSit, draft or identity companions. Each actual release must state its supported component composition and licence boundaries.
