# Impact Report Test Suite

Human reference corpus for qualifying the agentic post-issuance extraction engine
(Portal.SFI-data plan `docs/superpowers/plans/2026-09-15-agentic-extraction.md`, Task 2 and Task 18).

Thirty post-issuance impact reports, each with the analyst-completed form export and the
source PDF it was completed from. Every row of every table carries the page number(s) it was
taken from.

## Layout

```
testing/        10 reports — development set. Prompts, schemas and routes may be tuned against these.
verification/   20 reports — held-out set. Run once, after targets are agreed. Never tune against these.
manifest.csv    one row per report: split, issuer, report id/guid, year, type, PDF SHA-256, row counts.
```

Each report has its own folder named after the issuer, containing:

- `<report>.pdf` — the source document, byte-identical to the blob in the platform's
  `source-documents` container (`manifest.csv` records the SHA-256).
- `<report>.xlsx` — the platform's post-issuance export filtered to that report, five sheets:
  `Post-Issuance Report`, `Allocation`, `Projects`, `KPIs`, `Bond Attribution`. The `Page number`
  column on every sheet is the human-added provenance.

## Manifest and reference rows (generated)

```
manifest.json          the loader's manifest: split, hashes, tags, reviewers per report
references/<id>.jsonl  one line per (table, row, field): expected value, pages, quote, applicability
heldout_runs.jsonl     append-only log of held-out runs (written by the runner, never by hand)
```

Both are produced from `manifest.csv` and the workbooks by the platform:

```
manage.py agentic_extraction_corpus --corpus <this checkout> bootstrap --reviewer <name>
manage.py agentic_extraction_corpus --corpus <this checkout> check
```

Rerunning `bootstrap` never overwrites a row a reviewer has edited. `check` says what still
blocks the corpus from gating a release.

### What a reviewer fills in

- **`quote`** on every `present` row whose field is critical (ISIN, amounts, currency, KPI value/unit):
  the sentence or cell text in the PDF the value came from.
- **`applicability`** on every blank row: `not_in_report` (the PDF does not carry it) or
  `not_applicable` (the field has no meaning for this row). A blank the analyst simply missed gets
  its value, `present`, pages and a quote instead.
- **`external`** rows (CPP Investments' bond sizes, from Cbonds) are already marked and need nothing.
- **`adjudication`** when two reviewers disagree: both values, who decided, and why.
- **Tags** in `manifest.json` that only a person can set: `scanned` and `multi_period`. The manifest
  refuses to load until at least one report carries each.

## How to read the reference values

The export is **post-cleaning**, so not every column is something the PDF says. Compare an
extraction against the raw capture only:

| Reference (from the PDF) | Derived by the platform (not in the PDF) |
|---|---|
| ISIN, Bond programme, Use of proceeds category, Allocation + Currency, Project name/number/type/country, KPI + Value / Value Text + Unit, Bond Size, Page number | ICMA mapping, Allocation(EUR)/(USD), Fx dates/Provisional, Attributed/Unattributed Amount, Attributed KPI value, Attributed %, Unit Conversion Row Key, Core KPI, Identity Key |

`Identity Key` is the platform's form identity hash, not a hash of the PDF. Use `pdf_sha256`
in `manifest.csv` to bind an evaluation run to the exact source bytes.

## Split

The development set was chosen for coverage, not at random: 3 Bond + 7 Portfolio reports,
including the text-KPI reports (Akademiska Hus, CPP Investments, AYVENS), the table-heavy
ones (Belfius 88 projects, CPP 126 KPIs, Federal State of Hesse 94 KPIs, DNB 66 bonds) and
reports with unnamed projects. The remaining 20 are held out.

## Still to do before this corpus can gate a release

- Per-field quote (not just page), applicability and error criticality.
- Independent verification of the 20 held-out reports against the PDFs by someone not tuning prompts.
- Numeric field/row accuracy, omission, evidence-precision and reviewer-time targets, agreed before the held-out run.
- Coverage gaps: all 30 reports are 2025/2026 (no multi-period pair is labeled) and no report has been confirmed as scanned.
