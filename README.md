# CertiStock — Production Document AI & Inventory Reconciliation Engine

[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![OCR Engine](https://img.shields.io/badge/OCR-PaddleOCR%20%7C%20PyPDF-FF6F00?style=flat)](https://github.com/PaddlePaddle/PaddleOCR)
[![Desktop](https://img.shields.io/badge/Desktop-Electron%20%7C%20TypeScript-47848F?style=flat&logo=electron&logoColor=white)](https://www.electronjs.org/)
[![Distribution](https://img.shields.io/badge/Platform-Microsoft_Store-0078D7?style=flat&logo=windows&logoColor=white)](https://apps.microsoft.com/)
[![License: Proprietary / Academic](https://img.shields.io/badge/License-Proprietary%20%2F%20Academic%20Review-blue)](LICENSE)

> **Commercial Desktop Software published on the Microsoft Store** by **AinzaTechLabs**.  
> Engineered by **Mohammad Anas Africawala** to automate transaction certificate extraction, mass-balance calculations, and inventory reconciliation for industrial textile supply chains.

---

## 1. Executive Summary & Problem Formulation

Global textile manufacturing supply chains are subject to stringent regulatory verification standards—including the **Global Organic Textile Standard (GOTS)** and **Global Recycled Standard (GRS)**. Verifying raw material origin requires manually auditing and reconciling multi-page Transaction Certificates (TCs), supplier shipping notes, and inventory dumps.

Manual verification suffers from:
1. **Severe Latency:** Auditing 500+ weekly non-standard vendor PDFs takes 25+ human engineering hours.
2. **Data Transcription Errors:** Typographical discrepancies in fiber composition percentages break mass-balance equations.
3. **Enterprise Privacy Constraints:** Proprietary batch pricing and vendor supply routes prevent uploading unencrypted trade documents to public multi-tenant cloud APIs.

**CertiStock** resolves this with a **zero-cloud-egress local desktop architecture**, combining a high-throughput **PaddleOCR & PyPDF field-extraction worker**, a **Node.js/TypeScript orchestration server**, and an **Electron native desktop UI** distributed via the **Microsoft Store**.

---

## 2. System Architecture & Data Flow

```mermaid
flowchart TD
    subgraph Client ["Desktop Client Tier (Electron / React)"]
        UI["Electron Desktop Window<br>(Vite + React + TailwindCSS)"]
        Preload["Electron Secure Bridge<br>(IPC preload.cjs)"]
    end

    subgraph Orchestrator ["Local Orchestrator Tier (Node.js / Express)"]
        Server["Express Orchestration Server<br>(Port 8787 / Localhost)"]
        DB["Local Relational Store<br>(SQLite / Transaction Cache)"]
        ExcelEngine["Excel Reconciliation Engine<br>(Spreadsheet Parser & Validator)"]
    end

    subgraph AIWorker ["Document AI Tier (FastAPI / Python 3.11)"]
        Worker["FastAPI OCR Worker (Port 8000)"]
        PyPDF["PyPDF Text & Structural Extractor"]
        Paddle["PaddleOCR Deep Neural Vision Fallback"]
        Reconciler["Mass-Balance & Field Extractor<br>(Regex + Geometric Rules)"]
    end

    UI <-->|IPC Events| Preload
    Preload <-->|HTTP REST / JSON| Server
    Server <-->|CRUD Queries| DB
    Server <-->|Batch Uploads| ExcelEngine
    Server <-->|Async OCR Request (Base64)| Worker
    Worker --> PyPDF
    PyPDF -.->|Scan / Degraded Quality| Paddle
    Worker --> Reconciler
```

---

## 3. Core Technical Capabilities

### A. Dual-Engine Document AI Extraction (`ocr-worker/app.py`)
* **Primary Digital Parser:** Utilizes `PyPDF` structural token streams to extract tabular lot numbers, net weights, certified fiber percentages, and seller/buyer IDs in **<150ms per document**.
* **Optical Vision Fallback:** If a vendor submits a scanned or rasterized certificate, the pipeline automatically routes the page to **PaddleOCR** with directional text angle classification.
* **Accuracy:** Exceeds **92% field-extraction accuracy** across non-standard layout variations across 15+ major global certification issuing bodies.

### B. Automated Mass-Balance Verification
* Formulates mass-balance conservation constraints:
  $$\sum 	ext{Input Certified Fiber Weight} \ge \sum 	ext{Output Product Weight} + 	ext{Documented Wastage}$$
* Automatically flags volume discrepancies, expired validity windows, and mismatched standard certification codes before stock issuance.

### C. Enterprise Desktop Distribution
* Built using `electron-builder` with hardened Windows AppX/MSIX packaging.
* Distributed securely through the **Microsoft Store** for seamless automated enterprise deployment across factory workstations without requiring complex Python environment setup by end operators.

---

## 4. Repository Structure

```text
certistock/
├── electron/               # Native Electron desktop wrapper
│   ├── main.cjs            # Application lifecycle & background IPC
│   ├── preload.cjs         # Context-isolated secure bridge
│   └── build.cjs           # Windows Store packaging script
├── server/                 # Local Node.js orchestration backend
│   ├── src/                # Express controllers & validation logic
│   ├── sql/                # Schema definitions for local transaction cache
│   └── tsconfig.json       # Strict TypeScript configuration
├── ocr-worker/             # High-throughput Python Document AI worker
│   ├── app.py              # FastAPI microservice (Extraction & Mass-Balance)
│   ├── requirements.txt    # Fast core dependencies (PyPDF, openpyxl, Pydantic)
│   └── requirements-paddleocr.txt # Neural OCR fallback packages
├── src/                    # Frontend React/Tailwind user interface
├── public/                 # Static visual assets and icons
└── package.json            # Root multi-package workspace configuration
```

---

## 5. Microservice API Interface (`ocr-worker`)

| Method | Endpoint | Description | Payload / Response |
| :--- | :--- | :--- | :--- |
| `GET` | `/health` | Worker heartbeat & version diagnostic | `{"status": "ok", "version": "1.0.3"}` |
| `POST` | `/extract` | Parses base64 PDF and extracts certified metadata | Input: `OcrRequest` (base64 string)<br>Output: Structured Certificate JSON |
| `POST` | `/mass-balance` | Executes conservation verification rules | Input: `MassBalanceRequest`<br>Output: Reconciliation Status & Variance |

---

## 6. Local Setup & Reproduction

### Prerequisites
* **Node.js** >= 18.x
* **Python** >= 3.10
* **PowerShell** (Windows Administrator for desktop integration)

### Running the Services

```bash
# 1. Start the Document AI Worker
cd ocr-worker
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
uvicorn app:app --port 8000 --reload

# 2. Start the Local Orchestration Server
cd ../server
npm install
npm run dev

# 3. Launch the Desktop Application
cd ..
npm install
npm run electron:dev
```

---

## 7. Engineering Attribution & Industry Role

* **Lead Software & ML Architect:** Mohammad Anas Africawala
* **Organization:** AinzaTechLabs
* **Status:** Commercial Production Software (Microsoft Store Published)
