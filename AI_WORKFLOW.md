# AI-Assisted Engineering Workflow Documentation

This document explains the AI-assisted methodologies, tools, prompting strategies, debugging cycles, and engineering decisions made during the development of the **BiztelAI Flow** monorepo application.

---

## 🛠️ AI Developer Ecosystem Used

During the lifecycle of this project, we leveraged the following AI tools and models:

1. **Antigravity AI Agent (powered by Google DeepMind)**:
   - **Role**: Primary autonomous engineering partner.
   - **How it was used**: Executed code search, managed package builds, performed JSX tag analysis, wrote the multi-row validation engine, redesigned the Operations Dashboard using vanilla HSL CSS tokens, and implemented robust error handling.
2. **Gemini 2.5 Flash API**:
   - **Role**: Multimodal log sheet OCR and table extraction engine.
   - **How it was used**: Processes uploaded images/PDF logs to extract structured JSON data, identify handwritten fields, compute confidence scores, and provide legibility explanation reasons.

---

## 💡 Prompting and Data Ingestion Strategy

To extract operational data from unstructured handwritten log sheets, we designed a structured system prompt sent to **Gemini 2.5 Flash** with the base64 encoded document bytes.

### 1. System Prompt Constraints
We instructed the model to act as an expert log sheet digitizer, returning a structured JSON format conforming to this layout:
```json
{
  "rows": [
    {
      "date": { "value": "YYYY-MM-DD", "confidence": 0.95 },
      "shift": { "value": "I", "confidence": 0.74, "reason": "Text is faint and partially cut off." },
      "employeeNumber": { "value": "BT1234", "confidence": 0.96 },
      "operationCode": { "value": "54321", "confidence": 0.95 },
      "machineNumber": { "value": "ABC-T30", "confidence": 0.92 },
      "workOrderNumber": { "value": "165455", "confidence": 0.65, "reason": "Last digit is written in a hurried stroke." },
      "quantityProduced": { "value": "-", "confidence": 0.94 },
      "timeTaken": { "value": "2.0", "confidence": 0.93 }
    }
  ]
}
```

### 2. Prompt Safety Guards
- **Unrelated Document Rejection**: Explicit instructions were added telling the model to return an empty rows array `{"rows": []}` for invoices, receipts, cat/dog photos, or generic text documents to guarantee a clean pipeline failure.
- **Null Value Standardization**: The model was directed to map blank quantities or dashes as `"-"` rather than empty strings, ensuring custom business validations evaluate successfully.
- **JSON Output Mode**: We leveraged Gemini's `responseMimeType: "application/json"` parameter to prevent raw text block wrapper hallucinations (e.g. markdown code blocks ` ```json `), avoiding parsing failures.

---

## 🔄 Debugging and Resilience Workflows

### 1. API Quota & Rate Limit Resilience (RESOURCE_EXHAUSTED)
- **Problem**: When testing the application under Google's shared/free-tier limits, the Gemini API can throw a `RESOURCE_EXHAUSTED` (Quota exceeded) error.
- **Debugging**: We analyzed the API JSON error responses and built an automatic demo fallback directly into the server's extraction handler (`server/server.js`).
- **Solution**: If the API fails with `RESOURCE_EXHAUSTED` or rate limit errors, the system intercepts the error, falls back to simulated mock logs, and flags a dashboard warning: `"Demo Mode: Live API daily quota exceeded. Showing simulated extraction results."` This prevents the app from failing hard and preserves the demo flow.

### 2. Syntax and JSX Tag Matching Errors
- **Problem**: Misplaced tag closes caused compilation failures in the frontend bundler (e.g., `Expected "}" but found ";"` in `ReviewPanel.jsx` during Vite build).
- **Debugging**: We ran the Vite production compiler (`npm run build --prefix client`) via the CLI to catch esbuild syntax warnings, traced open JSX components, and resolved the tag structure immediately.

---

## 🚀 Impact Analysis of AI-Assisted Engineering

### Areas Where AI Accelerated Development Most:
- **HSL-Based Styling**: Generating the CSS tokens, glassmorphism dashboard grid layout, custom progress widgets, and SVG charts without external CSS framework overhead.
- **Checklist Management**: Breaking down multi-row migrations, nested field schema changes, and UI inspection requirements into incremental checklists tracked in `task.md`.
- **Validation Engine**: Translating complex rules (e.g. Roman numeral shifts, duplicate work order checks, suspiciously high quantities, and nil-production cases) into high-fidelity unit tests.

### Areas Requiring Manual Intervention:
- **Sandbox Simulation Architecture**: Implementing a system to check if an uploaded file has `unrelated` or `fail` in the filename to simulate pipeline failure states on demand without requiring active API calls.
- **Git Push Operations**: Standardizing changes and committing them safely while leaving secret environment configuration files (`.env`) excluded from source control.
- **Vite Reverse Proxy Settings**: Routing backend port mappings (`/api`, `/uploads`) to prevent cross-origin resource sharing (CORS) exceptions.
