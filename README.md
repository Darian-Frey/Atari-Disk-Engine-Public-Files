# <div align="center">🕹️ Atari Disk Engine (Linux Edition)</div>

<div align="center">
  <img src="resources/Atari_Disk_Engine.ico" alt="Atari Disk Engine Logo" width="128" height="128">
  <p><b>"Power Without The Price" — Modern Disk Manipulation for Retro Hardware.</b></p>
</div>

---

## 💾 Overview

The **Atari Disk Engine** is a specialized RAM-disk forensic utility designed to bridge the gap between your modern Linux environment and the classic 16-bit Atari ST ecosystem. Built with a focus on speed and accuracy, it allows you to manipulate `.ST` and `.MSA` floppy images with surgical precision.

Whether you are rebuilding a game disk, extracting old code, or managing GEM-based software, this toolkit provides the low-level access required to handle **FAT12** filesystems and **TOS-specific** boot sectors directly in memory.

---

## 🚀 Key Features

### Core Capabilities
* **Dynamic Disk Architect**: Create blank disks across **12 unique storage capacities**, from classic 360KB single-sided floppies up to massive 16MB Gotek images. Automatically scales FAT12 geometries safely.
* **RAM-Disk Engine**: Instant disk creation and manipulation in system memory, avoiding slow disk I/O until you save.
* **MSA Read & Write**: Loads `.MSA` images (decompressed transparently on open) and saves back to `.MSA` using per-track RLE compression compatible with the Magic Shadow Archiver standard.
* **Format-Aware Save Dialog**: When saving, choose explicitly between **Atari ST Raw (*.st)** and **MSA Disk Image (*.msa)** — the correct extension is applied automatically.
* **Brute-Force Directory Scanning**: Advanced logic to recover file structures from non-standard "compact" disks (Vectronix style) and D-Bug high-bit encoded disks.
* **Diagnostic Hex Viewer**: Real-time visualization of raw disk data with sector-aligned mapping. Includes a **Retro Terminal Theme** (green-on-black).
* **Disk Metadata Profiling**: Deep-scan diagnostics for cluster health and space utilization, with a built-in **Disk Size Chart** reference table.

### File & Directory Management
* **Drag-and-Drop Injection**: Drag files directly from your Linux file manager into the Tree View to instantly inject them.
* **Single File Injection** (`File > Inject File to Disk`): Add a file into a specific subdirectory (or the root) with automatic FAT chain allocation.
* **Batch Injection** (`File > Inject All Files from Folder...`): Pick a host folder and inject every file inside it in one operation. Reports injected/failed counts on completion.
* **Single File Extraction** (`File > Extract Selected File` / `Ctrl+E`): Reconstruct a file from its FAT cluster chain and save it to the host.
* **Batch Extraction** (`File > Extract All Files to Folder...`): Extract the entire disk — preserving subdirectory structure — to a host folder. If multiple files are **Ctrl+click / Shift+click** selected in the Tree View, only those files are extracted instead.
* **Multi-Selection**: Ctrl+click or Shift+click to highlight multiple files for batch operations.
* **Recursive Folder Deletion**: Delete entire directory trees in a single click. Safely walks the FAT chain to free all associated clusters.
* **New Folder Creation**: Create new subdirectories anywhere on the disk, even on freshly formatted blank images.
* **File Renaming**: Rename directory entries in-place (8.3 format enforced).
* **Save File As**: Right-click any file in the tree to extract it directly to your host filesystem.
* **File Properties**: View detailed metadata (size, cluster, attributes) for any file or folder via right-click.

### Tree View
* **Visual Hierarchy**: Directories are displayed in **bold blue** text; files use normal weight — making nested structure immediately obvious at a glance.
* **Alternating Row Shading**: Subtle row banding and comfortable 22 px row height for easy scanning of large file lists.
* **SVG Icons**: Crisp file and folder icons that scale cleanly at any DPI.
* **Disk Name Header**: The Tree View column header always shows the filename of the currently open disk image.

### Forensic & Analysis Tools
* **Disk Health Audit** (`Disk > Disk Audit`): Full structural analysis detecting orphaned clusters, cross-linked chains, bad sectors, and fragmentation, displayed in a dedicated dialog.
* **Pattern / Signature Scanner** (`Disk > Pattern Scanner / Signatures...`): Scan the entire disk for known byte signatures or custom hex/text patterns. Double-click any match to jump to it in the Hex Viewer.
* **Boot Sector Editor** (`Disk > Edit Boot Sector...`): Interactively modify BPB geometry parameters and commit them safely back to the image.
* **FAT Cluster Map** (`Disk > FAT Cluster Map...`): Color-coded graphical map of every cluster (Free, Used, EOF, Bad, Orphaned, Cross-Linked, Fragmented) with a live disk health statistics panel.
* **Executable Checksum Fixer** (`Disk > Make Disk Bootable`): Calculates and writes the `0x1234` TOS boot checksum to make any disk bootable on real hardware.

### Personalization
* **Theme Engine**: Light and Dark themes available under `View > Theme`.
* **Native OS Theming**: Automatically reads and applies your operating system's dark/light mode preference at startup.

---

## 🛠️ Hardware & Environment

This project is optimized and tested for the following environment:

* **Machine**: Dell Latitude-5480 / ThinkPad P15 Gen 2i
* **OS**: Linux (Ubuntu 22.04+ recommended)
* **Compiler**: GCC (C++17)
* **Toolkit**: Qt 5 Core/Widgets

---

## 📂 Quick Start

### 1. Build from Source

```bash
make clean
make -j$(nproc)
```

### 2. Launch the Engine

```bash
./bin/AtariDiskEngine
```

### 3. Standard Workflow

1. **Open or Create**: `File > Open Disk` to load an existing `.ST` or `.MSA` image, or `File > New Disk` to create a fresh formatted image.
2. **Organize**: Right-click empty space or existing folders to create a `New Folder`.
3. **Inject**: Drag a file and drop it over your target folder, or use `File > Inject All Files from Folder...` for batch injection.
4. **Inspect**: Click any file in the Tree View to view its raw bytes in the Hex Viewer.
5. **Audit**: `Disk > Disk Audit` to check for orphaned clusters, bad sectors, or FAT corruption.
6. **Extract**: Ctrl+click the files you want, then `File > Extract All Files to Folder...` to save just those files — or save the whole disk in one go.
7. **Save**: `File > Save Disk As...` and choose `.st` (raw, maximum compatibility) or `.msa` (compressed, smaller file size) for use in emulators (Hatari) or on real hardware.

---

## 📝 Atari Technical Specs (Standard 720KB)

| Attribute | Value |
| :--- | :--- |
| **Sectors per Track** | 9 |
| **Heads** | 2 |
| **Sectors per Cluster** | 2 (1024 bytes) |
| **Directory Start** | Sector 11 |
| **Data Start** | Sector 18 |
| **FS Type** | FAT12 (Atari TOS Variant) |

---

## 🎨 Aesthetic Credits

* **Boot Sector Signature**: "ANTIGRAV"
* **UI Inspiration**: Classic GEM Desktop / Neodesk
* **Philosophy**: Keep it fast, keep it accurate.

---
<div align="center">
  <p><b>Atari Disk Engine</b></p>
  <p><i>Copyright © 2026 Darian Frey</i></p>
  <p>Author: Shane Hartley | <a href="mailto:shane.hartley06@gmail.com">shane.hartley06@gmail.com</a></p>
  <p><a href="https://github.com/Darian-Frey">GitHub Profile</a></p>
  <br>
  <p><i>Developed for the Atari Community in 2026. Long live the ST.</i></p>
</div>
