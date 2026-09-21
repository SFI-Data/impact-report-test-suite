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
