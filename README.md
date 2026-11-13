# Toy Story 2 decomp

A decompilation of **Toy Story 2: Buzz Lightyear to the Rescue!** (USA, `SLUS-00893`), set up with
[splat](https://github.com/ethteck/splat). It currently holds the split configuration for every code image on the
disc, a symbol map from our own reverse engineering, and the RE notes. No C is matched yet.

The PC port [psx-toy-story-2](https://github.com/SomeoneIsWorking/psx-toy-story-2) consumes this repository as a
submodule and reads its guest addresses from the symbol map.

## Bring your own disc

No game bytes are ever committed here: not the executable, not the modules, not the split assembly or data. Copy
the files from your own disc into `disc/` (gitignored), keeping the disc's layout:

```
disc/SLUS_008.93
disc/BITS/MEMORY.BIN
disc/FMV/FMV.BIN
disc/LEVEL01/LEVEL.BIN  LEVEL1.BIN   .. disc/LEVEL10/LEVEL.BIN  LEVEL1.BIN
```

Each config carries the file's SHA-1 and splat refuses a mismatch. The images are listed in `docs/overlays.md`.

## Split

```
uv run --frozen splat split config/slus_008.93.yaml
uv run --frozen splat split config/overlays/level01.yaml     # one config per module
```

Output goes to `asm/<image>/` and `build/`, both gitignored. splat is pinned in `pyproject.toml`/`uv.lock`.

## Layout

| path | what |
|---|---|
| `config/slus_008.93.yaml`, `config/overlays/*.yaml` | splat configs, one per image |
| `config/symbols/<image>.txt` | the symbol map, splat `symbol_addrs` format; module configs also read `slus_008.93.txt` |
| `docs/overlays.md` | the image set, load slots and segment boundaries |
| `docs/re-frontier.md` | the ordered RE evidence chain |
| `docs/notes/` | behaviour notes on guest routines |
| `docs/prior-art.md` | other Toy Story 2 work, licences and what we take |

Notes cite psx-toy-story-2 paths (`game/...`, `docs/issues/...`) where a port owner or an issue holds the evidence.

## Licence

MIT (`LICENSE`), covering this repository's own files only. Toy Story 2 is © Disney/Pixar, developed by
Traveller's Tales and published by Activision; nothing of it is distributed here.

Built with AI assistance (Claude and other LLMs).
