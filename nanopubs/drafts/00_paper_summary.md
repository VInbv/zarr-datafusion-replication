# Paper summary

**Working scratchpad** for the paper-analysis phase.  
This file feeds the Quote / AIDA / Claim drafts. It is **not** itself a nanopub.

---

**Reference**  
**Prototype under replication:** `zarr-datafusion-search`

- **Repo**: https://github.com/developmentseed/zarr-datafusion-search
- **Docs**: https://developmentseed.org/zarr-datafusion-search/
- **Authors / Org**: Development Seed (Kyle Barron, etc.)
- **License**: MIT
- **First commit**: 2025-09-08
- **Latest release**: `py-v0.1.2-beta.1` (2026-05-20)
- **DOI**: None (prototype, not archived on Zenodo)

**Underlying concept**  
Earthmover whitepaper *"Level 2 Data Collections in Zarr / Icechunk"*

---

## Headline claim (Claim 1 — Heterogeneous Arrays)

> With the advent of VirtualiZarr we are often representing chunks from source files that we don't control. For Level 2 and Level 3 datasets like Sentinel 2 this means that virtual Zarr arrays have varying `dtypes`, `codecs` and `crs` values.  
> If the source arrays are heterogeneous, they cannot be concatenated along a dimension to form a single datacube. Because of this we need an alternative to select or discover these arrays other than the normal coordinate or dimensional slicing we use with datacubes.

### Decomposed into two testable sub-claims

1. **Non-concatenability**: Virtual Zarr arrays built from heterogeneous sources (different `dtype`, `codec`, `crs`, shapes) cannot be reliably concatenated along a dimension into a single coherent datacube.
2. **Metadata-search necessity**: Because of (1), an alternative discovery mechanism (metadata stored as 1-D arrays in a `meta` group + DataFusion SQL queries) is needed and sufficient.

---

## How the prototype solves it

- Stores metadata as 1-D Zarr arrays inside a `meta/` group (each field = one array, chunks aligned like Parquet row-groups).
- Uses a custom DataFusion `TableProvider` to expose this `meta` group as a queryable SQL table.
- Supports efficient filtering on `datetime`, `bbox`, `collection`, etc.
- Optional R-tree spatial indexes.

---

## Proposed replication strategy

**Study Type**: Reproduction/Replication Study

**Plan**:
- Use public Sentinel-2 L2A scenes (multiple UTM zones → different CRS, different bands → different dtypes)
- Build virtual Zarr references with VirtualiZarr
- Demonstrate that concatenation fails or is semantically problematic (Sub-claim 1)
- Implement the `meta` group + DataFusion SQL approach and show that metadata-based discovery works (Sub-claim 2)

---

## Open decisions (to be confirmed)

1. Exact anchoring of the Quote (prototype README vs Earthmover whitepaper)
2. Scope of data for the demonstration (number of scenes, AOI, etc.)
3. Whether we want CI execution or local-only for heavy steps

---

**Next files to produce**:
- `01_quote.md`
- `nanopubs/drafts/claim-1-aida.md` etc.
