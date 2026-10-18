# Confocal ND2 Visualizer

A **single self-contained HTML file** for organizing, renaming, and visualizing Nikon confocal `.nd2` images — entirely in your browser, with **no installation, no server, and no uploads**. Open the file in Chrome or Edge, point it at a folder of `.nd2` files, and you have a lab-ready image organizer and multi-channel viewer.

Built for fluorescent confocal microscopy (Nikon A1 / Ti2) and reads the `.nd2` format directly in pure JavaScript — no Fiji, NIS-Elements, or Python required just to look at your images.

This tool was developed collaboratively with [Claude Code](https://claude.com/claude-code), Anthropic's AI coding assistant.

> ⚠️ **Chrome or Edge only.** The tool relies on the browser File System Access API, which Firefox and Safari do not support.

---

## Why use it

- **Everything stays on your machine.** Images are read locally through the browser; nothing is ever sent to a server. Your image files are **never deleted** — the tool only renames (with confirmation).
- **One file, zero install.** Download one `.html`, double-click, done. Easy to share with labmates or drop into a shared Drive.
- **Reads `.nd2` natively.** A pure-JS decoder parses the Nikon ND2 (Ver 3.0) chunk format, including raw 16-bit pixel data, calibration, and acquisition metadata.
- **Understands your acquisition.** Auto-detects magnification, Z-stacks, focused/EDF images, denoised files, and large/stitched montages straight from the metadata.
- **Portable, side-by-side metadata.** Library structure and per-image settings are saved as small `.json` files next to your data, so they travel with the images (e.g. via Google Drive) and reload on any machine or browser.

---

## Requirements

- **Google Chrome** or **Microsoft Edge** (desktop).
- Your `.nd2` files in a local folder (a synced Drive/OneDrive folder works too).

No other dependencies.

---

## Quick start

1. **Download** `Confocal ND2 Visualizer.html` (from this repo's releases or the file itself).
2. **Open it** in Chrome or Edge (double-click, or drag it into a browser window).
3. In the **Library** tab, click **📁 Set library folder…** and choose the folder that holds the HTML file and your experiment folders.
4. Click **+ Add Experiment** to link an experiment folder (it must live *inside* the library folder).
5. Organize and rename in the **Organize & Rename** tab, then explore and export in the **Visualizer** tab.

> The browser asks for permission to access your folder **once per session** (a security requirement). Just allow it — the tool remembers the folder and reloads your library automatically.

### Recommended folder layout

```
My Library Folder/
├── Confocal ND2 Visualizer.html
├── nd2viz_library.json              (created automatically)
├── EXP 2026.06.10 (ATOH1p50)/       (an experiment folder)
│   └── D34_Brightfield(2)_ATOH1(4)/  (a channel sub-folder)
│       ├── ..._10X_Focused_01.nd2
│       └── ..._20X_Zstack_01.nd2
└── EXP 2026.07.02 (Immunostain)/
    └── Hoechst(3)_SIX1(5)_Caspase3(6)/
        └── ...nd2
```

Channel sub-folders are named `Name(band)_Name(band)_…` in band order (see the [band scheme](#channel--band-scheme) below). An optional prefix such as `D34_` or `Day5_` is allowed and ignored by the parser.

---

## Features

### 📚 Library tab — organize your experiments

- A three-level hierarchy: **Category → Experiment → channel sub-folder → `.nd2` files**.
- Default categories (**Immunostaining**, **Live-cell Imaging**, **Whole-mount staining**) that you can rename, add to, or delete.
- Link experiment folders from disk and tag sub-folders with reusable **project** labels.
- The whole structure is saved to `nd2viz_library.json` in your library folder — not in browser cache and not baked into the HTML, so sharing the HTML leaks none of your paths or data.

### ✏️ Organize & Rename tab — clean, consistent filenames

- A spreadsheet-style table per sub-folder with a **Subject** field and your own **condition columns** (add / rename / delete; drag to fill down).
- Auto-detected, read-only fields: **Magnification**, **Z-stack**, **Focused/EDF**, **Denoised**, **Large/stitched**.
- **Automatic series numbering** for replicate images.
- **Batch rename on disk** to a consistent convention (files are renamed, never deleted — you confirm first):

  ```
  Subject_Condition…_Magnification_[Large]_[Focused|Zstack]_[Denoised]_Series.nd2
  e.g.  IEO_Day5_20X_Focused_Denoised_01.nd2
  ```

- **Per-channel band override:** if auto-detection ever assigns the wrong channel (common with multipoint-split / EDF files whose metadata is stripped), set it manually from a dropdown per channel. Overrides are saved alongside your data.

### 🔬 Visualizer tab — compare and export

- A **comparison grid**: one row per file, columns for the **Overlay** plus each individual channel.
- **NIS-style LUTs** with live histogram editing — adjust color, gamma, black/white points per channel, globally or per image.
- **Z-projections:** Maximum Intensity, focused/EDF, mean, or single plane (single-plane files are never re-projected).
- **Denoising** (classical, per Z-section before projection): Gaussian, Median, Anisotropic diffusion (Perona–Malik), Total-Variation (ROF). Already-denoised files are skipped.
- **Scale bar** drawn from the file's real calibration — global, selection-aware, or per-file.
- **Filters** for magnification, image type (z-stack / projected), stitching (single / large), and each of your condition columns independently.
- Drag to **reorder channel columns**, show/hide individual channels, resize images up to 1000 px.
- **Export** the grid, a per-file strip, or a single cell — as **PNG or TIFF**, with a transparent or theme-matched background. Save folders are remembered per export type.

---

## Channel / band scheme

Channels map to a fixed "band" number by excitation/emission, following the lab convention:

| Band | Channel | Typical excitation | Display |
|------|---------|--------------------|---------|
| **2** | Brightfield / transmitted (TD) | — | Grayscale (kept out of the overlay by default) |
| **3** | DAPI / Hoechst | ~405 nm | Blue |
| **4** | GFP / FITC / EGFP | 488 nm | Green |
| **5** | RFP / DsRed / mCherry / Cy3 | 561 nm | Red |
| **6** | Cy5 / far-red | 640 nm | Far-red |

Bands are detected from the acquisition metadata (laser → emission → filter cube → plane name), with the sub-folder name used as a fallback and a manual override available when needed.

---

## How your data is stored

Nothing is uploaded anywhere. Two small JSON files are written **next to your files** on disk:

- **`nd2viz_library.json`** (in the library folder) — your categories, experiments, and projects, stored as relative paths.
- **`nd2viz_metadata.json`** (in each experiment folder) — rename metadata, condition columns, channel overrides, and all Visualizer adjustments (LUTs, colors, scale bars).

Because these live with your data, they sync through Google Drive / OneDrive and reappear on any machine or browser that opens the same folder. The HTML file itself stores no paths or data, so it is safe to share.

---

## Supported files

- Nikon **ND2 (Ver 3.0)** files, validated against **Nikon A1 confocal** and **Ti2 widefield** acquisitions, including single images, Z-stacks, focused/EDF exports, multipoint-split files, and large/stitched montages.
- Other ND2 variants may work but have not been tested.

---

## Limitations & notes

- **Chrome/Edge only** (File System Access API).
- Folder access must be **re-granted once per session** — a browser security rule no web page can bypass. The tool makes this a single click (or your first click anywhere) and then auto-loads your library.
- Denoising here is **classical** only. For AI denoising (NIS-Elements *Denoise.ai* or Fiji's *NIA / DnCNN*), run those in their own software; this tool offers anisotropic-diffusion and total-variation filtering as an in-browser substitute.
- Experiment folders must live **inside** the library folder so they can be stored as relative paths.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Chrome or Edge required" message | Open the file in Chrome or Edge, not Firefox/Safari. |
| Library is empty after reopening | Click **🔓 Load library** (or just click anywhere) and allow folder access — browsers reset permission each session. |
| An experiment shows "⚠ not found" | The folder moved or was renamed. Use **🔗 Link folder** to re-point it. |
| A channel is the wrong color/identity | Use the per-channel **band override** dropdown in the Organize & Rename tab. |

---

## Privacy & data safety

- 🔒 **No uploads, no server, no telemetry.** All processing happens locally in your browser.
- 🗂️ **Your images are never deleted.** The tool only renames files, and only after you confirm.
- 📤 **Safe to share.** The HTML contains no experiment paths or data.

---

## License

Released under the [MIT License](LICENSE) — free to use, modify, and share.

---

## Acknowledgements

Developed for confocal microscopy image management in the lab, in collaboration with [Claude Code](https://claude.com/claude-code) (Anthropic). The ND2 reader was reverse-engineered and validated against real Nikon A1 / Ti2 acquisitions.

<!-- Optional: add screenshots here to show the Library, Organize & Rename, and Visualizer tabs. -->
<!-- ![Visualizer](docs/visualizer.png) -->
