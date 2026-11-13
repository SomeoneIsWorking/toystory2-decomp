# Images and modules

The disc boots `SLUS_008.93`; the game then loads code modules into two fixed slots. Every module is a raw image (no
header) loaded at its slot address. The slots and module tables come from psx-toy-story-2's overlay owners
(`game/overlay/level_slot_image.h`, `game/overlay/shared_slot_image.h`).

| slot | address | window | images |
|---|---|---|---|
| LEVEL | `0x800D12C0` | 19,040 B | `LEVELnn/LEVEL.BIN` and `LEVELnn/LEVEL1.BIN` of levels 01..10, alternatives of one another |
| SHARED | `0x800D5D20` | 510,960 B | `BITS/MEMORY.BIN` (UI screens) and `FMV/FMV.BIN` (movie player, entry `0x800D6628`) |

`LEVEL00/LEVEL.BIN` is a four-byte placeholder, not an image.

## Executable

| image | load | file | gp | SHA-1 |
|---|---|---|---|---|
| `SLUS_008.93` | `0x80010000` | 598,016 B, text from file `0x800` | `0x800A0CD8` | `f90c9cd6b4fc9845adfe34e306b7df393bf9154c` |

gp comes from splat's `create_config`; it agrees with the gp-relative words the port names (gp+`0x7C0` =
`0x800A1498`, gp+`0x790` = `0x800A1468`). The header carries 0. Everything from `0x800A1800` to the LEVEL slot is
bss; its size is not recorded yet.

Text, by vram (file offset = vram - `0x8000F800`):

| from | to | what |
|---|---|---|
| `0x80010000` | `0x800206EC` | code |
| `0x800206EC` | `0x80020EF0` | data inside text: no return and 118 words that are not R3000/GTE instructions |
| `0x80020EF0` | `0x800218B4` | code |
| `0x800218B4` | `0x800239F4` | data inside text: pointer tables, then strings (the asset-set names at `0x80021D18`) |
| `0x800239F4` | `0x80082E0C` | code, ending in crt0's `jal main` and `break` |
| `0x80082E0C` | `0x80082E1C` | four padding words |
| `0x80082E1C` | `0x80097DC8` | code: PsyQ libraries (their `Ps` stamp words stay inside functions as `.word`) |
| `0x80097DC8` | `0x800A1800` | data |

## Modules

File offsets of each split. Every LEVEL module opens with one word, a small per-file number (`LEVEL.BIN` of levels
01..10 `0x04..0x0D`, `LEVEL1.BIN` of 01..09 `0x0E..0x16`, LEVEL10/LEVEL1 `0x0F` like LEVEL02/LEVEL1; the LEVEL00
placeholder's one word is `0x04`), then code; LEVEL04 and LEVEL08 then hold a jump table before their first function. The
24-byte `LEVEL1.BIN` of levels 07..09 and the 624-byte one of level 06 hold two empty functions (`jr ra; nop`) at
`0x4..0x14`, then data.

| image | slot | size | code from | data from | SHA-1 |
|---|---|---|---|---|---|
| `BITS/MEMORY.BIN` | SHARED | 0xF750 | 0x8F4 | 0xC904 | `b735c3e5471c2641c204077ebb9bb437690b6a5c` |
| `FMV/FMV.BIN` | SHARED | 0x7CBF0 | 0x908 | 0x45CC | `47321941eaa9a841bf48c8fa64edc70df42db87f` |
| `LEVEL01/LEVEL.BIN` | LEVEL | 0x362C | 0x4 | 0x28BC | `7f512da2fd70117db4061fce392b438bafc2810a` |
| `LEVEL01/LEVEL1.BIN` | LEVEL | 0x4938 | 0x4 | 0x4024 | `05f4426dbe842c7d6e8c9df27398864b765a90d5` |
| `LEVEL02/LEVEL.BIN` | LEVEL | 0x4A60 | 0x4 | 0x43BC | `d2e455282720b3dd6a8d3ca4a964b6c9cf3547d6` |
| `LEVEL02/LEVEL1.BIN` | LEVEL | 0x1480 | 0x4 | 0x1428 | `91ea59b27352238c8219b1796d1f5364f8b3f0af` |
| `LEVEL03/LEVEL.BIN` | LEVEL | 0x149C | 0x4 | 0x141C | `cbcaf0a478770dcd4c9a350beb32ac7a52935d18` |
| `LEVEL03/LEVEL1.BIN` | LEVEL | 0x2888 | 0x4 | 0x2124 | `100d16a60a9c8ef329c865d1fb791dd910c5a215` |
| `LEVEL04/LEVEL.BIN` | LEVEL | 0x4154 | 0x34 | 0x3870 | `b555714096ccd8592e29f162ff3daa832eaa01d0` |
| `LEVEL04/LEVEL1.BIN` | LEVEL | 0x2FC8 | 0x4 | 0x2984 | `8cc7552037078172d15fc1855236a2119ae753e3` |
| `LEVEL05/LEVEL.BIN` | LEVEL | 0x3478 | 0x4 | 0x2F0C | `b5a692f53f01ae73daf5562cdf108a0ecb57098d` |
| `LEVEL05/LEVEL1.BIN` | LEVEL | 0x1FD4 | 0x4 | 0x1F0C | `02401765f43780458507651972866228c55927cd` |
| `LEVEL06/LEVEL.BIN` | LEVEL | 0xE18 | 0x4 | 0xDC8 | `10a99b0375e414a1efc8494e9fb7b44075c13620` |
| `LEVEL06/LEVEL1.BIN` | LEVEL | 0x270 | 0x4 | 0x14 | `bbd8c81956d4de0c04b4ca070c588b6a2364c90a` |
| `LEVEL07/LEVEL.BIN` | LEVEL | 0x3254 | 0x4 | 0x2918 | `fbba4f35b653e9ba57a787fe9e2e215151e1eebc` |
| `LEVEL07/LEVEL1.BIN` | LEVEL | 0x18 | 0x4 | 0x14 | `81feee7e87987916a39bf1f5ed058df1537889f7` |
| `LEVEL08/LEVEL.BIN` | LEVEL | 0x326C | 0x1C | 0x2C20 | `f0f701d74b189ddadc5a9e4679e7200b70828ac3` |
| `LEVEL08/LEVEL1.BIN` | LEVEL | 0x18 | 0x4 | 0x14 | `5835612c52f786c4acdfd7ea58f242a747f6c7b9` |
| `LEVEL09/LEVEL.BIN` | LEVEL | 0x1DB8 | 0x4 | 0x1D2C | `e854d618f9f74f52336f039e1033533de57c1a0d` |
| `LEVEL09/LEVEL1.BIN` | LEVEL | 0x18 | 0x4 | 0x14 | `ae6f105bd9ae48f2a13cd32b76b808a2bc0af7c3` |
| `LEVEL10/LEVEL.BIN` | LEVEL | 0x47D8 | 0x4 | 0x3E38 | `deddf9f5b1e84b4b65e4011302383860c90c5f69` |
| `LEVEL10/LEVEL1.BIN` | LEVEL | 0xA5C | 0x4 | 0x920 | `0081f5bb00ac350433c123eff63f7bc6b2abdbd7` |

## Boundaries

Code starts at the first function: the first `addiu sp, sp, -n` prologue, or the leaf before it (LEVEL07's code
opens with a leaf at `0x4`). It ends after the last `jr ra` and its delay slot. FMV's two later `jr ra` words
(`0x800EAF84`, `0x800EAF88`) lie inside its decoder tables. A data run inside text is a stretch with no `jr ra`
holding words that are not R3000/GTE instructions, from the delay slot of the return before it to the next
prologue. Refine a range when a function is named that falls outside it.

A module config reads `config/symbols/slus_008.93.txt` for the resident names its code calls; splat lists those as
symbols it cannot place in the module's segment, which is expected.
