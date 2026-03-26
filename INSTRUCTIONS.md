# Atari Disk Engine — Complete User Manual

---

## Table of Contents

1. [Getting Started](#1-getting-started)
2. [File Menu — Disk Operations](#2-file-menu--disk-operations)
3. [File Tree Panel](#3-file-tree-panel)
4. [Hex Viewer](#4-hex-viewer)
5. [Disk Menu — Analysis Tools](#5-disk-menu--analysis-tools)
6. [View Menu — Display Options](#6-view-menu--display-options)
7. [Repair Menu — Disk Recovery](#7-repair-menu--disk-recovery)
8. [Help Menu](#8-help-menu)
9. [Toolbar](#9-toolbar)
10. [Status Bar](#10-status-bar)
11. [Keyboard Shortcuts](#11-keyboard-shortcuts)
12. [Warnings & Dangers](#12-warnings--dangers)
13. [Best Practices](#13-best-practices)

---

## 1. Getting Started

When you launch Atari Disk Engine, you will see a welcome screen in the centre panel displaying the application name, version, and author information. The file tree panel on the left will show "No disk loaded".

To begin working, open a disk image using **File > Open Disk** or press **Ctrl+O**. The engine supports two formats:

- **.ST** — Raw Atari ST sector dump. Every byte maps directly to a sector on the floppy. Compatible with all emulators and real hardware.
- **.MSA** — Magic Shadow Archiver compressed format. Each track is RLE-compressed. The engine decompresses transparently on load.

Disk images from 360 KB single-sided floppies up to 16 MB Gotek images are supported. The engine automatically detects geometry using the BPB (BIOS Parameter Block) in the boot sector. If the BPB is missing or corrupt, Hatari-compatible size-based geometry detection is used as a fallback.

---

## 2. File Menu — Disk Operations

### New Disk (`File > New Disk`)

Creates a blank, formatted disk image in memory. Choose from 12 geometries grouped into three categories:

**Standard & Common**
- 360 KB (Single-Sided) — 80 tracks, 9 SPT, 1 side
- 720 KB (Double-Sided) — 80 tracks, 9 SPT, 2 sides

**Non-Standard / Oversized**
- 800 KB — 80 tracks, 10 SPT, 2 sides
- 820 KB — 82 tracks, 10 SPT, 2 sides
- 1.0 MB — 82 tracks, 12 SPT, 2 sides

**High-Density & Gotek-Only**
- 1.44 MB — 80 tracks, 18 SPT, 2 sides
- 2.0 MB — 80 tracks, 25 SPT, 2 sides
- 4.0 MB through 16.0 MB (Gotek virtual floppies)

The engine automatically scales FAT12 Sectors Per Cluster to keep even 16 MB images compliant with FAT12 hardware limitations.

**What it does:** Initialises a clean FAT12 filesystem with boot sector, two FAT copies, and an empty root directory. The disk exists only in memory until you save it.

**Danger:** If you have an unsaved disk open, the engine will prompt you to save before creating the new disk. If you click "Discard", all unsaved changes to the previous disk are lost permanently.

---

### Open Disk (`File > Open Disk` / `Ctrl+O`)

Opens a file dialog filtered to `.ST` and `.MSA` files. The selected image is loaded into memory, the file tree is populated, and the status bar shows full geometry information.

**What it does:** Reads the entire disk image into RAM. MSA files are decompressed. The BPB is parsed and validated. If the BPB is damaged, the engine falls back to Hatari-style size-based geometry detection.

**Danger:** If you have an unsaved disk open, you will be prompted to save first.

---

### Save Disk (`File > Save Disk` / `Ctrl+S`)

Writes the in-memory disk image back to its original file path without any dialog.

**What it does:** Overwrites the original file on disk with the current in-memory contents. If the disk was loaded as MSA, it is saved as MSA with per-track RLE compression.

**Danger:** This overwrites the original file immediately and irreversibly. There is no undo. If you want to keep the original, use Save Disk As first to create a copy.

---

### Save Disk As (`File > Save Disk As...` / `Ctrl+Shift+S`)

Opens a save dialog with two format options:

- **Atari ST Raw Image (*.st)** — writes raw sector data byte-for-byte
- **MSA Disk Image (*.msa)** — encodes with per-track RLE compression

The correct extension is applied automatically based on which filter you select.

**What it does:** Creates a new file at the chosen location. The in-memory disk is not modified. After saving, the "current file path" updates to the new location, so subsequent Save operations target the new file.

**Danger:** If you save to an existing filename, that file is overwritten without additional warning beyond the OS file dialog confirmation.

---

### Close Disk (`File > Close Disk`)

Unloads the current disk from memory. The file tree clears, the hex view returns to the welcome screen, and the status bar resets.

**What it does:** Discards the in-memory disk buffer. The original file on your filesystem is not modified.

**Danger:** If you have unsaved changes, you will be prompted to save. Clicking "Discard" loses all changes permanently.

---

### Inject File to Disk (`File > Inject File to Disk...`)

*Pro feature.*

Opens a file dialog to select a single file from your filesystem. The file is injected into the currently selected directory on the disk (or the root directory if nothing is selected).

**What it does:** Allocates free clusters on the disk, writes the file data into those clusters, updates the FAT chain, and creates a new directory entry with the correct filename, size, and attributes.

**How to use:**
1. Select a folder in the file tree (optional — defaults to root)
2. Choose File > Inject File to Disk
3. Select the file to inject
4. The file tree refreshes to show the new file

**Danger:** Filenames must conform to 8.3 DOS format (8 character name, 3 character extension, no spaces or special characters). If the disk is full, the injection will fail. The change exists only in memory until you save.

---

### Inject All Files from Folder (`File > Inject All Files from Folder...`)

*Pro feature.*

Opens a folder picker. Every file inside the selected folder is injected into the currently selected directory on the disk.

**What it does:** Iterates through all files in the chosen folder and injects each one sequentially. A summary dialog reports how many files succeeded and how many failed.

**Danger:** Does not recurse into subfolders. If the disk runs out of space partway through, some files will be injected and others will not. Check the summary dialog carefully.

---

### Extract Selected File (`File > Extract Selected File` / `Ctrl+E`)

Saves the currently selected file from the disk image to your filesystem.

**What it does:** Follows the file's FAT cluster chain, reconstructs the complete binary data, and writes it to the chosen location.

**Danger:** None — this is a read-only operation on the disk image.

---

### Extract All Files to Folder (`File > Extract All Files to Folder...`)

Opens a folder picker, then extracts files:

- If **two or more files are selected** in the tree (via Ctrl+click or Shift+click), only those specific files are extracted
- If **one or no files are selected**, all files on the entire disk are extracted, preserving the subdirectory structure

**What it does:** Walks the FAT cluster chain for each file and writes the reconstructed data to the target folder. Subdirectories are recreated on the host filesystem.

**Danger:** None — read-only operation. Existing files in the target folder with the same name will be overwritten without warning.

---

### Exit (`File > Exit` / `Ctrl+Q`)

Closes the application. If there are unsaved changes, you will be prompted to save.

---

## 3. File Tree Panel

The left panel displays the FAT12 directory structure of the loaded disk.

### Layout

- **Header:** Shows the disk image filename. Hover over it for a tooltip with the full filename (useful for long names that get truncated).
- **Columns:** File/folder name (left), file size (right-aligned)
- **Directories:** Shown in **bold blue** text with a folder icon and file count in parentheses, e.g. `AUTO (3)`
- **Files:** Normal weight text with a file icon and size in KB or bytes
- **Alternating row shading:** Subtle alternating row colours for readability
- **Indentation:** 32px per level to make the hierarchy visually clear

### Dot Entries

By default, the `.` (current directory) and `..` (parent directory) entries inside subdirectories are hidden. Toggle them on with **View > Show . and .. Entries**.

### Selection

- **Click** a file to select it and view its contents in the hex viewer
- **Ctrl+click** to add individual files to the selection (for batch extraction)
- **Shift+click** to select a range of files
- **Click a directory** to expand/collapse it

### Drag and Drop

Drag a file from your Linux file manager and drop it onto the tree view to inject it. Dropping on a folder targets that folder; dropping on empty space targets the root directory. *Pro feature.*

### Right-Click Context Menu

- **Properties** — Opens a dialog showing filename, raw size, starting cluster, byte offset, and all attribute flags (Read-Only, Hidden, System, Directory, Archive)
- **Save File As...** — Extract the selected file to your filesystem
- **Rename File** — Rename the directory entry in-place (8.3 format). *Pro feature.*
- **Delete File** — Marks the entry as deleted and frees its FAT clusters. *Pro feature.*
- **Forensic Wipe Slack Space** — Zeroes out any unused bytes in the file's last cluster (the gap between the file's actual size and the end of its last allocated cluster). *Pro feature.*
- **New Folder** — Creates a new subdirectory with `.` and `..` entries. *Pro feature.*

---

## 4. Hex Viewer

The central panel provides direct byte-level visibility into raw sector data.

### Viewing Modes

**Boot Sector View (default):** When a disk is first loaded and no file is selected, the hex view shows a formatted boot sector information panel displaying:
- Disk name and OEM label
- Full geometry (tracks, sectors per track, sides, sectors per cluster)
- FAT layout (FAT count, FAT size, reserved sectors)
- Root directory location and entry count
- File and directory statistics
- Free space remaining
- Geometry detection mode (BPB Standard / Hatari Size-Based / Manual Override)
- Boot checksum and bootability status

**File View:** Click any file in the tree to display its raw cluster data. The status bar shows the file name and size.

**Full Disk View** (`View > Toggle Full Disk Hex View`): Displays the entire disk image in the hex viewer. Every byte is accessible by scrolling. Colour-coded highlighting shows disk layout regions:
- **Green** — Boot sector (sector 0)
- **Red/Pink** — FAT 1
- **Dark Red** — FAT 2
- **Blue** — Root directory
- **Yellow** — Data area (used clusters)
- **Gray** — Free space

**BPB Field Highlighting:** In boot sector view, BPB fields are colour-coded:
- **Gold** — Geometry fields (bytes per sector, sectors per track, sides, total sectors)
- **Blue** — FAT layout fields (FAT count, FAT size, reserved sectors)
- **Plum** — Directory fields (root entries, media descriptor)

In dark mode, all highlight colours are automatically muted to maintain text readability.

### Navigation

- **Scroll** to move through the data
- **Prev Data / Next Data** (toolbar buttons) — Skip to the previous/next non-zero byte region. Useful for mostly-empty disks where you want to find actual data quickly.
- **Click a file in the tree** — Jumps to that file's data
- **Click a sector in the Disk Surface Visualiser** — Jumps to that sector's byte offset

### Minimap

The thin strip to the right of the hex view is the **minimap**. It shows a compressed overview of the entire data currently loaded, with coloured bands indicating where data exists versus empty space. The current viewport position is shown as a highlighted region.

### Terminal Mode (`View > Terminal Theme Mode`)

Switches the hex viewer to a green-on-black retro terminal aesthetic with an optional CRT scanline effect. All data is displayed identically — only the visual rendering changes.

---

## 5. Disk Menu — Analysis Tools

### Disk Information (`Disk > Disk Information`)

*Pro feature.*

Opens a panel showing comprehensive disk metadata:
- Used and free space with percentages
- Cluster counts (total, used, free)
- FAT type and geometry mode
- OEM signature from boot sector
- Boot checksum value and bootability status

**Danger:** None — read-only operation.

---

### Edit Boot Sector (`Disk > Edit Boot Sector...`)

*Pro feature.*

Opens a structured dialog to view and edit individual BPB fields:
- Bytes per Sector, Sectors per Cluster, Reserved Sectors
- FAT Count, Root Entries, Total Sectors
- Media Descriptor, Sectors per FAT, Sectors per Track, Number of Sides

**What it does:** Directly modifies bytes in sector 0 of the in-memory disk image. Changes take effect immediately in the engine's geometry calculations.

**Danger:** Incorrectly editing BPB fields can make the disk unreadable by emulators and real hardware. Common mistakes:
- Setting Sectors per Cluster to 0 causes division-by-zero crashes in any tool that reads the disk
- Changing Total Sectors to a value larger than the image size causes out-of-bounds reads
- Modifying FAT Size or Reserved Sectors without understanding the layout will misalign all data access
- Only edit these fields if you understand FAT12 disk geometry. Always save a backup first.

---

### FAT Cluster Map (`Disk > FAT Cluster Map...`)

*Pro feature.*

Displays a colour-coded graphical grid of every cluster on the disk with live statistics:
- **Blue (Used)** — Data cluster in an active file's chain
- **Yellow (End-of-Chain)** — Last cluster of a file
- **White/Gray (Free)** — Available for allocation
- **Red (Bad)** — Marked as bad sector (0xFF7 in FAT)
- **Orange (Fragmented)** — Non-contiguous chain segment
- **Purple (Cross-Linked)** — Cluster claimed by multiple files (data corruption)
- **Magenta (Orphaned)** — Allocated in FAT but not reachable from any directory entry

**Danger:** None — read-only visualisation.

---

### View FAT Table (`Disk > View FAT Table`)

*Pro feature.*

Displays the raw FAT entry values for every cluster in a scrollable table.

---

### Format Disk (`Disk > Format Disk`)

*Pro feature.*

Wipes the FAT tables and root directory, returning the disk to a clean empty state. The boot sector geometry is preserved.

**What it does:** Zeros out both FAT copies and the root directory area. All file data remains physically on the disk but is no longer referenced by any FAT entry or directory entry.

**Danger:** All files on the disk are permanently lost (they still exist as raw bytes but cannot be accessed normally). This cannot be undone except by reloading the original file (if you haven't saved).

---

### Make Disk Bootable (`Disk > Make Disk Bootable`)

*Pro feature.*

Makes the disk bootable by setting the Atari TOS `0x1234` boot checksum — but only after verifying the disk has a valid boot target.

**Safety checks (performed automatically):**

1. **No AUTO folder** — The operation is blocked. A warning explains that you need to create an AUTO folder and add an executable program before the disk can be made bootable.
2. **AUTO folder exists but contains no executables** — The operation is blocked. A warning explains that you need to inject an Atari ST program (.PRG, .TOS, .TTP, .APP, or .GTP) into the AUTO folder.
3. **AUTO folder contains a valid executable** — A confirmation dialog shows which program was found (e.g. "Found executable: AUTO/GAME.PRG") and asks you to confirm before proceeding.

**What it does:** Reads all 256 words of the boot sector, sums them (minus the last word), calculates the value needed in the last word to reach `0x1234`, and writes it. This tells Atari TOS to execute the boot sector code on startup, which in turn loads and runs the first executable in the AUTO folder.

**How to make a blank disk bootable:**
1. Create a new disk (File > New Disk)
2. Create an AUTO folder (right-click > New Folder, name it `AUTO`)
3. Inject an Atari ST program into AUTO (File > Inject File to Disk, select a .PRG/.TOS file)
4. Use Disk > Make Disk Bootable — the engine will confirm the executable and set the checksum
5. Save the disk

**Note:** Newly created blank disks are intentionally non-bootable (checksum ≠ 0x1234) to prevent crashes when mounted in emulators or on real hardware.

---

### Search Disk (`Disk > Search Disk...`)

*Pro feature.*

Search for any text string or hex pattern (prefix with `0x`) across the entire disk image. Results list the byte offset and sector number. Double-click any result to jump to that location in the hex viewer.

**Danger:** None — read-only search.

---

### Pattern Scanner / Signatures (`Disk > Pattern Scanner / Signatures...`)

*Pro feature.*

Scans every byte of the disk for known Atari ST file signatures (PRG headers, MOD files, GEM metafiles, etc.) or a custom binary pattern. Results show the sector number and byte offset. Double-click any result to jump directly to that location in the hex viewer (automatically switches to Full Disk Mode).

**What it's for:** Finding files that exist on disk but aren't listed in the directory — deleted files, hidden data, or files on a disk with a corrupted FAT/directory.

**Danger:** None — read-only scan.

---

## 6. View Menu — Display Options

### Toggle Full Disk Hex View

Switches the hex viewer between showing the selected file's data and showing the entire raw disk image. When in Full Disk mode, colour-coded highlighting shows the boot sector, FAT areas, root directory, and data region.

### Terminal Theme Mode

Toggles the retro green-on-black CRT aesthetic in the hex viewer.

### Show . and .. Entries

Toggles visibility of the `.` and `..` directory entries in the file tree. These are standard FAT directory self-references and are hidden by default to reduce clutter.

### Theme > Light (Default) / Dark (Modern)

Switches between a clean light palette and a dark, low-glare palette. The theme affects the entire application including the file tree, hex viewer, menus, and all dialogs. On first launch, the application detects your OS dark/light preference automatically.

---

## 7. Repair Menu — Disk Recovery

All repair operations are **Pro features** and modify the in-memory disk image. Save a backup before attempting any repair.

### Fix Standard Geometry (`Repair > Fix Standard Geometry`)

Detects common BPB geometry errors and corrects them to match the disk image's actual size.

**What it does:** Compares the BPB's claimed total sectors against the image file size. If they disagree, it overwrites the BPB geometry fields (SPT, sides, total sectors) with values calculated from the file size using the Hatari lookup table.

**When to use:** When a disk loads with warnings like "BPB totalSectors ≠ image sectors" in the status bar, or when the geometry mode shows "Hatari Size-Based" instead of "BPB Standard".

**Danger:** If the disk intentionally uses non-standard geometry (some copy-protected or demoscene disks do), this will overwrite the original values. The original geometry is lost unless you saved a backup.

---

### Recalculate FAT Size (`Repair > Recalculate FAT Size`)

Recalculates the correct number of sectors needed for the FAT based on the total cluster count.

**What it does:** Computes `ceil(totalClusters * 1.5 / bytesPerSector)` and writes the result to the BPB FAT size field.

**When to use:** When the FAT Cluster Map shows clusters beyond the expected range, or when Disk Audit reports "FAT size mismatch".

**Danger:** If the FAT size increases, previously valid data sectors may now be treated as FAT sectors. If it decreases, FAT entries for high-numbered clusters become inaccessible.

---

### Align Root Directory Offset (`Repair > Align Root Directory Offset`)

Recalculates and corrects the root directory's starting sector based on the BPB fields.

**What it does:** Computes `rootStart = reservedSectors + (numFATs × fatSizeSectors)` and ensures the engine reads the root directory from that location.

**When to use:** When the file tree shows garbage filenames or no files on a disk that you know contains data.

**Danger:** If the BPB's reserved sectors or FAT size values are themselves wrong, this will point the root directory to the wrong location, making things worse. Fix geometry and FAT size first.

---

### Integrity Scanner / Disk Health Report (`Repair > Integrity Scanner / Disk Health Report...`)

Runs a comprehensive structural integrity check and opens a dialog showing:
- **Orphaned Clusters** — Allocated in FAT but not reachable from any directory entry
- **Cross-Linked Chains** — Two or more files sharing the same cluster (data corruption)
- **Bad Sectors** — Clusters marked `0xFF7` in the FAT
- **Fragmented Files** — Files whose cluster chain is non-contiguous
- **Overall health score** with repair recommendations

The dialog offers one-click repair buttons:
- **Adopt Orphans** — Creates directory entries for orphaned cluster chains
- **Sync FAT 1 to FAT 2** — Copies FAT 1 over FAT 2 to fix FAT inconsistencies
- **Wipe Slack Space** — Zeros unused bytes in the last cluster of every file

**Danger:** Repair operations modify the disk. "Adopt Orphans" creates new files that may contain garbage data. "Sync FAT" overwrites FAT 2 — if FAT 1 is the corrupted copy, this destroys the good backup. Always save a copy of the original disk before running repairs.

---

## 8. Help Menu

### Instructions (`Help > Instructions`)

Opens a scrollable dialog containing this manual.

### Disk Size Chart (`Help > Disk Size Chart`)

Reference table showing all 12 supported disk geometries with their tracks, sectors per track, sides, total sectors, and total size.

### Hex View Colour Legend (`Help > Hex View Colour Legend`)

Explains the colour coding used in the hex viewer across all modes:
- Full Disk View layout colours (boot, FAT, root, data, free)
- BPB field colours (geometry, FAT layout, directory)
- File view highlighting
- Terminal mode colours
- Disk Surface Visualiser colours

### About (`Help > About`)

Shows the application version, edition (Pro or Lite), copyright, author, and contact information. In the Lite version, includes a link to upgrade to Pro.

---

## 9. Toolbar

The toolbar provides quick access to common operations:

| Button | Action |
|--------|--------|
| **Open Disk** | Same as File > Open Disk |
| **View Disk Surface** | Opens the Disk Surface Visualiser window (Pro) |
| **Prev Data** | Skip hex view backward to previous non-zero data |
| **Next Data** | Skip hex view forward to next non-zero data |
| **Scavenger** (when visible) | Attempts forensic recovery of scrambled filenames |
| **Boot Sector** (when visible) | Shows boot sector information |
| Status text | Shows the currently loaded disk name and size |

---

## 10. Status Bar

The bottom bar shows two sections:

**Left side:** Current view context — "No disk loaded", "Viewing FILENAME (N bytes)", or disk name when in full disk/boot sector view.

**Right side:** Full disk geometry when a disk is loaded:
`Sectors 1640  Tracks 82  SPT 10  Sides 2  SPC 2  Geo BPB (Standard)  Files 22  Dirs 1  Free 15 KB`

Each field explained:
- **Sectors** — Total sector count
- **Tracks** — Number of tracks (typically 80 or 82)
- **SPT** — Sectors per track
- **Sides** — Number of sides (1 or 2)
- **SPC** — Sectors per cluster
- **Geo** — Geometry detection mode: `BPB (Standard)`, `Hatari Size-Based`, or `Manual Override`
- **Files** — Total file count
- **Dirs** — Total directory count
- **Free** — Remaining free space

---

## 11. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+O` | Open Disk |
| `Ctrl+S` | Save Disk |
| `Ctrl+Shift+S` | Save Disk As |
| `Ctrl+E` | Extract Selected File |
| `Ctrl+Q` | Exit |
| `Ctrl+I` | Inject File to Disk (Pro) |

---

## 12. Warnings & Dangers

### Operations That Can Lose Data

These operations modify the in-memory disk and cannot be undone:

| Operation | Risk Level | What Can Go Wrong |
|-----------|-----------|-------------------|
| **Save Disk** | HIGH | Overwrites original file permanently |
| **Format Disk** | HIGH | All files become inaccessible |
| **Delete File** | MEDIUM | File's directory entry and FAT chain are erased |
| **Fix Standard Geometry** | MEDIUM | Overwrites intentionally non-standard BPB values |
| **Sync FAT 1 to FAT 2** | MEDIUM | Destroys FAT 2 — if FAT 1 is corrupt, both copies are now corrupt |
| **Edit Boot Sector** | MEDIUM | Wrong values can make the disk unreadable everywhere |
| **Recalculate FAT Size** | MEDIUM | Can shift data region boundaries |
| **Align Root Directory** | LOW-MEDIUM | Points to wrong location if BPB is already wrong |
| **Adopt Orphans** | LOW | Creates files that may contain meaningless data |
| **Wipe Slack Space** | LOW | Zeros bytes — only affects unused space within clusters |
| **Make Disk Bootable** | LOW | Modifies 2 bytes in boot sector — Atari will try to execute boot code |

### Operations That Are Always Safe

These never modify the disk image:

- Open Disk, Close Disk
- Extract files (single or batch)
- All View menu options
- Disk Information, FAT Cluster Map, Pattern Scanner
- Search Disk
- Disk Health Report (the report itself — repair buttons do modify)
- All Help menu items

### The Golden Rule

**No changes are written to your original file until you explicitly choose Save or Save As.** You can experiment freely — just don't save until you're confident the changes are correct. If you make a mistake, close the disk without saving and reopen the original.

---

## 13. Best Practices

1. **Always save a backup** before using any Repair menu operation. Copy the original `.st` or `.msa` file to a safe location first.

2. **Run Disk Audit before extracting** from old or damaged disks. Cross-linked clusters mean two files share the same data — extracting them will give you corrupted files without any warning.

3. **Multi-select before batch extracting.** Ctrl+click the specific files you want in the tree, then use Extract All Files to Folder. Only your highlighted files will be extracted.

4. **Choose your save format wisely.** Use `.st` for maximum compatibility with emulators and real hardware. Use `.msa` when archiving or sharing — the per-track compression can reduce file size noticeably.

5. **Check the Geo field in the status bar.** If it says "Hatari Size-Based" instead of "BPB (Standard)", the disk has a non-standard or damaged BPB. The engine is working from file-size heuristics, which is usually correct but not guaranteed.

6. **Use the Disk Surface Visualiser** to understand disk layout visually before performing surgery. The colour-coded sectors show you exactly where boot, FAT, root, data, and free space live.

7. **Don't edit BPB fields unless you understand FAT12.** The Boot Sector Editor gives you direct access to the geometry fields that control how every tool interprets the disk. Wrong values don't just break this tool — they break every tool that reads the disk.

8. **Scavenger is for D-Bug encrypted disks.** When the engine detects scrambled filenames (high-bit encoded directory entries used by the D-Bug demo group), a gold warning bar appears with a "Forensic Recovery" button. This decodes the filenames without needing to emulate the disk's boot code.
