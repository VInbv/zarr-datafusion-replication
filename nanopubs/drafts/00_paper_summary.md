# Source summary

> Working scratchpad for the Phase 1 analysis. The output of this file feeds the
> Quote / AIDA / Claim drafts. It is not itself a nanopub.
>
> **Note on source type:** this replication does *not* target a peer-reviewed
> article with a DOI. The "source" is an open-source **prototype** plus the
> **whitepaper** that inspired it. The template fields below are adapted
> accordingly, and the open decisions this raises are listed at the end.

**Prototype under replication:** `zarr-datafusion-search` — "A prototype for
querying STAC or CMR style metadata about Zarr arrays and groups using
DataFusion, an extensible query engine written in Rust."

- Repo: https://github.com/developmentseed/zarr-datafusion-search
- Docs: https://developmentseed.org/zarr-datafusion-search/
- Authors / org: Development Seed (kylebarron, sharkinsspatial)
- License: MIT
- First commit: 2025-09-08 · latest release `py-v0.1.2-beta.1` (2026-05-20)
- DOI: none found (prototype is not archived on Zenodo as of this writing)

**Upstream concept:** Earthmover whitepaper *"Level 2 Data Collections in Zarr /
Icechunk"*.

- Earthmover: https://www.earthmover.io/
- The whitepaper is referenced (italic, no hyperlink) by the prototype docs and
  README; no public URL / DOI located. Closest DOI-bearing related artefact is
  the Icechunk talk record on Zenodo (`10.5281/zenodo.13989108`), which is *not*
  the whitepaper itself.

## Scope of this replication

The prototype's "Why" section lists two rationales for storing metadata inside a
Zarr/Icechunk store. **Per the requester, we focus on the first claim only.**

### Claim 1 — Heterogeneous Arrays (the claim we test)

Verbatim from the prototype `README.md` (lines 16–17) and the identical text in
`docs/index.md`:

> With the advent of Virtualizarr we are often representing chunks from source
> files that we don't control.  For Level 2 and Level 3 datasets like Sentinel 2
> this means that virtual Zarr arrays have varying `dtypes`, `codecs` and `crs`
> values.
> If the source arrays are heterogeneous, they cannot be concatenated along a
> dimension to form a single datacube.  Because of this we need an alternative
> to select or discover these arrays other than the normal coordinate or
> dimensional slicing we use with datacubes.

### Claim 2 — Synchronization (out of scope, recorded for context)

> Our current metadata management solutions (STAC, CMR, ODC) all use
> disconnected metadata stores which reference raw data assets in object storage.
> ... Using Icechunk as store can alleviate this as array data and metadata
> updates can be completed in a single atomic transaction.

## Decomposing Claim 1 into testable sub-claims

Claim 1 is really a conjunction of two empirical assertions (kept separate so
each maps to its own atomic AIDA / Claim later):

1. **Non-concatenability.** Virtual Zarr arrays built over heterogeneous Level-2/3
   sources (e.g. Sentinel-2) with varying `dtype` / `codec` / `crs` cannot be
   concatenated along a dimension into a single datacube.
2. **Metadata-search necessity/sufficiency.** Because of (1), discovery/selection
   must use a metadata query rather than coordinate/dimensional slicing — and the
   prototype's approach (metadata as 1-D arrays in a `meta` group, queried via
   DataFusion SQL) is a working alternative.

## How the prototype makes the claim (method summary)

- **Storage convention (schema).** Each metadata "field" is a 1-D Zarr array in a
  root group named `meta`; each array's Zarr v3 `dtype` maps to an Arrow type to
  build an Arrow schema (`docs/architecture/zarr.md`). Example fields: `date`
  (`datetime64[ms]` → Timestamp), `collection` (`VariableLengthUTF8` → Utf8View),
  `bbox` (`VariableLengthBytes` → WKB Polygon / GeoArrow). Chunks are aligned
  across the `meta` arrays so a chunk behaves like a Parquet row group. Optional
  R-tree indexes (geo-index) under an `indexes` group accelerate predicates.
- **Query engine.** A custom DataFusion `TableProvider` (Rust crate + Python
  bindings via `datafusion` PyPI) exposes the `meta` group as a SQL table. Example
  (`docs/examples/stac_virtualizarr.ipynb`):
  `ZarrTable.from_icechunk(session, group_path="/meta")` →
  `SELECT id, datetime, "eo:cloud_cover" FROM icechunk_data WHERE datetime < CAST('2025-10-11' AS DATE) AND "eo:cloud_cover" < 12;`
- **Ingestion.** `ingest_stac_search(...)` queries a STAC API (e.g.
  earth-search) and writes item properties into the `meta` arrays of an Icechunk
  store; `bbox`, `shape`, `transform`, `assets` are flattened into columns.
- **Benchmarks.** `benches/` measures bbox / datetime filter performance locally
  and on S3, with vs. without indexes, and a raw-`zarrs` baseline.

The prototype itself is exploratory; it ships no single numerical "headline
result" the way a paper would. It demonstrates *feasibility* (SQL discovery over
in-store metadata) and *performance* (benchmarks), not a hypothesis test.

## Proposed replication design

**Recommended Study Type: Reproduction/Replication Study.**

- [ ] **Reproduction Study** — same methodology, same tools.
- [ ] **Replication Study** — different methodology / conditions.
- [x] **Reproduction/Replication Study** — both.

Justification: we reproduce the prototype's *approach* (the `meta`-group +
DataFusion-SQL discovery pattern) but on *independent, freely accessible* data
and conditions (the prototype's notebooks/benchmarks lean on credentialed,
requester-pays buckets such as `usgs-landsat`). Concretely, the plan is:

- **Sub-claim 1 (non-concatenability):** assemble several public Sentinel-2 L2A
  scenes spanning different UTM zones (different EPSG `crs`) and bands of
  differing native resolution/`dtype`; build virtual Zarr refs with VirtualiZarr
  and show that concatenating them into one datacube fails or is semantically
  invalid (incompatible CRS / shapes / dtypes along the concat dimension).
- **Sub-claim 2 (metadata search works):** ingest the same scenes' metadata into
  the `meta`-group convention and demonstrate discovery via DataFusion SQL
  predicates (datetime, bbox, collection), reproducing the prototype's pattern.

This maps onto the template's `notebooks/01_data_download.py` … `04_figures.py`
pipeline in Phase 2.

## Open decisions for the requester (need confirmation before drafting nanopubs)

1. **Quote anchoring / Cited reference.** There is no paper DOI. Options:
   (a) anchor the FORRT chain's Quote on the **prototype** (text is quotable
   verbatim from its README/docs; the prototype is what we replicate); or
   (b) anchor on the **Earthmover whitepaper** (preferred conceptually, but no
   accessible URL/DOI found — would need the requester to supply it); or
   (c) treat as a **question-rooted** chain (PCC/PICO) instead of paper-rooted.
   *Recommendation:* (a) for now, citing the whitepaper as conceptual origin,
   and switch to (b) if the requester can provide the whitepaper URL/DOI.
2. **Study Type** confirmation (recommended: Reproduction/Replication).
3. **Data scope** for the demo (how many scenes / which AOI / how heavy a build
   we want on CI vs. local-only).

## Notes for downstream drafts

- Keep sub-claim 1 and sub-claim 2 as **separate** atomic AIDA sentences; do not
  join them with "and" (see `docs/forrt-form-fields.md` § Atomic AIDA).
- `crs` / `dtype` / `codec` are code identifiers in the source — preserve
  backticks when quoting.
