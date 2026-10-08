# VYOM+ — End-to-End AI-Powered GST Invoice Intelligence System

> **HacktoberFest Hack Day · Open Source AI Hackathon · Problem Statement 3**  

### 👥 Team - Merge Conflict

| # | Team Members |
|---|-------------|
| 1 | Krishna Mall |
| 2 | Aaroos Patel |
| 3 | Aryan Malhotra |

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

**VYOM+** is an open-source document intelligence platform built for the Indian GST ecosystem. It reads invoices in whatever form they arrive (handwritten, scanned, printed or digital) and turns them into clean, validated records that accounting software can use right away.

---

## 2. Problem Statement

Indian businesses handle **millions of GST invoices every month**, and no two look alike. Some come as Excel sheets from suppliers, some as scanned PDFs from old printers, some as handwritten paper bills from small vendors, and plenty as photos taken on a phone.

Most of this still gets typed into accounting systems by hand, and that creates real problems:

- ⏱ **Slow.** A single batch can take hours of data entry.
- ❌ **Error-prone.** Manual transcription leads to arithmetic slips and GSTIN mistakes.
- 💸 **Costly.** Skilled accountants spend their time on repetitive extraction instead of analysis.
- 📋 **A compliance risk.** GSTIN format errors, CGST/IGST mismatches and wrong totals can get a GST return rejected.

There is no affordable, open-source tool that covers the **full range** of invoice formats, especially **handwritten** ones, while also performing **statutory GST validation** and returning **structured JSON** that accounting software can consume.

**That is the gap VYOM+ is built to fill.**

---

## 3. Project Overview

VYOM+ is a **multi-modal document intelligence pipeline**. It:

1. Accepts invoices in **6 formats**: `.xlsx`, `.csv`, `.pdf` (digital), `.pdf` (scanned), `.jpg`/`.jpeg` and `.png`
2. Sends each document down the **right processing path** based on its detected type
3. Uses **open-source vision-language models** and **OCR engines** to extract the invoice fields
4. Applies a **deterministic GST rules engine** to validate what was extracted
5. Scores **field-level confidence** and flags uncertain extractions for human review
6. Exports results as **JSON, CSV or Excel**
7. Provides a **web UI** where evaluators and operators can upload, inspect, correct and export invoices

The whole design rests on one principle:

> **"AI extracts. Deterministic code validates. Confidence decides. Humans resolve uncertainty."**

Here is the upload screen. Drop a file in and the side panel walks through what happens next: format detection, extraction, GST and totals verification, then confidence scoring and exceptions.

<img width="1190" height="513" alt="Screenshot 2026-10-08 015259" src="https://github.com/user-attachments/assets/eb3e326a-f8f5-4508-963b-9a8e14a020f0" />

---

## 4. Proposed Solution

VYOM+ uses a **three-layer architecture**.

### Layer 1 — Document Ingestion & Parsing
A file router detects the input type and passes it to the right parser:
- **Structured files** (Excel, CSV) go through `pandas` + `openpyxl`
- **Digital PDFs** go through `PyMuPDF` / `pdfplumber` for text and table extraction
- **Scanned PDFs and images** are rendered to images and sent through a multi-stage OCR pipeline

### Layer 2 — AI-Powered Extraction Engine (Open-Source Core)
Two open-source AI stages work together:

1. **PaddleOCR** pulls raw text out of scanned and image documents and rebuilds their tables. It performs well on Indian scripts and low-quality scans.
2. **Qwen2.5-VL (7B)**, an open-weight Vision-Language Model, receives the document image along with the OCR text and returns structured JSON containing every invoice field, the line items and confidence scores.

Together they outperform pure OCR or a pure VLM on the messy, real-world invoices Indian businesses actually deal with.

### Layer 3 — Validation, Confidence & Export
- A **deterministic GST rules engine** cross-checks GSTIN format, supply-type consistency, line-item arithmetic and grand-total reconciliation.
- A **multi-signal confidence scorer** combines OCR confidence, AI confidence and the validation pass rate.
- A **human-in-the-loop API** lets operators correct flagged fields and re-run validation.
- An **export layer** produces JSON, CSV or Excel output.

---

## 5. Objectives

| # | Objective |
|---|-----------|
| 1 | Accept and correctly identify all 6 supported invoice input formats |
| 2 | Extract the key GST invoice fields: invoice number, date, supplier GSTIN, buyer GSTIN, line items, HSN/SAC codes, CGST, SGST, IGST and totals |
| 3 | Handle handwritten invoices with an open-source OCR + VLM pipeline |
| 4 | Validate extracted data against 7+ deterministic GST statutory rules |
| 5 | Produce field-level confidence scores and flag low-confidence extractions |
| 6 | Provide an interface where humans can review and correct flagged fields |
| 7 | Export standardized JSON / CSV / Excel for downstream accounting |
| 8 | Use **only open-source / open-weight AI models**, with no proprietary API calls |
| 9 | Provide a working evaluator interface for uploading and inspecting documents |

---

## 6. Target Users / Use Case

| User | Use Case |
|------|----------|
| **SME Accountants** | Cut down the manual GST data entry that comes with vendor bills |
| **CA Firms** | Process client invoice bundles in batches for GST return filing |
| **ERP / Accounting Software** | Plug in as a document-ingestion microservice |
| **GST Consultants** | Validate large sets of invoices for compliance audits |
| **Hackathon Evaluators** | Upload sample invoices and inspect the extracted structured data |
| **VYOM+ Platform** | Serve as the AI document intelligence backend for accounting workflows |

### Real-World Scenario

> A small textile trader in Surat uploads a blurry photo of a handwritten purchase bill from a local supplier. VYOM+ reads the image, extracts the supplier GSTIN, line items and GST components, works out that the sale is intra-state, confirms that CGST + SGST were applied correctly, scores its own confidence, and returns a validated JSON record, all in under 10 seconds.

---

## 7. Open-Source AI Technology Selected

### Primary: **Qwen2.5-VL-7B-Instruct**
- **Model family:** Qwen2.5-VL (Alibaba Cloud, open-weight, Apache 2.0 license)
- **Model size:** 7B parameters. It fits on a single 16GB GPU, or on 8GB with quantization.
- **HuggingFace:** [`Qwen/Qwen2.5-VL-7B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct)
- **Input:** Image(s) plus a text prompt, producing structured JSON output
- **Key capability:** Layout-aware visual reasoning over documents, tables and handwritten text

### Secondary OCR: **PaddleOCR (PP-OCRv4)**
- **Library:** `paddlepaddle` + `paddleocr`
- **License:** Apache 2.0
- **Key capability:** State-of-the-art OCR for printed and semi-handwritten documents, with table structure recognition through `PP-Structure`

### Fallback OCR: **EasyOCR**
- **License:** Apache 2.0
- **Key capability:** Lightweight multi-language OCR that runs on CPU alone, so no GPU is needed

### LLM for Text Structuring: **Qwen2.5-7B-Instruct** (text-only fallback)
- Used when the document is already digital text (CSV, Excel, digital PDF) and vision isn't required
- Structures the raw text into the invoice JSON schema
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

**Qwen2.5-VL-7B comes out ahead** because it is the only option that can handle our three hardest input categories (handwritten bills, scans from low-end printers, and wildly varied vendor templates) without a labeled dataset to fine-tune on.

### Why PaddleOCR?
PaddleOCR's `PP-Structure` module rebuilds table cells from scanned documents. That is a key pre-processing step, and it noticeably improves the VLM's accuracy on invoices with complex line-item tables. It also beats EasyOCR and Tesseract on Indian-language labels in headers and footers.

### Why open-source over APIs?
- **Data privacy:** GST invoices contain sensitive financial and GSTIN data. Local inference keeps all of it on-premises.
- **No per-call cost:** API calls get expensive at scale, while open-weight models run freely after the initial setup.
- **Offline capable:** The system works where internet access is unreliable, which is common in smaller Indian offices.
- **Hackathon requirement:** The problem statement explicitly mandates open-source AI.

---

## 9. AI's Role in the System

AI is the **core intelligence layer** of VYOM+, not a wrapper or an optional add-on. Here is what each AI component does:

```
Document Image / Text
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 1: PaddleOCR (PP-OCRv4 + PP-Structure)                    │
│  ─────────────────────────────────────────────                   │
│  • Detects text regions in the image                             │
│  • Reconstructs table cells and rows                             │
│  • Produces raw text + layout bounding boxes                     │
│  • Outputs: raw_text, table_markdown, ocr_confidence per region  │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (raw text + image)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 2: Qwen2.5-VL-7B-Instruct (Vision-Language Model)         │
│  ─────────────────────────────────────────────────────────────── │
│  Input: [Document Image] + [OCR Text] + [Extraction Prompt]      │
│  Task: Semantic understanding → structured JSON extraction       │
│  Output:                                                         │
│    • supplier.name, supplier.gstin, supplier.address             │
│    • buyer.name, buyer.gstin, buyer.address                      │
│    • invoice_number, invoice_date, place_of_supply, IRN          │
│    • line_items[]: description, HSN/SAC, qty, rate, taxes        │
│    • tax_summary: CGST, SGST, IGST, cess, total_tax              │
│    • totals: subtotal, grand_total, amount_in_words              │
│    • confidence scores per field (0.0 – 1.0)                     │
└──────────────────────────┬───────────────────────────────────────┘
                           │  (structured JSON)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 3: Deterministic GST Validation Engine (no AI)            │ 
│  Rules-based cross-checks on extracted values                    │
└──────────────────────────────────────────────────────────────────┘
```

**AI does:** reading, understanding layout, resolving handwriting ambiguities, mapping fields semantically and assigning confidence.  
**AI does NOT do:** GST rule checking, arithmetic verification or schema enforcement. Those stay deterministic so the results are always predictable.

---

## 10. System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          VYOM+ System Architecture                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐        ┌──────────────────────────────────────────────┐   │
│   │   User /    │        │              FastAPI Backend                 │   │
│   │  Evaluator  │◄──────►│                                              │   │
│   │  (Browser)  │  REST  │  ┌──────────┐  ┌───────────┐  ┌──────────┐   │   │
│   └─────────────┘        │  │  Upload  │  │  Invoice  │  │  Export  │   │   │
│                          │  │  Router  │  │   CRUD    │  │  Router  │   │   │
│   ┌─────────────┐        │  └────┬─────┘  └─────┬─────┘  └────┬─────┘   │   │
│   │  Streamlit  │        │       │              │             │         │   │
│   │  Demo UI   │◄───────►│  ┌────▼──────────────▼──────────────▼─────┐  │   │
│   └─────────────┘        │  │           Processing Pipeline          │  │   │
│                          │  │                                        │  │   │
│                          │  │  File Router → [Excel/CSV/PDF/Image]   │  │   │
│                          │  │       │                                │  │   │
│                          │  │  ┌────▼─────────────────────────────┐  │  │   │
│                          │  │  │     AI Extraction Engine         │  │  │   │
│                          │  │  │   PaddleOCR → Qwen2.5-VL-7B      │  │  │   │
│                          │  │  └────────────────┬─────────────────┘  │  │   │
│                          │  │                   │                    │  │   │
│                          │  │  ┌────────────────▼─────────────────┐  │  │   │
│                          │  │  │    GST Validation Engine         │  │  │   │
│                          │  │  │  (Deterministic Rules)           │  │  │   │
│                          │  │  └────────────────┬─────────────────┘  │  │   │
│                          │  │                   │                    │  │   │
│                          │  │  ┌────────────────▼─────────────────┐  │  │   │
│                          │  │  │  Confidence Scorer + HITL Flag   │  │  │   │
│                          │  │  └────────────────┬─────────────────┘  │  │   │
│                          │  └────────────────────┼───────────────────┘  │   │
│                          │                       ▼                      │   │
│                          │         ┌─────────────────────────┐          │   │
│                          │         │    SQLite / PostgreSQL  │          │   │
│                          │         │    (Invoice Records)    │          │   │
│                          │         └─────────────────────────┘          │   │
│                          └──────────────────────────────────────────────┘   │
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
├── docs/
│   └── screenshots/                 ← UI screenshots used in this README
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

VYOM+ uses a **Human-in-the-Loop (HITL) agentic review workflow** for documents it isn't confident about.

```
                    ┌─────────────────────────────┐
                    │      Document Uploaded      │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────┐
                    │AI Pipeline Runs (Stages 1-4)│
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │      Confidence ≥ 90%?      │
                    └──────┬───────────────┬──────┘
                   YES     │               │  NO
                           ▼               ▼
              ┌────────────────┐   ┌──────────────────────────┐
              │  Auto-Approved │   │  Flagged for Human Review│
              │  status=valid  │   │  status=needs_review     │
              └────────────────┘   │  Low-confidence fields   │
                                   │  highlighted in UI       │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Operator Views Invoice  │
                                   │  in Review UI            │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Operator Corrects Field │
                                   │  PUT /api/documents/{id} │
                                   │  { field_path: "...",    │
                                   │  corrected_value: "..." }│
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Re-validation Triggered │
                                   │  Confidence Recalculated │
                                   └──────────┬───────────────┘
                                              │
                                   ┌──────────▼───────────────┐
                                   │  Record Updated in DB    │
                                   │  Correction Logged       │
                                   └──────────────────────────┘
```

**Why this counts as agentic:** the system doesn't just extract and hand back a result. It **reasons about its own uncertainty** through confidence scores, **decides whether a person needs to step in** (90% and above is auto-approved, below 70% is flagged), and **adapts its output** once a human corrects it. That completes a full agent loop: *Perceive → Reason → Act → Adapt*.

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
| 1 | **Multi-format Upload** | Accepts xlsx, csv, pdf, jpg and png through the REST API and the Streamlit UI |
| 2 | **Smart File Routing** | Detects the input type automatically and picks the right pipeline |
| 3 | **Handwritten Invoice OCR** | PaddleOCR + Qwen2.5-VL read handwritten bills |
| 4 | **Structured Extraction** | Every GST field is extracted into one standard JSON schema |
| 5 | **GSTIN Validation** | Checks the 15-character format, state code and checksum |
| 6 | **Supply Type Check** | Confirms intra-state (CGST+SGST) vs inter-state (IGST) consistency |
| 7 | **Line Item Arithmetic** | Cross-checks Qty × Rate − Discount = Taxable Value |
| 8 | **Tax Total Validation** | Verifies CGST + SGST + IGST + Cess = Total Tax |
| 9 | **Grand Total Reconciliation** | Verifies Taxable + Total Tax + Round-off = Grand Total |
| 10 | **Confidence Scoring** | Per-field confidence (0–100%) along with an overall score |
| 11 | **HITL Review Interface** | Flagged documents appear in the UI with fields ready to correct |
| 12 | **Dot-Notation Correction** | Fix any field by path, such as `supplier.gstin` or `items[0].unit_price` |
| 13 | **Export: JSON** | Machine-readable, standardized JSON download |
| 14 | **Export: CSV** | Line-item CSV for spreadsheet workflows |
| 15 | **Export: Excel** | Multi-tab workbook with an invoice summary and line items |
| 16 | **Dashboard Stats** | Total processed, breakdown by status, average confidence and recent invoices |
| 17 | **Document History** | Every processed invoice is stored and queryable |
| 18 | **Status Filtering** | Filter documents by valid, warning, needs_review or invalid |

The dashboard brings several of these together. The workspace overview shows documents processed, how many validated, how many need review and the average confidence. A "Needs your attention" panel surfaces anything waiting on verification, and recent documents sit beside it with their status.

<img width="1622" height="677" alt="Screenshot 2026-10-08 015227" src="https://github.com/user-attachments/assets/250ec4bc-77c9-47cf-a855-27b8a4100963" />

### Advanced Features (Stretch Goals)

| # | Feature |
|---|---------|
| 19 | Batch upload (ZIP of multiple invoices) |
| 20 | PDF export with a QR code of the IRN for e-invoice verification |
| 21 | Duplicate invoice detection |
| 22 | LoRA fine-tuning pipeline on a custom GST invoice dataset |

---

## 16. Implementation Approach

### Phase 1 — Foundation (Day 1 of Final Hackathon)
- Set up the FastAPI backend with Pydantic schemas and a SQLite database
- Implement the Excel and CSV parsers (`pandas` + `openpyxl`)
- Define the Universal Invoice JSON schema
- Write the basic GST validation rules

### Phase 2 — OCR + Digital PDF (Day 1–2)
- Integrate `PyMuPDF` for digital PDF text extraction
- Set up `PaddleOCR` for image and scanned-PDF text extraction
- Build the image pre-processing pipeline with OpenCV (deskew, denoise, binarize)
- Build the `file_router.py` dispatcher

### Phase 3 — Open-Source VLM Integration (Day 2)
- Load `Qwen2.5-VL-7B-Instruct` through HuggingFace Transformers
- Design the GST extraction prompt with the schema definition
- Implement the two-stage pipeline: PaddleOCR → Qwen2.5-VL
- Test across all 6 input format types

### Phase 4 — Validation + Confidence (Day 2–3)
- Build the 7-rule deterministic GST validation engine
- Implement the multi-signal confidence scorer
- Wire up the Human-in-the-Loop correction API
- Add re-validation whenever a correction is made

### Phase 5 — Export + UI (Day 3)
- Implement the JSON, CSV and Excel exporters
- Build the Streamlit demo UI for evaluators
- (Stretch) Build the Next.js frontend
- Test end-to-end with every sample invoice type

### Phase 6 — Polish + Evaluation (Day 3–4)
- Add sample invoices for each format type
- Measure accuracy on held-out test invoices
- Write evaluation metrics (field extraction accuracy, GSTIN validation pass rate)
- Finish the README, demo video and submission

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
- ✅ A working REST API (FastAPI, auto-documented at `/docs`)
- ✅ A Streamlit evaluator interface for upload and inspection
- ✅ A dashboard with processing statistics
- ✅ Export in JSON, CSV and Excel
- ✅ Sample invoices tested across all 6 format types
- ✅ Accuracy metrics on a held-out test set

---

## 18. Future Scope / Scalability

### Scalability Path

| Milestone | Change |
|-----------|--------|
| **Scale storage** | Replace SQLite with PostgreSQL and connection pooling |
| **Scale inference** | Deploy Qwen2.5-VL on vLLM for batched GPU inference (roughly 10x throughput) |
| **Scale API** | Add a Celery + Redis task queue so large batches process asynchronously |
| **Containerize** | Use Docker + docker-compose for reproducible deployment |
| **Cloud deploy** | Run on a GPU cloud VM (AWS/GCP/Azure) or on RunPod |

### Feature Roadmap

| Feature | Timeline |
|---------|----------|
| **Fine-tuning on Indian GST data** | Post-hackathon: LoRA fine-tune Qwen2.5-VL on a labeled Indian invoice dataset to improve accuracy on regional vendor formats |
| **Batch ZIP upload** | Accept a ZIP of 50+ invoices, process them in parallel and return batch results |
| **E-invoice QR verification** | Decode IRN QR codes on e-invoices and cross-verify them with the NIC portal |
| **GSTR-2A reconciliation** | Match extracted purchase invoices against GSTR-2A data |
| **Multi-tenant SaaS** | Add organization accounts, role-based access and audit logs |
| **Mobile app** | Camera capture for instant invoice scanning through a mobile PWA |
| **Multilingual invoices** | Extend support to regional languages (Hindi, Gujarati, Tamil) in headers and footers |
| **ERP integrations** | REST webhooks for Tally Prime, Zoho Books and QuickBooks |

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

**Every component is open-source. No proprietary model APIs are used.**

---

## 20. Expected Challenges and Mitigation

| # | Challenge | Risk | Mitigation |
|---|-----------|------|------------|
| 1 | **Handwritten invoice quality** — low contrast, faded ink, irregular spacing | High | Multi-step image enhancement (deskew, denoise, adaptive thresholding) before OCR; Qwen2.5-VL reads handwriting natively |
| 2 | **Qwen2.5-VL GPU memory** — the 7B model needs about 14GB VRAM | High | Use 4-bit BitsAndBytes quantization (down to roughly 4–5GB), and fall back to the smaller Qwen2.5-3B-Instruct if needed |
| 3 | **Diverse Indian vendor formats** — hundreds of unique invoice layouts | High | Qwen2.5-VL needs no templates, and prompt-based extraction doesn't depend on layout |
| 4 | **OCR errors on regional fonts** — Hindi/Gujarati characters in headers | Medium | PaddleOCR has multilingual models, and English fields still extract correctly when headers are in another script |
| 5 | **Table cell misalignment** — merged cells in complex line-item tables | Medium | PP-Structure handles merged cells, and the VLM cross-checks against the full image |
| 6 | **JSON hallucination** — the AI producing plausible but wrong field values | Medium | Strict Pydantic schema enforcement, deterministic arithmetic checks and confidence scores that flag uncertain output |
| 7 | **Inference latency** — Qwen2.5-VL at 7B can be slow on CPU | Medium | A GPU is strongly preferred; for CPU-only setups, use EasyOCR with the text-only Qwen2.5-3B as a fast path |
| 8 | **Missing fields** — partial invoices without a buyer GSTIN, for example | Low | All fields are `Optional`, and the validator raises a `warning` (not an error) for missing non-critical fields |
| 9 | **Model hallucination on blank areas** — the AI inventing values for empty regions | Low | The prompt says to return null if a value isn't visible, and confidence scores and validation catch invented values |
| 10 | **GSTIN OCR confusion** — common mix-ups like O vs 0, I vs 1, S vs 5 | Low | PaddleOCR is accurate on printed characters, and the VLM prompt includes disambiguation rules for these confusions |

---

## 21. Deployment Strategy

VYOM+ is designed to run in **three different modes** depending on the hardware available, from a free cloud notebook up to a production server.

### Mode 1 — Hackathon Demo (Recommended for Final Round)

The AI model is heavy (about 7B parameters), and most laptops can't run it locally. The cleanest approach for the hackathon demo is a **split deployment**: the app runs on your laptop while the AI runs on a free cloud GPU.

```
┌──────────────────────────────────────────────────────────┐
│                  HACKATHON DEMO SETUP                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   Evaluator's Browser                                    │
│          │                                               │
│          ▼                                               │
│   Streamlit / Next.js UI  ◄──── runs on your laptop      │
│          │                                               │
│          ▼                                               │
│   FastAPI Backend         ◄──── runs on your laptop      │
│     (file routing, validation, export, DB)               │
│          │                                               │
│          │  HTTP POST (image + text)                     │
│          ▼                                               │
│   AI Inference Server  ◄──── Google Colab / HuggingFace  │
│   (Qwen2.5-VL-7B +         Spaces (FREE GPU T4/P100)     │
│    PaddleOCR)              exposed via ngrok tunnel      │
│          │                                               │
│          │  returns JSON                                 │
│          ▼                                               │
│   FastAPI continues with validation + response           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Step-by-step for demo day:**
1. Open Google Colab and load Qwen2.5-VL-7B on the free T4 GPU
2. Run a small FastAPI inference endpoint inside Colab
3. Use `ngrok` to get a public URL for the Colab server
4. Point your local FastAPI's `AI_INFERENCE_URL` environment variable at that ngrok URL
5. Run Streamlit locally. The evaluator uploads an invoice, it is routed to Colab, and the result comes straight back

**Cost: $0.** The Colab free tier gives you enough GPU time for a full demo session.

---

### Mode 2 — Cloud Production Deployment

For scaling beyond the hackathon into real-world use:

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

VYOM+ takes on a real, high-impact problem in Indian GST compliance with a carefully designed open-source AI pipeline. It brings together:

- 🔍 **PaddleOCR** for fast, accurate text and table extraction at any scan quality
- 🧠 **Qwen2.5-VL-7B**, an open-weight Vision-Language Model, to understand invoice layout and handwritten content
- ✅ **Deterministic GST rules** for reliable statutory validation
- 🔄 **Human-in-the-loop review** to resolve whatever the system is unsure about
- 📤 **Multi-format export** for downstream accounting integration

The end result is a system that can take a blurry photo of a handwritten vendor bill and return a validated, machine-readable GST invoice record, **with no proprietary API calls, no subscription cost, and no data leaving the user's infrastructure**.

---

*All technologies used are open-source or open-weight. No proprietary AI APIs.*
