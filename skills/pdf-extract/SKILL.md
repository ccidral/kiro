---
name: pdf-extract
description: Extract text, tables, and metadata from PDF files using pdfplumber. Use when converting PDFs to Markdown, extracting text content from PDF documents, reading tables from PDFs, or when the user mentions PDF extraction, PDF to text, or PDF to Markdown conversion.
compatibility: Requires Python 3.10+ and pdfplumber. Install with `pip install pdfplumber` in the workspace .venv.
metadata:
  author: ccidral
  version: "1.0"
---

# PDF Extraction Skill

Extract content from PDF files using pdfplumber for high-fidelity text extraction.

## Setup

Requires a Python virtual environment with pdfplumber at the workspace root:

```bash
python3 -m venv .venv
.venv/bin/pip install pdfplumber
```

## Usage

```bash
.venv/bin/python .kiro/skills/pdf-extract/scripts/pdfx <file> [--pages RANGE]
```

### Examples

```bash
# Extract entire PDF
.venv/bin/python .kiro/skills/pdf-extract/scripts/pdfx document.pdf

# Extract specific pages
.venv/bin/python .kiro/skills/pdf-extract/scripts/pdfx document.pdf --pages 4-10

# Extract individual pages
.venv/bin/python .kiro/skills/pdf-extract/scripts/pdfx document.pdf --pages 1,3,5
```

### Page selection

- Single page: `--pages 3`
- Range: `--pages 2-5`
- Comma-separated: `--pages 1,3,5`
- Combined: `--pages 1,3-5,8`
- Omit for all pages.

Pages are 1-indexed.

## Output format

Plain text, page by page, separated by `--- PAGE N ---` markers. No formatting, no chrome. Errors go to stderr.

## Notes

- pdfplumber does NOT do OCR. Scanned-image pages return `[NO TEXT]`.
- Tables appear as space-aligned text in the output (same as how pdfplumber's `extract_text()` renders them).
- For PDF-to-Markdown conversion: run the tool, then compose Markdown from the output — convert headings to `#` syntax, bullets to lists, space-aligned tables to Markdown tables.
