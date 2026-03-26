---
name: Disk pass rate and remaining failures
description: Current test results across all tested collections, fixes applied, and known failure patterns
type: project
---

Test suite: run via ./test_tools/build_and_run_test.sh

## Cumulative results (2026-03-24): 4364/4422 passing (98.69%)

| Collection | Pass | Total | Rate | Failures |
|------------|------|-------|------|----------|
| Automation Menu Disk | 2023 | 2029 | 99.7% | 6 (alt dumps + modified variants) |
| Vectronix Compilation | 592 | 628 | 94.3% | 36 (21 empty root, 15 garbage) |
| D-Bug Menu Disk | 468 | 471 | 99.4% | 3 (boot-encrypted multi-disk) |
| Fuzion CD | 313 | 314 | 99.7% | 1 (corrupted alt dump) |
| Medway Boys | 145 | 148 | 98.0% | 3 (garbage names) |
| Pompey Pirates (PP) | 128 | 131 | 97.7% | 3 (garbage + empty root) |
| Bad Brew Crew | 75 | 75 | 100% | 0 |
| Midland Boyz + misc | 56 | 59 | 94.9% | 3 (empty root — save disk + data disks) |
| Adrenalin UK CD | 57 | 57 | 100% | 0 |
| Awesome Menu Disk | 39 | 39 | 100% | 0 |
| **Total** | **3896** | **3951** | **98.61%** | **58** |

Note: D-Bug was tested twice (original + retest after SPC=0 fix). Deduplicated total above counts it once.

## Failure breakdown (58 disks)

### Empty/unreadable root directory (28 disks)
- Vectronix: 005, 030, 205, 302, 303, 367, 396, 458, 481, 485, 538, 550, 551, 552, 565, 573, 576, 577, 596, 597, 598 (21)
- Midland Boyz 19 Part 3a/3b (2) — multi-part data disks
- Cannon Fodder Save (1) — game save disk, non-standard
- D-Bug 117 Disk 1 (1) — boot-encrypted
- Automation 031 [b] (1), 319 Part C [m LGD] (1) — corrupted/modified copies
- Fuzion CD 124 [a] (1) — corrupted alt dump
- PP_007 (1)

### Garbage/scrambled filenames (27 disks)
- Vectronix: 163, 207, 281, 289, 377, 379, 399, 410, 425, 474, 544, 558, 570, 571, 572 (15)
- Automation: 032 [b], 319 Part B [m LGD], 380, 380 [m LGD] (4)
- Medway Boys: 064, 097, 112 (3)
- PP_001, PP_028 (2)
- D-Bug 075 Disk 1, 075 Disk 1 [a] (2) — boot-encrypted
- Automation 319 Part B [m LGD] (1)

### Boot-encrypted (not fixable by static analysis) (3 disks)
- D-Bug 075 (Disk 1) x2 variants, D-Bug 117 (Disk 1)

## Bugs fixed during testing (2026-03-24)

1. **SPC=0 division-by-zero** — D-Bug 075 (Disk 2 of 2) had `sectorsPerCluster=0` in BPB causing FPE. Added guard across all code paths: `(bpb.sectorsPerCluster > 0) ? bpb.sectorsPerCluster : 2`.

2. **Segfault from out-of-bounds clusterToByteOffset** — Vectronix disks with mismatched BPB geometry caused offsets past image end. Added `if (offset >= m_image.size()) return 0;` in `clusterToByteOffset()`.

3. **Segfault from readSubDirectory buffer overrun** — Added `if (offset + ((i + 1) * 32) > m_image.size()) break;` inside the directory entry loop.

4. **Infinite recursion in test reader** — Added depth limit of 16 in `listDirectory()` to prevent stack overflow on circular directory references.

## Previous fixes (2026-03-22)

1. `validateGeometryBySize()` 839680-byte override removed — was forcing wrong fatSize for D-Bug disks
2. Hatari BPB totalSectors validation — reject BPB if totalSectors != imageSize/512
3. `readSubDirectory()` break→continue — don't stop at unusual entries
4. `isDirectoryGarbage()` high-bit fallback — try stripping 0x80 before flagging
5. `huntForRootDirectory()` false positive guard — skip hunt if BPB is self-consistent
6. Underscore padding excluded from garbage detection

## Key technical notes

- D-Bug menu disks at 820KB use fatSize=3 (not 5). The BPB is correct.
- D-Bug multi-disk sets: disk 1 has boot code that decrypts disk 2's directory in RAM.
- Vectronix is the roughest collection — many non-standard formats and custom loaders.
- All `[b]` alternate dumps and `[m LGD]` modified variants that fail have passing primary versions.

