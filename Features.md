# Atari Disk Engine: Features and Abilities

The **Atari Disk Engine** is a comprehensive, forensic-grade toolkit designed for analyzing, recovering, and manipulating Atari ST floppy disk images. It combines deep filesystem knowledge with modern UI visualizations.

## 📂 Filesystem Operations
- **FAT12 Implementation**: Full support for the standard Atari ST FAT12 filesystem.
- **File Management**:
  - **Injection**: Add files from the host system into any directory on the virtual disk.
  - **Batch Injection**: Inject all files from a host folder into the disk in one operation (`File > Inject All Files from Folder...`). Targets the currently selected directory, or the root if nothing is selected.
  - **Extraction**: Save files from the disk image back to the host system (`File > Extract` or right-click > `Save File As...`).
  - **Batch Extraction**: Extract all files from the disk to a host folder, preserving the full subdirectory structure (`File > Extract All Files to Folder...`). If multiple files are highlighted in the tree (Ctrl+click / Shift+click), only those files are extracted.
  - **Deletion**: Securely remove a single file: marks the directory entry as deleted (`0xE5`) and frees the FAT cluster chain using geometry-aware (BPB-derived) offsets — only the selected file is affected.
  - **Renaming**: Modify filenames in-place (supports 8.3 format).
  - **Properties**: View detailed metadata for any file or folder — filename, size in bytes, starting cluster, and raw attribute flags — via the right-click context menu.
- **Directory Management**:
  - **Recursive Traversal**: Browse deeply nested subdirectories.
  - **Multi-Selection**: Ctrl+click or Shift+click to select multiple files for batch operations.
  - **Creation**: Create new subdirectories anywhere on the disk.
  - **Recursive Deletion**: Delete directories and all their contents in one operation.
- **Disk Formatting**: Format disk images with a clean structure based on the current BPB geometry.
- **Disk Geometry Auto-Repair**:
  - **Fix Standard Geometry**: Automatically restores correct tracks, sectors per track, and sides based on file size.
  - **Recalculate FAT Size**: Restores the correct sectors-per-FAT value for the current image capacity.
  - **Align Root Directory Offset**: Perfectly aligns the BPB metadata to match the physical location of the root directory.

## 🔍 Forensic & Recovery Tools
- **Disk Health Audit** (`Disk > Disk Audit`):
  - Full structural analysis of the FAT12 filesystem.
  - Detects and reports: **Orphaned Clusters** (FAT-allocated but unreachable), **Cross-Linked Chains** (two files sharing a cluster), **Bad Sectors**, and **Fragmented Files**.
  - Results presented in a dedicated `DiskHealthDialog`.
- **Pattern / Signature Scanner** (`Disk > Pattern Scanner / Signatures...`):
  - Scan the entire disk image for known byte sequences or custom text/hex patterns.
  - All matches are listed with sector number and byte offset.
  - Double-click any result to jump to that exact offset in the Hex Viewer.
- **Deep Sector Analysis**:
  - **Brute Scan (Manual Override)**: Recovery mode for "Vectronix-style" or "Compact" disks that lack a standard BPB.
  - **Boot Sector Validation**: Checks the integrity of the boot sector and BIOS Parameter Block (BPB).
  - **Executable Checksum Fixer**: Calculates and applies the `0x1234` checksum to make a disk bootable by Atari TOS.
  - **BPB Editor** (`Disk > Edit Boot Sector...`): Modify geometry parameters (sectors per track, sides, cluster size, etc.) and commit them safely back to the disk image.
- **Advanced Search**:
  - Search for text strings or hexadecimal patterns across the entire disk image.
  - Jump directly from search results to the exact offset in the hex viewer.

## 📊 Visualization & UI Features
- **Tree View**:
  - Displays the full FAT12 directory structure with SVG file/folder icons.
  - **Visual Hierarchy**: Directories are shown in bold blue text; files use normal weight, making nesting immediately clear at a glance.
  - **Alternating Row Colours**: Subtle row banding and 22 px row height for easy scanning of large file lists.
  - **Header label** shows the filename of the currently open disk image (e.g., `mygame.st`), falling back to `Disk Image` when nothing is loaded.
  - Right-click context menu: **Properties**, **Save File As**, **Rename File**, **Delete File**, **New Folder**.
- **Integrated Hex Viewer**:
  - **Dual Modes**: Switch between viewing the single Boot Sector or the Entire Disk.
  - **Terminal Mode**: Green-on-black monospace viewing for text-heavy sectors.
  - **Smart Navigation**: Synchronized scrolling and jump-to-offset.
  - File cluster highlighting in full-disk view.
- **Graphical FAT Map** (`Disk > FAT Cluster Map...`):
  - Color-coded map of every cluster: Free, Used, System, EOF, Bad, Fragmented, Orphaned, and Cross-Linked.
  - Hover tooltips with per-cluster detail.
  - Live disk health statistics panel (total/used/free clusters, capacity percentage).
- **Disk Geometry Chart**: Built-in reference for all 12 supported Atari ST disk formats.
- **Drag & Drop**: Inject files by dragging them from the host OS into the tree view.

## 🎨 Personalization & Aesthetics
- **Theme Support**:
  - **Light/Dark Modes**: Manually selectable via the `View` menu.
- **Native OS Integration**: Automatically reads and applies the operating system's dark/light mode preference at startup, integrating seamlessly without manual configuration.

## ⚙️ Technical Specifications
- **Format Support**: Reads and writes both `.st` (raw sector) and `.msa` (Magic Shadow Archiver) disk images. When saving as `.msa`, tracks are individually RLE-compressed using the standard `0xE5` escape byte scheme; raw track data is stored when compression yields no benefit.
- **Save As Format Selection**: The Save dialog presents `.st` and `.msa` as distinct filter options so the output format is always explicit.
- **Geometry Handling**: Supports standard 9-sector disks and oversized (10–11 sector) formats used by demos and games. Hatari-style BPB validation cross-checks the BPB total-sector count against the actual image size before trusting SPT/sides fields.
- **FAT Correctness**: All FAT read/write/free operations are geometry-aware, using BPB-derived offsets rather than hardcoded sector numbers.
- **Architecture**: Decoupled engine logic (`AtariDiskEngine`) from the UI layer, facilitating future CLI or batch processing integrations.
