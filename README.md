# VYOM+ — End-to-End AI-Powered GST Invoice Intelligence System

> **HacktoberFest Hack Day · Open Source AI Hackathon · Problem Statement 3**  

---

## Table of Contents

1. [Project Name](#1-project-name)
2. [Problem Statement](#2-problem-statement)
3. [Project Overview](#3-project-overview)
4. [Proposed Solution](#4-proposed-solution)
5. [Objectives](#5-objectives)
6. [Target Users / Use Case](#6-target-users--use-case)
7. [Open-Source AI Technology Selected](#7-open-source-ai-technology-selected)
8. [Why This Technology Was Selected](#8-why-this-technology-was-selected)
9. [AI's Role in the System](#9-ais-role-in-the-system)
10. [System Architecture](#10-system-architecture)
11. [Component-Level Architecture](#11-component-level-architecture)
12. [Data / Information Flow](#12-data--information-flow)
13. [Agentic Workflow](#13-agentic-workflow)
14. [Technology Stack](#14-technology-stack)
15. [Expected Features](#15-expected-features)
16. [Implementation Approach](#16-implementation-approach)
17. [Expected Final Output](#17-expected-final-output)
18. [Future Scope / Scalability](#18-future-scope--scalability)
19. [Open-Source Dependencies / Components](#19-open-source-dependencies--components)
20. [Expected Challenges and Mitigation](#20-expected-challenges-and-mitigation)

---

## 1. Project Name

### VYOM+ — End-to-End AI-Powered GST Invoice Intelligence System

**VYOM+** is an open-source, AI-driven document intelligence platform purpose-built for the Indian GST ecosystem. It accepts real-world invoice documents in any common format — handwritten, scanned, printed, or digital — and converts them into clean, validated, machine-readable financial records suitable for downstream accounting workflows.

---

## 2. Problem Statement

Indian businesses handle **millions of GST invoices every month** across wildly different formats — Excel sheets sent by suppliers, scanned PDFs from old printers, handwritten paper bills from small vendors, digitally generated PDFs, and image captures from mobile phones.

Manually keying this information into accounting systems is:

- ⏱ **Slow** — data entry takes hours per batch
- ❌ **Error-prone** — human transcription introduces arithmetic and GSTIN mistakes
- 💸 **Costly** — skilled accountants spend time on repetitive extraction instead of analysis
- 📋 **Non-compliant risk** — GSTIN format errors, CGST/IGST mismatches, and wrong totals lead to GST return rejections

No existing affordable, open-source tool handles the **full spectrum** of invoice formats — especially **handwritten invoices** — while also performing **statutory GST validation** and returning **structured JSON** ready for accounting software.

**This is the gap VYOM+ fills.**

---

## 3. Project Overview

VYOM+ is a **multi-modal document intelligence pipeline** that:

1. Accepts invoice documents in **6 formats**: `.xlsx`, `.csv`, `.pdf` (digital), `.pdf` (scanned), `.jpg`/`.jpeg`, `.png`
2. Routes each document through an **appropriate processing pipeline** based on detected type
3. Uses **open-source Vision-Language Models** and **OCR engines** to extract invoice fields
4. Applies a **deterministic GST rules engine** to validate extracted data
5. Scores **field-level confidence** and flags uncertain extractions for human review
6. Exports results as **JSON, CSV, or Excel**
7. Provides a **web UI** for evaluators and operators to upload, inspect, correct, and export invoices

The system is designed around a core principle:

> **"AI extracts. Deterministic code validates. Confidence decides. Humans resolve uncertainty."**

---

## 4. Proposed Solution

VYOM+ proposes a **three-layer architecture**:

### Layer 1 — Document Ingestion & Parsing
A smart file router detects the input type and sends it to the correct parser:
- **Structured files** (Excel, CSV) → `pandas` + `openpyxl` tabular parsing
- **Digital PDFs** → `PyMuPDF` / `pdfplumber` text & table extraction  
- **Scanned PDFs & Images** → rendered to images → multi-stage OCR pipeline

### Layer 2 — AI-Powered Extraction Engine (Open-Source Core)
A **two-stage open-source AI pipeline**:

1. **PaddleOCR** — extracts raw text and reconstructs tables from scanned/image documents with high accuracy on Indian scripts and low-quality scans
2. **Qwen2.5-VL (7B)** — an open-weight Vision-Language Model that receives the document image + OCR text and returns structured JSON with all invoice fields, line items, and confidence scores

This combination outperforms both pure OCR and pure VLM approaches for real-world, messy Indian invoices.

### Layer 3 — Validation, Confidence & Export
- A **deterministic GST rules engine** cross-checks GSTIN format, supply type consistency, line-item arithmetic, and grand total reconciliation
- A **multi-signal confidence scorer** aggregates OCR confidence + AI confidence + validation pass rate
- A **human-in-the-loop API** allows operators to correct flagged fields and re-trigger validation
- **Export layer** produces JSON, CSV, or Excel output

---

## 5. Objectives

| # | Objective |
|---|-----------|
| 1 | Accept and correctly identify all 6 supported invoice input formats |
| 2 | Extract all key GST invoice fields: invoice number, date, supplier GSTIN, buyer GSTIN, line items, HSN/SAC codes, CGST, SGST, IGST, totals |
| 3 | Handle handwritten invoices using open-source OCR + VLM pipeline |
| 4 | Validate extracted data against 7+ deterministic GST statutory rules |
| 5 | Produce field-level confidence scores and flag low-confidence extractions |
| 6 | Provide a human-in-the-loop correction interface |
| 7 | Export standardized JSON / CSV / Excel output for downstream accounting |
| 8 | Use **only open-source / open-weight AI models** — no proprietary API calls |
| 9 | Provide a working evaluator interface for document upload and inspection |

---

## 6. Target Users / Use Case

| User | Use Case |
|------|----------|
| **SME Accountants** | Automate manual GST invoice data entry from vendor bills |
| **CA Firms** | Batch-process client invoice bundles for GST return filing |
| **ERP / Accounting Software** | Plug in as a document ingestion microservice |
| **GST Consultants** | Validate large sets of invoices for compliance audits |
| **Hackathon Evaluators** | Upload sample invoices and inspect extracted structured data |
| **VYOM+ Platform** | Use as the AI document intelligence backend for accounting workflows |

### Real-World Scenario

> A small textile trader in Surat uploads a blurry photo of a handwritten purchase bill from a local supplier. VYOM+ reads the image, extracts the supplier GSTIN, line items, and GST components, detects that the supplier and buyer are in the same state (intra-state), verifies that CGST + SGST was correctly applied, scores confidence, and returns a validated JSON record — all in under 10 seconds.

---

## 7. Open-Source AI Technology Selected

### Primary: **Qwen2.5-VL-7B-Instruct**
- **Model family:** Qwen2.5-VL (Alibaba Cloud, open-weight, Apache 2.0 license)
- **Model size:** 7B parameters (fits on a single GPU with 16GB VRAM or via quantization on 8GB)
- **HuggingFace:** [`Qwen/Qwen2.5-VL-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- **Input:** Image(s) + text prompt → structured JSON output
- **Key capability:** Layout-aware visual reasoning over documents, tables, and handwritten text

### Secondary OCR: **PaddleOCR (PP-OCRv4)**
- **Library:** `paddlepaddle` + `paddleocr`
- **License:** Apache 2.0
- **Key capability:** State-of-the-art OCR for printed and semi-handwritten documents with table structure recognition (`PP-Structure`)

### Fallback OCR: **EasyOCR**
- **License:** Apache 2.0
- **Key capability:** Lightweight multi-language OCR, runs CPU-only, no GPU needed

### LLM for Text Structuring: **Qwen2.5-7B-Instruct** (text-only fallback)
- Used when document is already digital text (CSV/Excel/digital PDF) and no vision is needed
- Structures raw text into the invoice JSON schema
- **License:** Apache 2.0

---

## 8. Why This Technology Was Selected

### Why Qwen2.5-VL over other models?

| Criteria | Qwen2.5-VL-7B | LayoutLMv3 | Donut | Tesseract + Rules |
|----------|--------------|------------|-------|-------------------|
| Handles handwriting | ✅ Yes | ⚠️ Partial | ⚠️ Partial | ❌ Poor |
| No fine-tuning needed | ✅ Yes | ❌ Needs labeled data | ❌ Needs labeled data | ✅ Yes |
| Understands table layout | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| Produces JSON natively | ✅ Prompt-based | ❌ Needs post-processing | ✅ Yes | ❌ No |
| Open-source / open-weight | ✅ Apache 2.0 | ✅ MIT | ✅ MIT | ✅ Apache 2.0 |
| Works on 7B / quantized | ✅ Yes | ✅ Smaller | ✅ Smaller | ✅ CPU-only |
| Multi-language (Hindi labels) | ✅ Yes | ⚠️ English-focused | ⚠️ English-focused | ⚠️ Config needed |
| Diverse vendor layouts | ✅ Robust | ❌ Layout-sensitive | ❌ Template-sensitive | ❌ Brittle |

**Qwen2.5-VL-7B wins** because it is the only model that can handle all three of our hardest input categories — handwritten bills, scanned invoices from low-end printers, and varied vendor templates — without needing a labeled dataset to fine-tune.

### Why PaddleOCR?
PaddleOCR's `PP-Structure` module reconstructs table cells from scanned documents — a critical pre-processing step that dramatically improves VLM accuracy on invoices with complex line-item tables. It outperforms EasyOCR and Tesseract on Indian-language labels in headers and footers.

### Why open-source over APIs?
- **Data privacy:** GST invoices contain sensitive financial and GSTIN data. Local inference keeps all data on-premises
- **No per-call cost:** API calls at scale are expensive; open-weight models run freely after initial setup
- **Offline capable:** Works in environments without reliable internet (common in smaller Indian offices)
- **Hackathon requirement:** The problem statement explicitly mandates open-source AI

---

## 9. AI's Role in the System

AI is the **core intelligence layer** — not a wrapper, not an optional add-on. Here is the precise role of each AI component:

```
Document Image / Text
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 1: PaddleOCR (PP-OCRv4 + PP-Structure)                   │
│  ─────────────────────────────────────────────                   │
│  • Detects text regions in the image                             │
│  • Reconstructs table cells and rows                             │
│  • Produces raw text + layout bounding boxes                     │
│  • Outputs: raw_text, table_markdown, ocr_confidence per region  │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (raw text + image)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 2: Qwen2.5-VL-7B-Instruct (Vision-Language Model)        │
│  ─────────────────────────────────────────────────────────────── │
│  Input: [Document Image] + [OCR Text] + [Extraction Prompt]      │
│  Task: Semantic understanding → structured JSON extraction        │
│  Output:                                                          │
│    • supplier.name, supplier.gstin, supplier.address              │
│    • buyer.name, buyer.gstin, buyer.address                       │
│    • invoice_number, invoice_date, place_of_supply, IRN           │
│    • line_items[]: description, HSN/SAC, qty, rate, taxes         │
│    • tax_summary: CGST, SGST, IGST, cess, total_tax              │
│    • totals: subtotal, grand_total, amount_in_words               │
│    • confidence scores per field (0.0 – 1.0)                     │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (structured JSON)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 3: Deterministic GST Validation Engine (no AI)           │
│  Rules-based cross-checks on extracted values                    │
└──────────────────────────────────────────────────────────────────┘
```

**AI does:** Reading, understanding layout, resolving handwriting ambiguities, mapping fields semantically, assigning confidence  
**AI does NOT do:** GST rule checking, arithmetic verification, schema enforcement — those are deterministic

---

## 10. System Architecture

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

## 11. Component-Level Architecture

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
└── sample_invoices/
    ├── handwritten/                 ← Handwritten sample bills
    ├── scanned_pdf/                 ← Scanned PDF invoices
    ├── digital_pdf/                 ← Digitally generated PDFs
    ├── excel/                       ← Excel invoice samples
    ├── csv/                         ← CSV invoice samples
    └── printed/                     ← Printed invoice images
```

### Component Interaction Table

| Component | Input | Output | AI? |
|-----------|-------|--------|-----|
| `file_router.py` | filename, bytes | file type string | No |
| `ocr_service.py` | image bytes | raw text, table markdown, confidence | **PaddleOCR** |
| `ai_extractor.py` | image + OCR text | InvoiceRecord JSON | **Qwen2.5-VL** |
| `validator.py` | InvoiceRecord dict | same dict + validation result | No |
| `confidence.py` | InvoiceRecord dict | same dict + confidence scores | No |
| `exporter.py` | InvoiceRecord dict | JSON / CSV bytes / Excel bytes | No |

---

## 12. Data / Information Flow

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
         │                                  Output: raw_text + table_md
         │                                         │
         └─────────────┬───────────────────────────┘
                       │
                       ▼
              [4] Qwen2.5-VL-7B-Instruct
                 Input:
                   - Document image (if available)
                   - OCR text / table markdown
                   - GST extraction system prompt
                   - Target JSON schema
                 Output:
                   - All invoice fields
                   - Per-field confidence scores
                   - Structured line items array
                       │
                       ▼
              [5] Pydantic Validation
                 Strict type enforcement
                 Default values for null fields
                 InvoiceRecord object creation
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
                 Output: status (valid/warning/needs_review/invalid)
                       │
                       ▼
              [7] Confidence Scorer
                 Combines: OCR confidence + AI confidence + validation score
                 Flags: fields below 70% threshold for HITL review
                 Output: overall_confidence, per-field scores, review_required
                       │
                       ▼
              [8] Database (SQLite → PostgreSQL)
                 Saves: DocumentModel + InvoiceModel + InvoiceItemModel
                       │
                       ▼
              [9] API Response + Export
                 Returns: full InvoiceRecord JSON
                 Available: /export/json, /export/csv, /export/excel
```

---

## 13. Agentic Workflow

VYOM+ implements a **Human-in-the-Loop (HITL) agentic review workflow** for low-confidence documents.

```
                    ┌─────────────────────────────┐
                    │      Document Uploaded       │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────┐
                    │  AI Pipeline Runs (Stages 1-4)│
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │  Confidence ≥ 90%?           │
                    └──────┬───────────────┬───────┘
                   YES     │               │  NO
                           ▼               ▼
              ┌────────────────┐   ┌──────────────────────────┐
              │  Auto-Approved │   │  Flagged for Human Review │
              │  status=valid  │   │  status=needs_review      │
              └────────────────┘   │  Low-confidence fields    │
                                   │  highlighted in UI        │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Operator Views Invoice   │
                                   │  in Review UI            │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Operator Corrects Field  │
                                   │  PUT /api/documents/{id} │
                                   │  { field_path: "...",     │
                                   │    corrected_value: "..." }│
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Re-validation Triggered  │
                                   │  Confidence Recalculated  │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Record Updated in DB     │
                                   │  Correction Logged        │
                                   └──────────────────────────┘
```

**Why this is agentic:** The system does not just extract and return. It **reasons about its own uncertainty** (confidence scores), **decides whether to involve a human** (≥ 90% auto-approved, < 70% flagged), and **adapts its output** based on human correction — completing a full agent loop: *Perceive → Reason → Act → Adapt*.

---

## 14. Technology Stack

| Layer | Technology | Version | License | Role |
|-------|-----------|---------|---------|------|
| **Backend Framework** | FastAPI | 0.115+ | MIT | REST API, async request handling |
| **AI — Vision Model** | Qwen2.5-VL-7B-Instruct | latest | Apache 2.0 | Invoice understanding from images |
| **AI — Text Model** | Qwen2.5-7B-Instruct | latest | Apache 2.0 | Structured extraction from digital text |
| **Primary OCR** | PaddleOCR (PP-OCRv4) | 2.8+ | Apache 2.0 | Text + table extraction from scans |
| **Fallback OCR** | EasyOCR | 1.7+ | Apache 2.0 | CPU-only multi-language OCR |
| **Model Inference** | HuggingFace Transformers | 4.46+ | Apache 2.0 | Load and run Qwen2.5-VL locally |
| **Quantization** | BitsAndBytes / llama.cpp | latest | MIT | 4-bit inference for lower GPU memory |
| **PDF Parsing** | PyMuPDF (fitz) | 1.24+ | AGPL 3.0 | Digital PDF text + scanned PDF → images |
| **Image Processing** | OpenCV | 4.10+ | Apache 2.0 | Deskew, denoise, contrast enhance |
| **Data Schemas** | Pydantic v2 | 2.8+ | MIT | Strict data validation and models |
| **ORM** | SQLAlchemy | 2.0+ | MIT | Database abstraction |
| **Database (MVP)** | SQLite | built-in | Public domain | Invoice record storage |
| **Database (Prod)** | PostgreSQL | 16+ | PostgreSQL | Scalable production storage |
| **Spreadsheet** | pandas + openpyxl | latest | BSD | Excel/CSV parsing |
| **Export** | reportlab, fpdf2 | latest | BSD/LGPL | PDF export |
| **Demo UI** | Streamlit | 1.40+ | Apache 2.0 | Evaluator demo interface |
| **Frontend (Full)** | Next.js + React | 15+ | MIT | Production web interface |
| **Styling** | Tailwind CSS | 3+ | MIT | Frontend styling |
| **Runtime** | Python 3.11 | — | PSF | Backend runtime |

---

## 15. Expected Features

### Core Features (MVP — Final Hackathon)

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Multi-format Upload** | Accept xlsx, csv, pdf, jpg, png via REST API and Streamlit UI |
| 2 | **Smart File Routing** | Auto-detect input type and route to correct pipeline |
| 3 | **Handwritten Invoice OCR** | PaddleOCR + Qwen2.5-VL handles handwritten bills |
| 4 | **Structured Extraction** | All GST fields extracted into standard JSON schema |
| 5 | **GSTIN Validation** | 15-char format, state code, checksum validation |
| 6 | **Supply Type Check** | Intra-state (CGST+SGST) vs inter-state (IGST) consistency |
| 7 | **Line Item Arithmetic** | Qty × Rate − Discount = Taxable Value cross-check |
| 8 | **Tax Total Validation** | CGST + SGST + IGST + Cess = Total Tax verification |
| 9 | **Grand Total Reconciliation** | Taxable + Total Tax + Round-off = Grand Total |
| 10 | **Confidence Scoring** | Per-field confidence (0–100%) with overall score |
| 11 | **HITL Review Interface** | Flagged documents shown in UI with correction fields |
| 12 | **Dot-Notation Correction** | Correct any field via `supplier.gstin`, `items[0].unit_price` |
| 13 | **Export: JSON** | Machine-readable standardized JSON download |
| 14 | **Export: CSV** | Line-item CSV for spreadsheet workflows |
| 15 | **Export: Excel** | Multi-tab workbook with invoice summary + line items |
| 16 | **Dashboard Stats** | Total processed, by status, average confidence, recent invoices |
| 17 | **Document History** | All processed invoices stored and queryable |
| 18 | **Status Filtering** | Filter documents by: valid, warning, needs_review, invalid |

### Advanced Features (Stretch Goals)

| # | Feature |
|---|---------|
| 19 | Batch upload (ZIP of multiple invoices) |
| 20 | PDF export with QR code of IRN for e-invoice verification |
| 21 | Duplicate invoice detection |
| 22 | LoRA fine-tuning pipeline on custom GST invoice dataset |

---

## 16. Implementation Approach

### Phase 1 — Foundation (Day 1 of Final Hackathon)
- Set up FastAPI backend with Pydantic schemas and SQLite database
- Implement Excel and CSV parsers (`pandas` + `openpyxl`)
- Define Universal Invoice JSON schema
- Write basic GST validation rules

### Phase 2 — OCR + Digital PDF (Day 1–2)
- Integrate `PyMuPDF` for digital PDF text extraction
- Set up `PaddleOCR` for image and scanned PDF text extraction
- Implement image pre-processing pipeline (OpenCV: deskew, denoise, binarize)
- Build `file_router.py` dispatcher

### Phase 3 — Open-Source VLM Integration (Day 2)
- Load `Qwen2.5-VL-7B-Instruct` via HuggingFace Transformers
- Design GST extraction prompt with schema definition
- Implement two-stage pipeline: PaddleOCR → Qwen2.5-VL
- Test on all 6 input format types

### Phase 4 — Validation + Confidence (Day 2–3)
- Build 7-rule deterministic GST validation engine
- Implement multi-signal confidence scorer
- Wire up Human-in-the-Loop correction API
- Add re-validation on correction

### Phase 5 — Export + UI (Day 3)
- Implement JSON, CSV, Excel exporters
- Build Streamlit demo UI for evaluators
- (Stretch) Build Next.js frontend
- Test end-to-end with all sample invoice types

### Phase 6 — Polish + Evaluation (Day 3–4)
- Add sample invoices for each format type
- Measure accuracy on held-out test invoices
- Write evaluation metrics (field extraction accuracy, GSTIN validation pass rate)
- Final README, demo video, and submission

---

## 17. Expected Final Output

### For Each Processed Invoice:

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

### System-Level Outputs:
- ✅ Working REST API (FastAPI, auto-documented at `/docs`)
- ✅ Streamlit evaluator interface with upload + inspection
- ✅ Dashboard with processing statistics
- ✅ Export in JSON, CSV, Excel
- ✅ Sample invoices tested across all 6 format types
- ✅ Accuracy metrics on held-out test set

---

## 18. Future Scope / Scalability

### Scalability Path

| Milestone | Change |
|-----------|--------|
| **Scale storage** | Replace SQLite with PostgreSQL + connection pooling |
| **Scale inference** | Deploy Qwen2.5-VL via vLLM for batched GPU inference (10x throughput) |
| **Scale API** | Add Celery + Redis task queue for async processing of large batches |
| **Containerize** | Docker + docker-compose for reproducible deployment |
| **Cloud deploy** | Deploy on cloud VM with GPU (AWS/GCP/Azure) or use RunPod |

### Feature Roadmap

| Feature | Timeline |
|---------|----------|
| **Fine-tuning on Indian GST data** | Post-hackathon — LoRA fine-tune Qwen2.5-VL on labeled Indian invoice dataset to boost accuracy on regional vendor formats |
| **Batch ZIP upload** | Accept ZIP of 50+ invoices, process in parallel, return batch results |
| **E-invoice QR verification** | Decode IRN QR codes on e-invoices, cross-verify with NIC portal |
| **GSTR-2A reconciliation** | Match extracted purchase invoices against GSTR-2A data |
| **Multi-tenant SaaS** | Add organization accounts, role-based access, audit logs |
| **Mobile app** | Camera capture → instant invoice scan via mobile PWA |
| **Multilingual invoices** | Extend to regional languages (Hindi, Gujarati, Tamil) in headers/footers |
| **ERP integrations** | REST webhooks for Tally Prime, Zoho Books, QuickBooks |

---

## 19. Open-Source Dependencies / Components

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| `transformers` | 4.46+ | Apache 2.0 | Load and run Qwen2.5-VL-7B |
| `paddlepaddle` | 2.6+ | Apache 2.0 | PaddleOCR inference backend |
| `paddleocr` | 2.8+ | Apache 2.0 | OCR + PP-Structure table extraction |
| `easyocr` | 1.7+ | Apache 2.0 | Fallback OCR engine |
| `pymupdf` | 1.24+ | AGPL 3.0 | PDF text extraction and rendering |
| `opencv-python` | 4.10+ | Apache 2.0 | Image preprocessing |
| `Pillow` | 10+ | MIT-CMU | Image loading and manipulation |
| `fastapi` | 0.115+ | MIT | REST API framework |
| `uvicorn` | 0.30+ | BSD | ASGI server |
| `pydantic` | 2.8+ | MIT | Data validation and schemas |
| `pydantic-settings` | 2.5+ | MIT | Environment-based config |
| `sqlalchemy` | 2.0+ | MIT | ORM and database abstraction |
| `pandas` | 2.2+ | BSD | CSV and Excel data processing |
| `openpyxl` | 3.1+ | MIT | Excel file read/write |
| `streamlit` | 1.40+ | Apache 2.0 | Evaluator demo UI |
| `python-multipart` | 0.0.9+ | Apache 2.0 | File upload parsing |
| `python-dotenv` | 1.0+ | BSD | `.env` configuration loading |
| `reportlab` | 4.2+ | BSD | PDF export |
| `fpdf2` | 2.8+ | LGPL | Lightweight PDF generation |
| `bitsandbytes` | 0.44+ | MIT | 4-bit model quantization |
| `accelerate` | 0.34+ | Apache 2.0 | Multi-GPU inference acceleration |
| `qrcode` | 8.0+ | BSD | QR code generation (e-invoice) |
| `pytest` | 8.0+ | MIT | Unit and integration testing |

**All components are 100% open-source. No proprietary model APIs are used.**

---

## 20. Expected Challenges and Mitigation

| # | Challenge | Risk | Mitigation |
|---|-----------|------|------------|
| 1 | **Handwritten invoice quality** — low-contrast, faded ink, irregular spacing | High | Multi-step image enhancement (deskew, denoise, adaptive thresholding) before OCR; Qwen2.5-VL handles handwriting natively |
| 2 | **Qwen2.5-VL GPU memory** — 7B model needs ~14GB VRAM | High | Use 4-bit BitsAndBytes quantization (reduces to ~4-5GB); fallback to smaller Qwen2.5-3B-Instruct if needed |
| 3 | **Diverse Indian vendor formats** — hundreds of unique invoice layouts | High | Qwen2.5-VL does not need templates; prompt-based extraction is layout-agnostic |
| 4 | **OCR errors on regional fonts** — Hindi/Gujarati characters in headers | Medium | PaddleOCR supports multilingual models; English extraction still works if headers are non-English |
| 5 | **Table cell misalignment** — merged cells in complex line-item tables | Medium | PP-Structure table reconstruction handles merged cells; VLM cross-verifies with the full image |
| 6 | **JSON hallucination** — AI generating plausible but wrong field values | Medium | Strict JSON schema enforcement via Pydantic; deterministic validator catches arithmetic errors; confidence scores flag uncertain outputs |
| 7 | **Inference latency** — Qwen2.5-VL at 7B may be slow on CPU | Medium | GPU strongly preferred; for CPU-only: use EasyOCR + Qwen2.5-3B text-only model as a fast path |
| 8 | **Missing fields** — partial invoices without buyer GSTIN etc. | Low | All fields are `Optional` in schema; validator produces `warning` (not `error`) for missing non-critical fields |
| 9 | **Model hallucination on blank areas** — AI invents values for empty regions | Low | Prompt explicitly instructs: "return null if not visible"; confidence scores and validation catch invented values |
| 10 | **GSTIN OCR confusion** — common mistakes: O vs 0, I vs 1, S vs 5 | Low | PaddleOCR has high accuracy on printed characters; VLM prompt includes disambiguation rules for character confusion |

---

## 21. Deployment Strategy

VYOM+ is designed to run in **three different modes** depending on available hardware — from a free cloud notebook to a production server.

### Mode 1 — Hackathon Demo (Recommended for Final Round)

The AI model is heavy (~7B parameters). Most laptops cannot run it locally. The cleanest approach for the hackathon demo is a **split deployment**:

```
┌──────────────────────────────────────────────────────────┐
│                  HACKATHON DEMO SETUP                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   Evaluator's Browser                                    │
│          │                                               │
│          ▼                                               │
│   Streamlit / Next.js UI  ◄──── runs on your laptop     │
│          │                                               │
│          ▼                                               │
│   FastAPI Backend         ◄──── runs on your laptop     │
│     (file routing, validation, export, DB)               │
│          │                                               │
│          │  HTTP POST (image + text)                     │
│          ▼                                               │
│   AI Inference Server  ◄──── Google Colab / HuggingFace │
│   (Qwen2.5-VL-7B +         Spaces (FREE GPU T4/P100)   │
│    PaddleOCR)              exposed via ngrok tunnel      │
│          │                                               │
│          │  returns JSON                                 │
│          ▼                                               │
│   FastAPI continues with validation + response           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Step-by-step for demo day:**
1. Open Google Colab → load Qwen2.5-VL-7B on free T4 GPU
2. Run a small FastAPI inference endpoint inside Colab
3. Use `ngrok` to get a public URL for the Colab server
4. Point your local FastAPI's `AI_INFERENCE_URL` env variable to that ngrok URL
5. Run Streamlit locally — evaluator uploads invoice → routes to Colab → result returns instantly

**Cost: $0** (Colab free tier has enough GPU time for a full demo session)

---

### Mode 2 — Cloud Production Deployment

For scaling beyond the hackathon to real-world use:

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
               [10x throughput vs naive HuggingFace]
                        │
                        ▼
               PostgreSQL Database
```

**Affordable GPU cloud options:**

| Platform | GPU | Cost | Best for |
|----------|-----|------|----------|
| Google Colab Free | T4 16GB | $0 | Demo / testing |
| HuggingFace Spaces | T4 | Free tier | Public demo hosting |
| RunPod | RTX 4090 | ~$0.44/hr | Hackathon final |
| Vast.ai | RTX 3090 | ~$0.20/hr | Budget production |
| AWS EC2 `g4dn.xlarge` | T4 16GB | ~$0.53/hr | Scalable deployment |

---

### Environment Configuration (`.env`)

```env
# AI Inference endpoint — "local" or remote Colab/cloud URL
AI_INFERENCE_URL=http://localhost:8001     # local GPU
# AI_INFERENCE_URL=https://xxxx.ngrok.io  # Colab tunnel for demo

# Model settings
MODEL_NAME=Qwen/Qwen2.5-VL-7B-Instruct
USE_4BIT_QUANTIZATION=true                # reduces VRAM from 14GB → ~5GB

# App settings
DATABASE_URL=sqlite:///./vyom_invoices.db
UPLOAD_DIR=./uploads
MAX_FILE_SIZE_MB=20
CONFIDENCE_THRESHOLD_REVIEW=0.70
```

---

## Summary

VYOM+ addresses a **real, high-impact problem** in the Indian GST compliance space using a thoughtfully designed open-source AI pipeline. The system combines:

- 🔍 **PaddleOCR** for fast, accurate text and table extraction from any scan quality
- 🧠 **Qwen2.5-VL-7B** — an open-weight Vision-Language Model — for semantic understanding of invoice layout and handwritten content
- ✅ **Deterministic GST rules** for reliable statutory validation
- 🔄 **Human-in-the-loop review** for uncertainty resolution
- 📤 **Multi-format export** for downstream accounting integration

The result is a system that can take a blurry photo of a handwritten vendor bill and return a validated, machine-readable GST invoice record — **without any proprietary API calls, without any subscription cost, and without any data leaving the user's infrastructure**.

---
 
*All technologies used are open-source or open-weight. No proprietary AI APIs.*
