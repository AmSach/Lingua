# 
#  ██╗     ██╗███╗   ██╗ ██████╗ ██╗   ██╗ █████╗ 
#  ██║     ██║████╗  ██║██╔════╝ ██║   ██║██╔══██╗
#  ██║     ██║██╔██╗ ██║██║  ███╗██║   ██║███████║
#  ██║     ██║██║╚██╗██║██║   ██║██║   ██║██╔══██╗
#  ███████╗██║██║ ╚████║╚██████╔╝╚██████╔╝██║  ██║
#  ╚══════╝╚═╝╚═╝  ╚═══╝ ╚═════╝  ╚═════╝ ╚═╝  ╚═╝ v23
#
#  ── THE ZERO-BACKEND, OFFLINE-FIRST DOCUMENT TRANSMUTER ──
#

> **"It's just a single, self-contained index.html file. No servers, no user accounts, and absolutely no telemetry. Yet, it packs a 10-provider redundant fallback cascade, a multi-threaded WebAssembly OCR pool, and a custom high-DPI canvas-based shaper to render Indic/Arabic text perfectly. Run it locally, host it on Vercel, or throw it on a floppy disk—it just works."**

---

## 🛠️ THE QUICK START

1. Double-click `index.html`.
2. There is no step 2. You are now running a heavy-duty document translation suite.

---

## 📋 TABLE OF CONTENTS

1. [What is this?](#-what-is-this)
2. [The 10-Engine Fallback Cascade](#-the-10-engine-fallback-cascade)
3. [Behind the Scenes: Indic & Arabic Layout Shaping](#-behind-the-scenes-indic--arabic-layout-shaping)
4. [Local Memory Cache (IndexedDB)](#-local-memory-cache-indexeddb)
5. [Document Reconstruction Engine (PDF, DOCX, EPUB)](#-document-reconstruction-engine-pdf-docx-epub)
6. [Design System & Aesthetics](#-design-system--aesthetics)
7. [Running Locally or Hosting on Vercel](#-running-locally-or-hosting-on-vercel)

---

## 🔮 WHAT IS THIS?

**Lingua v23** is a lightweight, zero-dependency translation tool delivered as a single HTML file. It runs entirely inside your browser, making direct client-side API calls to a cascade of free public translation endpoints.

It handles two primary workflows:
*   **Plain text**: Paste up to 5,000 characters and watch a swarm of public APIs return the translation.
*   **Documents**: Drag & drop any mix of PDFs, DOCXs, ODTs, EPUBs, or scanned images. Lingua OCRs them (if scanned), splits them into paragraphs, preserves bold/italic/strikethrough styles, translates them, and reflows them onto a new PDF.

---

## 🚀 THE 10-ENGINE FALLBACK CASCADE

Lingua uses a redundant pipeline of 10 free translation providers. If one goes down (or blocks us), the orchestrator automatically routes the query to the next alive provider.

```
┌─────────────────────────────────────────────────────┐
│                  Browser (index.html)                │
│                                                      │
│  ┌──────────┐  ┌────────────┐  ┌─────────────────┐  │
│  │  Text UI  │  │ File Queue │  │ Provider Pills  │  │
│  └────┬──────┘  └─────┬──────┘  └────────┬────────┘  │
│       │               │                  │            │
│       └───────┬────────┘                 │            │
│               ▼                          │            │
│     ┌──────────────────┐                 │            │
│     │  IndexedDB Cache │◄────────────────┘            │
│     │   (Local Memory) │                              │
│     └────────┬─────────┘                              │
│              │ cache hit / miss                       │
│              ▼                                        │
│     ┌─────────────────────────────────┐               │
│     │   translateAll() orchestrator   │               │
│     │                                 │               │
│     │  Phase 1: Google Translate Race │               │
│     │  (Primary + Alt in parallel)    │               │
│     │                                 │               │
│     │  Phase 2: Fallback Sweep        │               │
│     │  (LT x3 + MyMemory + Lingva x3) │               │
│     └───────────────┬─────────────────┘               │
│                     │                                  │
│        ┌────────────┼────────────┐                    │
│        ▼            ▼            ▼                    │
│   GT Endpoints   LT Instances  Lingva Instances       │
│   (dict-chrome,   (terraprint,  (garuda, ducks,       │
│    googleapis)    astian, eral) plausibility)         │
└─────────────────────────────────────────────────────┘
```

### The Engine Redundancy Tier:
1.  **Google Translate (Primary)**: Batches up to 40 paragraphs per request using the `gtx` endpoint.
2.  **Google Translate (Alt)**: Individual fallback request via the `dict-chrome-ex` endpoint.
3.  **MyMemory**: Reliable public backup (limited to 1,000 words/day without registration).
4.  **LibreTranslate (3 Public Nodes)**: Round-robin load-balanced across Terraprint, Astian, and Eral.
5.  **Lingva Translate (3 Public Nodes)**: Round-robin fallback across Garuda, Ducks, and Plausibility.
6.  **Apertium**: Rule-based translation engine for specialized language pairs.

---

## 🤯 BEHIND THE SCENES: INDIC & ARABIC LAYOUT SHAPING

Standard PDF libraries (like `pdf-lib`) are basic and do not support **complex text shaping** (OpenType GSUB/GPOS tables) or **RTL text flow**. 

If you try to write Hindi (`प्रतिकृति`) or Arabic (`مرحبا`) directly, pdf-lib will draw them as a series of detached, broken, or reversed characters. 

### How Lingua Solves This (The Canvas Hybrid Engine):
1.  **Dynamic Browser Registration**: When you translate into an extended language (Hindi, Arabic, Chinese, etc.), Lingua fetches the font TTF bytes from jsDelivr and registers it globally in the browser using the `FontFace` API.
2.  **Native Browser Shaping**: We spin up a high-DPI (4x scale, 300 DPI equivalent) offscreen canvas, set `ctx.font` to the newly loaded font, and let the browser's native shaping engine handle the complex ligatures, cursive joins, vowel matra reordering, and RTL formatting.
3.  **Transparent Image Injection**: We export the shaped text canvas as a transparent PNG, embed it into the PDF via `outDoc.embedPng`, and draw it exactly over the original bounding box.
4.  **The Result**: Crisp, vector-aligned, perfectly spelled Hindi and Arabic, while Western languages remain selectable native text.

---

## 🧠 LOCAL MEMORY CACHE (INDEXEDDB)

Every single translation is stored in the browser's local **IndexedDB** cache. 
-   **Performance**: If you re-translate the same document or sentence, it takes **0ms**.
-   **Privacy**: Your cache resides 100% on your local disk. No centralized server knows what you are translating.

---

## ✂️ DOCUMENT RECONSTRUCTION ENGINE

### 📄 The PDF Reflow Pipeline
1.  **OCR Detection**: Checks if the page contains extractable text. If not, it uses a multi-threaded `Tesseract.js` worker pool to scan and extract bounding boxes.
2.  **Heuristics Line-Grouping**: Rebuilds logical paragraphs. Unlike simple line-by-line tools, Lingua knows not to merge address blocks, table cells, or signature titles by analyzing trailing commas, sentence endings, column alignments, and word counts.
3.  **Context-Aware Translation**: Translates paragraphs sequentially, passing the previous paragraph's translation as context to preserve pronoun genders and tenses.
4.  **Reflow & Drawing**: Covers the original text using white background rectangles and draws the new text with automatically scaled font sizes to prevent box overflow.

---

## 🎨 DESIGN SYSTEM & AESTHETICS

Lingua's UI is inspired by **Teenage Engineering** hardware manuals and classic control panels:
-   **Chassis**: Matte Obsidian Carbon (`#12100f`)
-   **Accents**: Copper / Bronze (`#c59c7d`)
-   **Indicators**: Mechanical safety orange (`#ff9933`) and live-pulsing green dots.
-   **Loading Viz**: A custom canvas-rendered neural network animation pulsing to the processing frequency of active translation jobs.

---

## ☁️ RUNNING LOCALLY OR HOSTING ON VERCEL

Since there is zero server code, you can run this locally or host it on Vercel/Netlify for free.

### Local Development:
```bash
# Start a simple HTTP server in the repository folder
npx -y http-server -p 8181
```

### Vercel Deployment:
Just link the repository to Vercel and hit deploy. It will serve `index.html` instantly as a high-performance edge-cached static app. OCR model files (`.traineddata.gz`) and fonts will be fetched on demand by the client browser.
