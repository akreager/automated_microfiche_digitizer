# Documentation Register

All project documentation for the MFDIG â€” Microfiche Digitizer project.

## File Format

Documents are authored in Markdown (.md) unless the document type calls for a structured format:
- Test Logs (TL) use ODS spreadsheets
- Timing diagrams use WaveDrom JSON
- State machine diagrams use Mermaid MMD

## Naming Convention

`TT-MFDIG-NN`

| Field | Description |
|---|---|
| `TT` | Two-letter document type code |
| `MFDIG` | Project identifier â€” fixed |
| `NN` | Two-digit sequence number, zero-padded |

## Document Type Codes

| Code | Type | Description |
|---|---|---|
| `DN` | Design Note | Decisions, research, architecture, reference material. Captures the *why* and *what*. |
| `CR` | Compatibility Report | Tested hardware/software combinations and outcomes on target platform. Records what works, what does not, and any workarounds. |
| `TP` | Test Plan | Test objectives, scope, procedure, and pass/fail criteria. |
| `TL` | Test Log | Companion data-collection document to a Test Plan. Always paired 1:1 with a TP of the same number. |

## Document Register

| Document | Title | Paired With | Status |
|---|---|---|---|
| CR-MFDIG-01 | Camera Stack & Capture Node Hardware Compatibility | â€” | In Progress |

---
*This README is a living document and must be updated when a document is added to the register.*
