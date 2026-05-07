<div align="center">

# ⬡ BoundaryQC

**AI-powered land boundary verification, drafting, and DXF generation for Windows.**

[![Version](https://img.shields.io/badge/version-1.5.49-blue?style=flat-square)](https://github.com/band72/BoundaryQC/releases/latest)
[![Platform](https://img.shields.io/badge/platform-Windows-0078D4?style=flat-square&logo=windows)](https://github.com/band72/BoundaryQC/releases/latest)
[![Framework](https://img.shields.io/badge/.NET-8.0%20WPF-512BD4?style=flat-square&logo=dotnet)](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
[![License](https://img.shields.io/badge/license-Proprietary-red?style=flat-square)](LICENSE)

[**Download Latest Installer**](https://github.com/band72/BoundaryQC/releases/latest) · [Report a Bug](https://github.com/band72/BoundaryQC/issues) · [Changelog](CHANGELOG.md)

</div>

---

## Overview

**BoundaryQC** is a professional-grade WPF desktop application built for land surveyors, civil engineers, and GIS professionals. It converts legal deed descriptions and survey plat images into mathematically verified, CAD-ready DXF geometry — automatically.

Powered by the **Google Gemini AI** engine, BoundaryQC handles the full pipeline from raw text or scanned plat image to a finished, closed boundary traverse with sub-foot accuracy.

---

## ✨ Features at a Glance

| Feature | Description |
|---|---|
| **AI Deed Drafting** | Paste any legal description — the AI extracts bearings, distances, curves, and meanders and converts them to a closed DXF traverse |
| **Survey Plat Vision** | Upload a scanned plat image; the vision model extracts the call table and drafts the geometry automatically |
| **Batch Processing** | Process entire folders of mixed file types (`.jpg`, `.png`, `.pdf`, `.docx`, `.txt`) with parallel AI agents |
| **Multi-Stage Repair Engine** | Dual-run consensus voting → AI misclosure repair → Adjacent Lot Isolation, applied automatically in sequence until closure is achieved |
| **DXF Export & Viewer** | Full CAD-ready DXF output with a built-in viewer — inspect geometry before opening in AutoCAD or Civil 3D |
| **2-Page ANSI-B PDF** | Every deed produces a professional deliverable: Page 1 is a to-scale boundary sketch; Page 2 contains the legal description, line data table, COGO coordinates, and traverse closure report |
| **Legal Description Writer** | Interactive editor to manually compose or correct survey calls with real-time closure feedback |
| **AI Agent Chat** | Conversational AI interface — drop an image or paste text and get a full DXF + deliverable set |
| **Auto-Update** | Background version checks with an in-app notification banner when a new release is available |

---

## 🤖 AI Processing Pipelines

BoundaryQC offers four entry points, each tuned for a different workflow.

### 1 — Deed DXF  *(text input)*

Best for: pasting a plain-text legal description directly.

```
Paste legal description
        ↓
ExtractDeedCallsAsync   ← text-to-JSON AI prompt
        ↓
DeedDrafter.GenerateDeedDxf
        ↓
error > 0.05' ? → SolveMisclosureAsync → regenerate
```

**Output:** DXF · OCR.txt · Cogo.txt · Cogo-Script.txt · QC_Report.txt · 2-page PDF

---

### 2 — Plat DXF  *(scanned plat image)*

Best for: uploading a scanned or photographed survey plat image.

```
Upload plat image (+ optional table image)
        ↓
ExtractPlatCallsFromImageAsync   ← vision model, V3 pattern recognition
        ↓
DeedDrafter.GenerateDeedDxf
        ↓
error > 1.0' ? → Run extraction a 2nd time, keep lower-error result (consensus)
        ↓
error > 1.0' ? → SolvePlatMisclosureAsync (heavy repair)
        ↓
error > 0.30' ? → AdjacentLotIsolator.IsolateAsync (focused segment isolation)
```

**Output:** DXF · OCR.txt · Cogo.txt · Cogo-Script.txt · QC_Report.txt · 2-page PDF

---

### 3 — Batch  *(folder of mixed files)*

Best for: processing dozens or hundreds of deeds unattended.

```
Classify each file (PLAT / TEXT / SECTION_BREAKDOWN / LOT_BLOCK)
        ↓
PLAT  → ExtractPlatCallsFromImageAsync (vision model, V3 pattern recognition)
TEXT  → ExtractTextFromFile + ExtractDeedCallsAsync
        ↓
error > 1.0' ? → Run extraction a 2nd time, keep lower-error result (consensus)
        ↓
DeedDrafter.GenerateDeedDxf
        ↓
error > 1.0' ? → SolvePlatMisclosureAsync (heavy repair)
        ↓
error > 0.30' ? → AdjacentLotIsolator.IsolateAsync (focused segment isolation)
```

**Output per item:** DXF · OCR.txt · Cogo.txt · Cogo-Script.txt · QC_Report.txt · 2-page PDF  
**Batch summary:** `ProcessingSummary.txt` with closure stats for every item

---

### 4 — AI Agent Chat  *(interactive)*

Best for: one-off images or conversational deed processing with live feedback.

```
Image attachment → ExtractPlatCallsFromImageAsync
     or
Typed text      → ExtractDeedCallsAsync
        ↓
DeedDrafter.GenerateDeedDxf
        ↓
SolveMisclosureAsync (if needed)
```

**Output:** Full file set dropped beside the original image.

---

### Pipeline Capability Matrix

| Capability | Deed DXF | Plat DXF | Batch | AI Agent | Writer |
|---|:---:|:---:|:---:|:---:|:---:|
| Document auto-classification | — | — | ✅ | — | — |
| Vision model (scanned plats) | — | ✅ | ✅ | ✅ | — |
| V3 pattern recognition | — | ✅ | ✅ | partial | — |
| Dual-run consensus voting | — | ✅ | ✅ | — | — |
| AI misclosure repair | ✅ light | ✅ heavy | ✅ heavy | ✅ light | — |
| Adjacent lot isolation engine | — | ✅ | ✅ | — | — |
| 2-page ANSI-B survey PDF | ✅ | ✅ | ✅ | ✅ | — |
| **Success Rate (%)** | **75%** | **50%** | **90%** | **70%** | **62%** |
| **Production Ready** | No | No | **Yes** | No | No |

> All four paths share the same final step — `DeedDrafter.GenerateDeedDxf()` — which writes the complete output.  
> **Production Ready** = ≥ 90% completion of required functional goals, with no critical failures and acceptable risk on remaining items.

---

## 🔧 Automated Repair System

When a traverse does not close mathematically, BoundaryQC applies repairs in sequence until the error falls within tolerance:

### Stage 1 — Bowtie Auto-Fix
Self-intersecting geometry ("bowtie" polygons) is detected using vector cross-products. BoundaryQC iteratively identifies the offending bearing quadrant, swaps it, and re-tests until the polygon ring seals cleanly.

### Stage 2 — Dual-Run Consensus Voting
If closure error exceeds **1.0 ft**, the AI extraction is run a second time independently. Whichever of the two runs produces the lower linear error is retained for final output.

### Stage 3 — AI Misclosure Repair
A dedicated repair prompt (`SolvePlatMisclosureAsync`) is given the original image, the extracted JSON, and the full closure diagnostic. The AI identifies and corrects the specific erroneous bearing or distance.

### Stage 4 — Adjacent Lot Isolation Engine
If error remains above **0.30 ft** after AI repair, BoundaryQC extracts the boundary calls of neighboring lots from the same plat. Any shared segment verified by a neighbor that closes perfectly is confirmed correct. The remaining unverified segments are flagged as suspects and targeted with a focused AI repair pass. A multi-lot combined DXF is written with each lot on its own named layer.

---

## 🗂️ Document Classification

BoundaryQC automatically routes each document to the correct extraction pipeline:

| Classification | Description | Pipeline |
|---|---|---|
| `METES_BOUNDS` | Standard deed with bearing/distance calls | `ExtractDeedCallsAsync` → DXF traverse |
| `SECTION_BREAKDOWN` | Pure PLSS aliquot fractions (no bearing calls) | Compute ideal rectangle from aliquot fractions |
| `QUALIFIED_PLSS` | PLSS parent trimmed by road R/W or offset | Compute gross parent → apply clip lines |
| `LOT_BLOCK` | Subdivision lot-and-block reference | `LotBlockDrafter` |
| `PLAT` | Scanned survey plat image | Vision model → `ExtractPlatCallsFromImageAsync` |

---

## 🖥️ Headless / CLI Mode

BoundaryQC supports a full headless CLI for server or script-driven automation:

```bash
# Single image
BoundaryQC.exe --image plat.png --table table.png

# Batch folder
BoundaryQC.exe --batch "C:\Deeds\Regency-Title" --output "C:\Output"

# Adjacent lot isolation
BoundaryQC.exe --image plat.png --adjacent-lots "Lot 712,Lot 714,Lot 716"

# Validate table-crop thresholds
BoundaryQC.exe --validate-correct-crops "C:\crops-correct"
```

Parallel batch mode forks up to **15 concurrent AI agents** — configure under **Settings → Output → Use Multiple Agents**.

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| UI | C# .NET 8.0 WPF |
| AI Engine | Google Gemini (via `RCS.AntiGravity.Gemini`) |
| COGO Math | Custom `CogoMath` engine — DMS bearing arithmetic, arc solver, shoelace area |
| DXF Generation | `DeedDrafter`, `DxfRedrawer` (netDxf-compatible output) |
| Data Persistence | LiteDB (project store) + SQLite/EF Core (relational) |
| Image Processing | Moore-Neighbor contour tracing, Zhang-Suen thinning |
| PDF Output | Custom 2-page ANSI-B renderer (`ParcelSketchRenderer`) |

---

## 🚀 Installation

### Installer (Recommended)

1. Download **[BoundaryQC_1.5.49.exe](https://github.com/band72/BoundaryQC/releases/latest)** from the Releases page.
2. Run the installer. .NET 8 Desktop Runtime will be detected and prompted if not present.
3. Launch **BoundaryQC** from the Start Menu or Desktop shortcut.
4. Set your **Gemini API key** under Settings → API Key.

### Build from Source

```bash
git clone https://github.com/band72/BoundaryQC.git
cd BoundaryQC
dotnet restore
dotnet publish BoundaryQC.csproj -c Release -r win-x64 --self-contained true -o bin\publish
```

Requires: Visual Studio 2022 (or .NET 8 SDK) · Windows 10/11 x64

---

## 📋 Output Files

Every successful run produces the following in the output folder:

| File | Contents |
|---|---|
| `*.dxf` | CAD-ready boundary geometry (AutoCAD / Civil 3D compatible) |
| `Cogo.txt` | Numbered coordinate list (Point 500, P1, P2 … format) |
| `Cogo-Script.txt` | Executable COGO script for import into survey software |
| `ocr.txt` | Raw OCR / extracted legal description text |
| `QC_Report.txt` | Traverse closure report with linear error and precision ratio |
| `Parcel_Sketch.png` | To-scale boundary sketch image |
| `Parcel_Sketch_v2.pdf` | 2-page ANSI-B PDF deliverable |

---

## 📐 Accuracy & Tolerances

- Bearing precision: **0.1-second** (DMS)
- Distance precision: **0.01 ft**
- Closure tolerance target: **≤ 0.05 ft** (PERFECT), **≤ 0.30 ft** (GOOD)
- Area reported to: **0.01 acres / 2 decimal places**
- Coordinate list origin: Point **500** (POB), then P1, P2 …

---

## 📄 License

Copyright © 2026 Band72. All rights reserved.

This software is proprietary. Permission is granted for personal and internal evaluation use only. Commercial use, redistribution, or incorporation into other products requires prior written permission. See [LICENSE](LICENSE) for full terms.

---

<div align="center">
<sub>Built with ⬡ for the land surveying community · v1.5.49</sub>
</div>
