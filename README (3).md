# VYOM+ — AI-Powered GST Invoice Intelligence

> Hacktober Fest 4 · Open Source AI Hackathon · Problem Statement 3
> Organized by Elevate · Qualifier Round · Team Submission

Drop in a blurry photo of a handwritten bill, a scanned PDF, or a messy Excel sheet, and get back a clean, validated GST invoice record. VYOM+ does that using only open-source AI, running entirely on your own infrastructure.

---

## Table of Contents

1. [What is VYOM+?](#1-what-is-vyom)
2. [The problem we're solving](#2-the-problem-were-solving)
3. [How it works at a glance](#3-how-it-works-at-a-glance)
4. [Our approach](#4-our-approach)
5. [What we set out to do](#5-what-we-set-out-to-do)
6. [Who it's for](#6-who-its-for)
7. [The open-source AI behind it](#7-the-open-source-ai-behind-it)
8. [Why we picked these tools](#8-why-we-picked-these-tools)
9. [What the AI does (and doesn't do)](#9-what-the-ai-does-and-doesnt-do)
10. [System architecture](#10-system-architecture)
11. [Project structure](#11-project-structure)
12. [How data flows through the system](#12-how-data-flows-through-the-system)
13. [The human-in-the-loop workflow](#13-the-human-in-the-loop-workflow)
14. [Tech stack](#14-tech-stack)
15. [Features](#15-features)
16. [How we plan to build it](#16-how-we-plan-to-build-it)
17. [What the output looks like](#17-what-the-output-looks-like)
18. [Where this can go next](#18-where-this-can-go-next)
19. [Open-source dependencies](#19-open-source-dependencies)
20. [Challenges and how we'll handle them](#20-challenges-and-how-well-handle-them)
21. [Deployment](#21-deployment)

---

## 1. What is VYOM+?

**VYOM+** is an open-source document intelligence platform built for the Indian GST ecosystem. It reads invoices in whatever shape they arrive (handwritten, scanned, printed or digital) and turns them into structured, validated records that accounting software can use straight away.

---

## 2. The problem we're solving

Indian businesses deal with millions of GST invoices every month, and no two look alike. Some arrive as Excel sheets, some as scans from an ancient printer, some as handwritten paper bills from small vendors, and plenty as phone photos.

Today, most of that gets typed in by hand, which means:

- **It's slow.** A single batch can eat hours.
- **It's error-prone.** Typos creep into GSTINs and totals.
- **It's expensive.** Skilled accountants spend their time on copy-paste work instead of analysis.
- **It's a compliance risk.** A wrong GSTIN, a CGST/IGST mismatch or a bad total can get a GST return rejected.

There's no affordable, open-source tool that handles the *full* range of invoice formats, handwritten ones especially, while also checking statutory GST rules and returning clean JSON. That's the gap VYOM+ fills.

---

## 3. How it works at a glance

VYOM+ is a multi-modal pipeline that:

1. Accepts invoices in six formats: `.xlsx`, `.csv`, digital `.pdf`, scanned `.pdf`, `.jpg`/`.jpeg` and `.png`
2. Detects the file type and sends it down the right processing path
3. Uses open-source vision-language models and OCR to pull out the invoice fields
4. Runs a deterministic GST rules engine over the result
5. Scores the confidence of every field and flags anything shaky for a human to check
6. Exports to JSON, CSV or Excel
7. Gives operators a web UI to upload, inspect, correct and export

Our guiding idea:

> **AI extracts. Deterministic code validates. Confidence decides. Humans resolve uncertainty.**

### A quick look at the interface

Upload is deliberately simple. Drop a file in, and the sidebar tells you what happens next: format detection, extraction, GST and totals verification, then confidence and exceptions.

![Upload screen: drag-and-drop area for PDF, JPG, PNG, XLSX and CSV files up to 20 MB, with a four-step "What happens next" panel](docs/screenshots/upload.png)

---

## 4. Our approach

The system has three layers.

### Layer 1: Ingestion and parsing

A file router looks at the input and picks the right parser:

- **Excel and CSV** go through `pandas` and `openpyxl`.
- **Digital PDFs** go through `PyMuPDF` / `pdfplumber` for text and tables.
- **Scanned PDFs and images** are rendered to images and sent through the OCR pipeline.

### Layer 2: AI extraction (the open-source core)

Two stages work together:

1. **PaddleOCR** pulls out raw text and rebuilds tables from scans and photos. It copes well with Indian scripts and low-quality images.
2. **Qwen2.5-VL (7B)**, an open-weight vision-language model, gets the document image plus the OCR text and returns structured JSON with every invoice field, the line items, and confidence scores.

Using both beats either one alone on real, messy Indian invoices.

### Layer 3: Validation, confidence and export

- A **GST rules engine** checks GSTIN format, supply-type consistency, line-item arithmetic and grand-total reconciliation.
- A **confidence scorer** combines OCR confidence, AI confidence and validation results.
- A **human-in-the-loop API** lets operators fix flagged fields and re-run validation.
- An **export layer** produces JSON, CSV or Excel.

---

## 5. What we set out to do

1. Correctly recognise and handle all six input formats
2. Extract the key GST fields: invoice number, date, supplier and buyer GSTIN, line items, HSN/SAC codes, CGST, SGST, IGST and totals
3. Handle handwritten invoices with an open-source OCR + VLM pipeline
4. Validate everything against 7+ deterministic GST rules
5. Produce field-level confidence scores and flag low-confidence extractions
6. Offer a human-in-the-loop correction interface
7. Export standardised JSON / CSV / Excel for accounting workflows
8. Use **only open-source or open-weight models**, with no proprietary API calls
9. Ship a working interface that evaluators can use to upload and inspect documents

---

## 6. Who it's for

| Who | What they get |
|-----|---------------|
| **SME accountants** | Less manual data entry from vendor bills |
| **CA firms** | Batch processing of client invoice bundles for GST filing |
| **ERP / accounting software** | A document-ingestion microservice they can plug in |
| **GST consultants** | A fast way to validate large invoice sets for audits |
| **Hackathon evaluators** | A simple place to upload samples and inspect the results |

### A real-world example

A small textile trader in Surat uploads a blurry photo of a handwritten purchase bill. VYOM+ reads it, extracts the supplier GSTIN, line items and GST components, works out whether the sale is intra-state, checks that CGST + SGST were applied correctly, scores its own confidence, and hands back a validated JSON record in under 10 seconds.

---

## 7. The open-source AI behind it

**Primary model: Qwen2.5-VL-7B-Instruct**
- Open-weight, Apache 2.0, from Alibaba Cloud
- 7B parameters; fits on a 16 GB GPU, or about 8 GB with quantization
- HuggingFace: [`Qwen/Qwen2.5-VL-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- Takes image(s) plus a text prompt and returns structured JSON
- Good at layout-aware reasoning over documents, tables and handwriting

**Main OCR: PaddleOCR (PP-OCRv4)**
- Apache 2.0, via `paddlepaddle` + `paddleocr`
- Strong on printed and semi-handwritten text, with table recognition through `PP-Structure`

**Fallback OCR: EasyOCR**
- Apache 2.0, lightweight, multi-language, runs on CPU only

**Text structuring: Qwen2.5-7B-Instruct**
- Used when the input is already digital text (CSV, Excel, digital PDF) and no vision is needed
- Apache 2.0

---

## 8. Why we picked these tools

### Why Qwen2.5-VL?

| Criteria | Qwen2.5-VL-7B | LayoutLMv3 | Donut | Tesseract + rules |
|----------|--------------|------------|-------|-------------------|
| Handles handwriting | ✅ Yes | ⚠️ Partial | ⚠️ Partial | ❌ Poor |
| No fine-tuning needed | ✅ Yes | ❌ Needs labeled data | ❌ Needs labeled data | ✅ Yes |
| Understands table layout | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| Produces JSON natively | ✅ Prompt-based | ❌ Needs post-processing | ✅ Yes | ❌ No |
| Open-source / open-weight | ✅ Apache 2.0 | ✅ MIT | ✅ MIT | ✅ Apache 2.0 |
| Works at 7B / quantized | ✅ Yes | ✅ Smaller | ✅ Smaller | ✅ CPU-only |
| Multi-language (Hindi labels) | ✅ Yes | ⚠️ English-focused | ⚠️ English-focused | ⚠️ Config needed |
| Diverse vendor layouts | ✅ Robust | ❌ Layout-sensitive | ❌ Template-sensitive | ❌ Brittle |

It's the only option that copes with our three hardest cases (handwritten bills, scans from low-end printers, and wildly varied vendor templates) without a labeled dataset to fine-tune on.

### Why PaddleOCR?

Its `PP-Structure` module rebuilds table cells from scanned documents. That makes a big difference to the VLM's accuracy on invoices with complex line-item tables, and it beats EasyOCR and Tesseract on Indian-language labels in headers and footers.

### Why open-source instead of paid APIs?

- **Privacy.** GST invoices hold sensitive financial data. Local inference keeps it all on-premises.
- **Cost.** Per-call API pricing adds up fast at scale. Open-weight models run free after setup.
- **Offline use.** It works in offices with unreliable internet, which is common across India.
- **It's the rule.** The hackathon problem statement requires open-source AI.

---

## 9. What the AI does (and doesn't do)

AI is the core of the system, not a bolt-on.

```
Document Image / Text
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 1: PaddleOCR (PP-OCRv4 + PP-Structure)                    │
│  • Detects text regions in the image                             │
│  • Reconstructs table cells and rows                             │
│  • Outputs: raw_text, table_markdown, ocr_confidence per region  │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (raw text + image)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 2: Qwen2.5-VL-7B-Instruct                                 │
│  Input:  [Document Image] + [OCR Text] + [Extraction Prompt]     │
│  Task:   Understand the document → return structured JSON        │
│  Output:                                                         │
│    • supplier / buyer: name, GSTIN, address                      │
│    • invoice_number, invoice_date, place_of_supply, IRN          │
│    • line_items[]: description, HSN/SAC, qty, rate, taxes        │
│    • tax_summary: CGST, SGST, IGST, cess, total_tax              │
│    • totals: subtotal, grand_total, amount_in_words              │
│    • confidence score per field (0.0 – 1.0)                      │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (structured JSON)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 3: Deterministic GST Validation Engine (no AI)            │
│  Rules-based cross-checks on the extracted values                │
└──────────────────────────────────────────────────────────────────┘
```

**The AI handles:** reading, understanding layout, resolving handwriting ambiguity, mapping fields to meaning, and assigning confidence.

**The AI does not handle:** GST rule checking, arithmetic verification or schema enforcement. Those are deterministic, so they're always predictable.

---

## 10. System architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VYOM+ System Architecture                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐        ┌──────────────────────────────────────────────┐  │
│   │   User /    │        │              FastAPI Backend                  │  │
│   │  Evaluator  │◄──────►│                                              │  │
│   │  (Browser)  │  REST  │  ┌──────────┐  ┌───────────┐  ┌──────────┐  │  │
│   └─────────────┘        │  │  Upload  │  │  Invoice  │  │  Export  │  │  │
│                           │  │  Router  │  │   CRUD   │  │  Router  │  │  │
│   ┌─────────────┐        │  └────┬─────┘  └─────┬─────┘  └────┬─────┘  │  │
│   │  Streamlit  │        │       │               │              │        │  │
│   │  Demo UI   │◄───────►│  ┌────▼──────────────▼──────────────▼─────┐  │  │
│   └─────────────┘        │  │           Processing Pipeline           │  │  │
│                           │  │                                         │  │  │
│                           │  │  File Router → [Excel/CSV/PDF/Image]   │  │  │
│                           │  │       │                                 │  │  │
│                           │  │  ┌────▼─────────────────────────────┐  │  │  │
│                           │  │  │     AI Extraction Engine         │  │  │  │
│                           │  │  │  PaddleOCR → Qwen2.5-VL-7B      │  │  │  │
│                           │  │  └────────────────┬─────────────────┘  │  │  │
│                           │  │                   │                     │  │  │
│                           │  │  ┌────────────────▼─────────────────┐  │  │  │
│                           │  │  │    GST Validation Engine         │  │  │  │
│                           │  │  │  (Deterministic Rules)           │  │  │  │
│                           │  │  └────────────────┬─────────────────┘  │  │  │
│                           │  │                   │                     │  │  │
│                           │  │  ┌────────────────▼─────────────────┐  │  │  │
│                           │  │  │  Confidence Scorer + HITL Flag   │  │  │  │
│                           │  │  └────────────────┬─────────────────┘  │  │  │
│                           │  └────────────────────┼─────────────────┘  │  │
│                           │                       ▼                     │  │
│                           │         ┌─────────────────────────┐         │  │
│                           │         │    SQLite / PostgreSQL  │         │  │
│                           │         │    (Invoice Records)    │         │  │
│                           │         └─────────────────────────┘         │  │
│                           └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. Project structure

```
vyom-invoice-ai/
├── backend/
│   └── app/
│       ├── main.py                  ← FastAPI app, CORS, lifespan
│       ├── config.py                ← Pydantic Settings (env vars)
│       ├── routers/
│       │   ├── upload.py            ← POST /api/upload
│       │   ├── invoice.py           ← GET/PUT /api/documents
│       │   └── export.py            ← GET /api/documents/{id}/json|csv|excel
│       ├── services/
│       │   ├── file_router.py       ← Detects type, dispatches to processor
│       │   ├── excel_processor.py   ← pandas + openpyxl parser
│       │   ├── csv_processor.py     ← pandas CSV parser
│       │   ├── pdf_processor.py     ← PyMuPDF (digital) + image render (scanned)
│       │   ├── image_processor.py   ← image pre-processing (OpenCV)
│       │   ├── ocr_service.py       ← PaddleOCR + EasyOCR fallback
│       │   ├── ai_extractor.py      ← Qwen2.5-VL-7B inference (HuggingFace)
│       │   ├── validator.py         ← Deterministic GST rules engine
│       │   ├── confidence.py        ← Multi-signal confidence scorer
│       │   └── exporter.py          ← JSON / CSV / Excel export
│       ├── schemas/
│       │   └── invoice.py           ← Pydantic v2 models (InvoiceRecord)
│       ├── database/
│       │   ├── models.py            ← SQLAlchemy ORM models
│       │   ├── database.py          ← Engine + session factory
│       │   └── crud.py              ← DB read/write operations
│       └── utils/
│           └── gst_rules.py         ← GSTIN regex, state codes, GST rate set
├── frontend/                        ← Next.js web interface
│   ├── pages/
│   │   ├── index.tsx                ← Dashboard
│   │   ├── upload.tsx               ← Upload page
│   │   ├── documents/[id].tsx       ← Invoice review + HITL correction
│   │   └── analytics.tsx            ← Stats + charts
│   └── components/
│       ├── InvoiceCard.tsx
│       ├── LineItemsTable.tsx
│       ├── ValidationBadge.tsx
│       └── ConfidenceBar.tsx
├── demo_ui/
│   └── streamlit_app.py             ← Streamlit demo (for evaluators)
├── docs/
│   └── screenshots/                 ← UI screenshots used in this README
└── sample_invoices/
    ├── handwritten/
    ├── scanned_pdf/
    ├── digital_pdf/
    ├── excel/
    ├── csv/
    └── printed/
```

### Who talks to whom

| Component | Input | Output | AI? |
|-----------|-------|--------|-----|
| `file_router.py` | filename, bytes | file type string | No |
| `ocr_service.py` | image bytes | raw text, table markdown, confidence | **PaddleOCR** |
| `ai_extractor.py` | image + OCR text | InvoiceRecord JSON | **Qwen2.5-VL** |
| `validator.py` | InvoiceRecord dict | same dict + validation result | No |
| `confidence.py` | InvoiceRecord dict | same dict + confidence scores | No |
| `exporter.py` | InvoiceRecord dict | JSON / CSV bytes / Excel bytes | No |

---

## 12. How data flows through the system

```
User uploads file (any format)
         │
         ▼
[1] File Router
     Detects: xlsx / csv / pdf / jpg / png
         │
    ┌────┴─────────────────────────────────────┐
    │                                          │
    ▼ (Excel / CSV)                            ▼ (PDF / Image)
[2A] Tabular Processor                   [2B] Image Pipeline
  pandas reads rows/columns               PyMuPDF renders pages to images
  Normalizes column names                 OpenCV: deskew, denoise, enhance
  Maps to invoice schema                  contrast-boost for faded scans
         │                                         │
         │                                         ▼
         │                                [3] PaddleOCR (PP-OCRv4)
         │                                  Text region detection
         │                                  Table cell reconstruction
         │                                  Per-word confidence scores
         │                                         │
         └─────────────┬───────────────────────────┘
                       │
                       ▼
              [4] Qwen2.5-VL-7B-Instruct
                 Input: image + OCR text + GST prompt + target schema
                 Output: all fields, line items, per-field confidence
                       │
                       ▼
              [5] Pydantic Validation
                 Strict typing, defaults for null fields
                       │
                       ▼
              [6] GST Rules Engine (validator.py)
                 ✓ GSTIN 15-char format + state code
                 ✓ Supply type: intra vs inter-state
                 ✓ CGST/SGST vs IGST consistency
                 ✓ Line item: Qty × Rate − Discount = Taxable Value
                 ✓ Per-item: Taxable × GST% = CGST+SGST or IGST
                 ✓ Tax total: CGST+SGST+IGST+Cess = Total Tax
                 ✓ Grand Total = Taxable + Total Tax + Round-off
                 Output: valid / warning / needs_review / invalid
                       │
                       ▼
              [7] Confidence Scorer
                 Combines OCR + AI + validation signals
                 Flags fields below the 70% threshold for review
                       │
                       ▼
              [8] Database (SQLite → PostgreSQL)
                       │
                       ▼
              [9] API Response + Export (JSON / CSV / Excel)
```

---

## 13. The human-in-the-loop workflow

VYOM+ doesn't just extract and move on. It reasons about its own uncertainty and decides when to pull a human in.

```
                    ┌─────────────────────────────┐
                    │      Document Uploaded       │
                    └──────────────┬──────────────┘
                                   ▼
                    ┌─────────────────────────────┐
                    │ AI Pipeline Runs (Stages 1-4)│
                    └──────────────┬──────────────┘
                    ┌──────────────▼──────────────┐
                    │  Confidence ≥ 90%?           │
                    └──────┬───────────────┬───────┘
                   YES     │               │  NO
                           ▼               ▼
              ┌────────────────┐   ┌──────────────────────────┐
              │  Auto-Approved │   │ Flagged for Human Review │
              │  status=valid  │   │ status=needs_review      │
              └────────────────┘   │ Low-confidence fields    │
                                   │ highlighted in the UI    │
                                   └──────────┬───────────────┘
                                              ▼
                                   ┌──────────────────────────┐
                                   │ Operator corrects a field│
                                   │ PUT /api/documents/{id}  │
                                   └──────────┬───────────────┘
                                              ▼
                                   ┌──────────────────────────┐
                                   │ Re-validation + new      │
                                   │ confidence calculation   │
                                   └──────────┬───────────────┘
                                              ▼
                                   ┌──────────────────────────┐
                                   │ Record updated in DB;    │
                                   │ correction logged        │
                                   └──────────────────────────┘
```

That's a full *perceive → reason → act → adapt* loop. Documents at 90% confidence or above are auto-approved, and anything below the 70% threshold is flagged for a person to check.

---

## 14. Tech stack

| Layer | Technology | Version | License | Role |
|-------|-----------|---------|---------|------|
| Backend framework | FastAPI | 0.115+ | MIT | REST API, async handling |
| Vision model | Qwen2.5-VL-7B-Instruct | latest | Apache 2.0 | Invoice understanding from images |
| Text model | Qwen2.5-7B-Instruct | latest | Apache 2.0 | Structuring digital text |
| Primary OCR | PaddleOCR (PP-OCRv4) | 2.8+ | Apache 2.0 | Text + tables from scans |
| Fallback OCR | EasyOCR | 1.7+ | Apache 2.0 | CPU-only multi-language OCR |
| Model inference | HuggingFace Transformers | 4.46+ | Apache 2.0 | Run Qwen2.5-VL locally |
| Quantization | BitsAndBytes / llama.cpp | latest | MIT | 4-bit inference for less GPU memory |
| PDF parsing | PyMuPDF (fitz) | 1.24+ | AGPL 3.0 | Digital PDF text; scanned PDF → images |
| Image processing | OpenCV | 4.10+ | Apache 2.0 | Deskew, denoise, contrast |
| Schemas | Pydantic v2 | 2.8+ | MIT | Strict validation |
| ORM | SQLAlchemy | 2.0+ | MIT | Database abstraction |
| Database (MVP) | SQLite | built-in | Public domain | Invoice storage |
| Database (prod) | PostgreSQL | 16+ | PostgreSQL | Scalable storage |
| Spreadsheets | pandas + openpyxl | latest | BSD | Excel/CSV parsing |
| Export | reportlab, fpdf2 | latest | BSD/LGPL | PDF export |
| Demo UI | Streamlit | 1.40+ | Apache 2.0 | Evaluator interface |
| Frontend | Next.js + React | 15+ | MIT | Full web interface |
| Styling | Tailwind CSS | 3+ | MIT | Frontend styling |
| Runtime | Python | 3.11 | PSF | Backend runtime |

---

## 15. Features

### Core features (MVP)

| # | Feature | What it does |
|---|---------|--------------|
| 1 | Multi-format upload | Accepts xlsx, csv, pdf, jpg, png via REST API and Streamlit UI |
| 2 | Smart file routing | Detects the input type and picks the right pipeline |
| 3 | Handwritten invoice OCR | PaddleOCR + Qwen2.5-VL read handwritten bills |
| 4 | Structured extraction | All GST fields land in one standard JSON schema |
| 5 | GSTIN validation | 15-character format, state code and checksum |
| 6 | Supply type check | Intra-state (CGST+SGST) vs inter-state (IGST) consistency |
| 7 | Line-item arithmetic | Qty × Rate − Discount = Taxable Value |
| 8 | Tax total validation | CGST + SGST + IGST + Cess = Total Tax |
| 9 | Grand total reconciliation | Taxable + Total Tax + Round-off = Grand Total |
| 10 | Confidence scoring | Per-field (0–100%) plus an overall score |
| 11 | Review interface | Flagged documents appear with fields ready to correct |
| 12 | Dot-notation corrections | Fix any field via paths like `supplier.gstin` or `items[0].unit_price` |
| 13 | JSON export | Machine-readable, standardised |
| 14 | CSV export | Line-item CSV for spreadsheet workflows |
| 15 | Excel export | Multi-tab workbook with summary and line items |
| 16 | Dashboard stats | Totals, status breakdown, average confidence, recent invoices |
| 17 | Document history | Every processed invoice stored and searchable |
| 18 | Status filtering | Filter by valid, warning, needs_review or invalid |

### The dashboard

The workspace overview shows documents processed, how many validated, how many need review, and average confidence. A "Needs your attention" panel surfaces anything awaiting verification, and recent documents sit alongside it with their status.

![Workspace overview dashboard showing 2 documents processed, 2 validated, 0 needing review and 99% average confidence, with sample_invoice.xlsx and sample_invoice.csv listed as validated](docs/screenshots/dashboard.png)

### Stretch goals

| # | Feature |
|---|---------|
| 19 | Batch upload (ZIP of many invoices) |
| 20 | PDF export with an IRN QR code for e-invoice verification |
| 21 | Duplicate invoice detection |
| 22 | LoRA fine-tuning pipeline on a custom GST invoice dataset |

---

## 16. How we plan to build it

**Phase 1: Foundation (Day 1)**
Set up the FastAPI backend, Pydantic schemas and SQLite. Build the Excel and CSV parsers. Define the universal invoice JSON schema and the first GST validation rules.

**Phase 2: OCR and digital PDFs (Day 1–2)**
Integrate `PyMuPDF` for digital PDFs and `PaddleOCR` for images and scans. Build the OpenCV pre-processing pipeline (deskew, denoise, binarize) and the `file_router.py` dispatcher.

**Phase 3: The vision-language model (Day 2)**
Load `Qwen2.5-VL-7B-Instruct` through HuggingFace Transformers. Design the GST extraction prompt with the schema, wire up the PaddleOCR → Qwen2.5-VL pipeline, and test across all six input types.

**Phase 4: Validation and confidence (Day 2–3)**
Build the 7-rule GST validation engine and the multi-signal confidence scorer. Add the human-in-the-loop correction API with re-validation.

**Phase 5: Export and UI (Day 3)**
Implement the JSON, CSV and Excel exporters, build the Streamlit demo UI, and (as a stretch) the Next.js frontend. Test end-to-end with every sample invoice type.

**Phase 6: Polish and evaluation (Day 3–4)**
Add sample invoices per format, measure accuracy on held-out invoices (field extraction accuracy, GSTIN validation pass rate), then finish the README, demo video and submission.

---

## 17. What the output looks like

For every processed invoice:

```json
{
  "document": {
    "id": "3f8a1b2c-...",
    "filename": "purchase_bill_oct_2026.jpg",
    "source_type": "image",
    "upload_time": "2026-10-10T09:15:00Z",
    "page_count": 1
  },
  "supplier": {
    "name": "Shree Textiles Pvt Ltd",
    "gstin": "27AABCS1234A1Z5",
    "address": "Mumbai, Maharashtra",
    "state_code": "27"
  },
  "buyer": {
    "name": "Fashion Hub Surat",
    "gstin": "24AAFCF5678B1Z3",
    "address": "Surat, Gujarat",
    "state_code": "24"
  },
  "invoice": {
    "invoice_number": "STX/2026/1042",
    "invoice_date": "2026-10-05",
    "place_of_supply": "Gujarat",
    "reverse_charge": false,
    "supply_type": "inter-state"
  },
  "items": [
    {
      "description": "Cotton Fabric (Grey)",
      "hsn_sac": "5208",
      "quantity": 100,
      "unit": "meters",
      "unit_price": 250.00,
      "discount": 0,
      "taxable_value": 25000.00,
      "gst_rate": 5,
      "cgst": null,
      "sgst": null,
      "igst": 1250.00,
      "total": 26250.00
    }
  ],
  "tax": {
    "taxable_amount": 25000.00,
    "cgst": null,
    "sgst": null,
    "igst": 1250.00,
    "cess": 0,
    "total_tax": 1250.00
  },
  "totals": {
    "subtotal": 25000.00,
    "grand_total": 26250.00,
    "amount_in_words": "Twenty Six Thousand Two Hundred Fifty Rupees Only"
  },
  "validation": {
    "status": "valid",
    "gstin_supplier_valid": true,
    "gstin_buyer_valid": true,
    "tax_calculation_valid": true,
    "line_items_valid": true,
    "grand_total_valid": true,
    "supply_type_consistent": true,
    "errors": [],
    "warnings": []
  },
  "confidence": {
    "overall": 0.93,
    "invoice_number": 0.97,
    "invoice_date": 0.98,
    "supplier_gstin": 0.91,
    "buyer_gstin": 0.89,
    "items": 0.95,
    "total": 0.98,
    "review_required": false,
    "low_confidence_fields": []
  }
}
```

### System-level deliverables

- ✅ A working REST API, auto-documented at `/docs`
- ✅ A Streamlit evaluator interface for upload and inspection
- ✅ A dashboard with processing statistics
- ✅ Export in JSON, CSV and Excel
- ✅ Sample invoices tested across all six formats
- ✅ Accuracy metrics on a held-out test set

---

## 18. Where this can go next

### Scaling up

| Goal | Change |
|------|--------|
| Storage | Swap SQLite for PostgreSQL with connection pooling |
| Inference | Serve Qwen2.5-VL via vLLM for batched GPU inference (about 10x throughput) |
| API | Add Celery + Redis for async processing of large batches |
| Packaging | Docker + docker-compose for reproducible deployment |
| Hosting | A GPU cloud VM (AWS/GCP/Azure) or RunPod |

### Roadmap

| Feature | Notes |
|---------|-------|
| Fine-tuning on Indian GST data | LoRA fine-tune Qwen2.5-VL on a labeled invoice dataset to improve regional vendor formats |
| Batch ZIP upload | Process 50+ invoices in parallel and return batch results |
| E-invoice QR verification | Decode IRN QR codes and cross-check with the NIC portal |
| GSTR-2A reconciliation | Match extracted purchase invoices against GSTR-2A data |
| Multi-tenant SaaS | Organisation accounts, role-based access, audit logs |
| Mobile app | Camera capture to instant scan via a mobile PWA |
| Multilingual invoices | Hindi, Gujarati, Tamil and other regional languages in headers and footers |
| ERP integrations | Webhooks for Tally Prime, Zoho Books and QuickBooks |

---

## 19. Open-source dependencies

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| `transformers` | 4.46+ | Apache 2.0 | Run Qwen2.5-VL-7B |
| `paddlepaddle` | 2.6+ | Apache 2.0 | PaddleOCR backend |
| `paddleocr` | 2.8+ | Apache 2.0 | OCR + PP-Structure tables |
| `easyocr` | 1.7+ | Apache 2.0 | Fallback OCR |
| `pymupdf` | 1.24+ | AGPL 3.0 | PDF extraction and rendering |
| `opencv-python` | 4.10+ | Apache 2.0 | Image preprocessing |
| `Pillow` | 10+ | MIT-CMU | Image handling |
| `fastapi` | 0.115+ | MIT | REST API |
| `uvicorn` | 0.30+ | BSD | ASGI server |
| `pydantic` | 2.8+ | MIT | Data validation |
| `pydantic-settings` | 2.5+ | MIT | Env-based config |
| `sqlalchemy` | 2.0+ | MIT | ORM |
| `pandas` | 2.2+ | BSD | CSV/Excel processing |
| `openpyxl` | 3.1+ | MIT | Excel read/write |
| `streamlit` | 1.40+ | Apache 2.0 | Demo UI |
| `python-multipart` | 0.0.9+ | Apache 2.0 | File upload parsing |
| `python-dotenv` | 1.0+ | BSD | `.env` loading |
| `reportlab` | 4.2+ | BSD | PDF export |
| `fpdf2` | 2.8+ | LGPL | Lightweight PDF generation |
| `bitsandbytes` | 0.44+ | MIT | 4-bit quantization |
| `accelerate` | 0.34+ | Apache 2.0 | Multi-GPU inference |
| `qrcode` | 8.0+ | BSD | QR codes (e-invoice) |
| `pytest` | 8.0+ | MIT | Testing |

Everything here is open-source. No proprietary model APIs are used.

---

## 20. Challenges and how we'll handle them

| # | Challenge | Risk | Plan |
|---|-----------|------|------|
| 1 | **Handwritten invoice quality** (faded ink, low contrast, uneven spacing) | High | Multi-step enhancement (deskew, denoise, adaptive thresholding) before OCR; Qwen2.5-VL reads handwriting natively |
| 2 | **GPU memory** (7B needs about 14 GB VRAM) | High | 4-bit BitsAndBytes quantization (about 4–5 GB); fall back to Qwen2.5-3B-Instruct if needed |
| 3 | **Hundreds of vendor layouts** | High | Prompt-based extraction needs no templates, so layout doesn't matter |
| 4 | **OCR errors on regional fonts** | Medium | PaddleOCR has multilingual models; English fields still extract fine when headers are in another script |
| 5 | **Misaligned table cells** (merged cells in line-item tables) | Medium | PP-Structure handles merged cells; the VLM cross-checks against the full image |
| 6 | **JSON hallucination** (plausible but wrong values) | Medium | Strict Pydantic schema, deterministic arithmetic checks, and confidence flags |
| 7 | **Slow inference on CPU** | Medium | GPU preferred; for CPU-only use EasyOCR + Qwen2.5-3B text-only as a fast path |
| 8 | **Missing fields** (e.g. no buyer GSTIN) | Low | Fields are `Optional`; missing non-critical fields produce a warning, not an error |
| 9 | **Invented values for blank areas** | Low | Prompt says "return null if not visible"; validation and confidence catch the rest |
| 10 | **GSTIN character confusion** (O vs 0, I vs 1, S vs 5) | Low | PaddleOCR is accurate on printed characters; the VLM prompt includes disambiguation rules |

---

## 21. Deployment

VYOM+ runs in different modes depending on your hardware, from a free cloud notebook to a production server.

### Mode 1: Hackathon demo (recommended)

A 7B model is too heavy for most laptops, so we split the setup: the app runs locally and the AI runs on a free cloud GPU.

```
┌──────────────────────────────────────────────────────────┐
│                  HACKATHON DEMO SETUP                    │
├──────────────────────────────────────────────────────────┤
│   Evaluator's Browser                                    │
│          ▼                                               │
│   Streamlit / Next.js UI  ◄──── runs on your laptop     │
│          ▼                                               │
│   FastAPI Backend         ◄──── runs on your laptop     │
│   (routing, validation, export, DB)                      │
│          │  HTTP POST (image + text)                     │
│          ▼                                               │
│   AI Inference Server  ◄──── Google Colab / HF Spaces   │
│   (Qwen2.5-VL-7B +          (free T4/P100 GPU,          │
│    PaddleOCR)                exposed via ngrok)          │
│          │  returns JSON                                 │
│          ▼                                               │
│   FastAPI finishes validation and responds               │
└──────────────────────────────────────────────────────────┘
```

**On demo day:**
1. Open Google Colab and load Qwen2.5-VL-7B on the free T4 GPU.
2. Run a small FastAPI inference endpoint inside Colab.
3. Use `ngrok` to get a public URL for it.
4. Point your local FastAPI's `AI_INFERENCE_URL` at that URL.
5. Run Streamlit locally. The evaluator uploads an invoice, it goes to Colab, and the result comes back.

**Cost: $0.** The Colab free tier is enough for a full demo session.

### Mode 2: Cloud production

```
Users ──► Nginx Load Balancer
                │
        ┌───────┴────────┐
        ▼                ▼
  FastAPI Pods      Streamlit UI
  (Docker/K8s)
        │
        ▼
  Redis Queue ──► Celery Workers (async invoice jobs)
                        │
                        ▼
               vLLM Inference Server
               (Qwen2.5-VL on A100 GPU)
                        │
                        ▼
               PostgreSQL Database
```

**Affordable GPU options:**

| Platform | GPU | Cost | Best for |
|----------|-----|------|----------|
| Google Colab Free | T4 16GB | $0 | Demo / testing |
| HuggingFace Spaces | T4 | Free tier | Public demo hosting |
| RunPod | RTX 4090 | ~$0.44/hr | Hackathon final |
| Vast.ai | RTX 3090 | ~$0.20/hr | Budget production |
| AWS EC2 `g4dn.xlarge` | T4 16GB | ~$0.53/hr | Scalable deployment |

### Environment configuration (`.env`)

```env
# AI inference endpoint: local GPU or a remote Colab/cloud URL
AI_INFERENCE_URL=http://localhost:8001
# AI_INFERENCE_URL=https://xxxx.ngrok.io   # Colab tunnel for the demo

# Model settings
MODEL_NAME=Qwen/Qwen2.5-VL-7B-Instruct
USE_4BIT_QUANTIZATION=true                 # cuts VRAM from ~14GB to ~5GB

# App settings
DATABASE_URL=sqlite:///./vyom_invoices.db
UPLOAD_DIR=./uploads
MAX_FILE_SIZE_MB=20
CONFIDENCE_THRESHOLD_REVIEW=0.70
```

---

## In short

VYOM+ tackles a real, high-impact problem in Indian GST compliance with a carefully designed open-source pipeline:

- 🔍 **PaddleOCR** reads text and tables from any scan quality
- 🧠 **Qwen2.5-VL-7B** understands layout and handwriting
- ✅ **Deterministic GST rules** handle statutory validation reliably
- 🔄 **Human-in-the-loop review** resolves whatever the system isn't sure about
- 📤 **Multi-format export** feeds straight into accounting workflows

The result: a blurry photo of a handwritten vendor bill becomes a validated, machine-readable GST record, with no proprietary APIs, no subscription cost, and no data leaving your infrastructure.

---

*Hacktober Fest 4 · Open Source AI Hackathon · Qualifier Submission · Problem Statement 3*
*All technologies used are open-source or open-weight. No proprietary AI APIs.*
