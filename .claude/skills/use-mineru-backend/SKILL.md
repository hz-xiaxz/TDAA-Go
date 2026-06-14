---
name: use-mineru-backend
description: Use when converting course reference PDFs with MinerU, choosing a MinerU backend/method, smoke-testing MinerU extraction, or comparing PyMuPDF vs MinerU for scanned, math-heavy, table-heavy, or layout-heavy PDFs in this TDAA-Go project.
---

# Use MinerU Backend

Use this workflow when PDF conversion quality matters more than the default lightweight PyMuPDF path.

## Backend Selection

Prefer the default PyMuPDF backend when:

- The PDF has selectable text and simple layout.
- You only need rough text ingestion.
- Fast setup and low resource use matter.

Prefer MinerU with the local `pipeline` backend when:

- The PDF is scanned, image-heavy, math-heavy, table-heavy, or multi-column.
- Figures, tables, formulas, and reading order matter.
- The machine can spend time downloading/initializing local models.

Prefer MinerU HTTP/client backends only when the user already has a service URL and credentials configured outside the repo:

- `vlm-http-client` for remote VLM parsing.
- `hybrid-http-client` for remote hybrid parsing.
- Pass service URLs with CLI args, not committed config.
- Never write tokens, JWTs, API keys, or Authorization headers into repository files.

## Method Selection

Use `--mineru-method txt` for born-digital PDFs with selectable text when testing quickly.

Use `--mineru-method auto` for mixed PDFs when uncertain.

Use `--mineru-method ocr` for scanned pages or image-only PDFs.

## Commands

Install local MinerU pipeline support outside committed project metadata:

```bash
uv pip install -U "mineru[pipeline]" six
```

Default conversion remains PyMuPDF:

```bash
uv run python -m scripts.convert_references reference/
```

Use local MinerU pipeline:

```bash
uv run python -m scripts.convert_references reference/ \
  --pdf-backend mineru \
  --mineru-backend pipeline \
  --mineru-method auto
```

Use first-page smoke testing when available via raw MinerU args:

```bash
uv run python -m scripts.convert_references /tmp/tdaa-mineru-smoke \
  --pdf-backend mineru \
  --mineru-backend pipeline \
  --mineru-method txt \
  --mineru-arg=--start=0 \
  --mineru-arg=--end=0
```

Use an existing local or remote MinerU FastAPI service:

```bash
uv run python -m scripts.convert_references reference/ \
  --pdf-backend mineru \
  --mineru-backend pipeline \
  --mineru-api-url http://127.0.0.1:30000
```

Use OpenAI-compatible MinerU client backends only when the user supplies a URL and token through their shell environment or existing service config:

```bash
uv run python -m scripts.convert_references reference/ \
  --pdf-backend mineru \
  --mineru-backend hybrid-http-client \
  --mineru-url http://127.0.0.1:30000 \
  --mineru-effort high
```

## End-to-End Test Workflow

1. Check the working tree and confirm no secret files are staged.
2. Copy one PDF into a temporary directory under `/tmp`, not into the repo.
3. Run a first-page smoke test with `--mineru-method txt` and `--mineru-arg=--start=0 --mineru-arg=--end=0`.
4. Inspect the generated Markdown for headings, formulas, tables, and images.
5. If the first-page test passes, run full conversion on the temporary copy.
6. Compare against PyMuPDF if deciding whether MinerU should be used for that source.
7. Report generated temporary paths, but do not commit smoke-test outputs.

## Quality Checks

A MinerU output is better than PyMuPDF for this project when it preserves more of:

- Section headings as Markdown headings.
- Display math as LaTeX blocks.
- Tables as structured HTML or Markdown tables.
- Figures as local asset links.
- Multi-column reading order as coherent paragraphs.

Watch for MinerU OCR/layout artifacts:

- Misspellings introduced by OCR.
- Over-spaced LaTeX tokens.
- Missing page boundary markers.
- Incorrect superscripts/subscripts around author affiliations or fractions.

## Security Rules

Never commit:

- MinerU/OpenXLab tokens.
- JWTs.
- Authorization or Bearer headers.
- Service credentials.
- Temporary parsed PDFs or generated assets unless the user explicitly asks.

Before committing related changes, search changed files for token-like material and verify `.forge/` remains uncommitted unless explicitly requested.
