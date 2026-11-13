# Title facts

Short guest facts the port relies on; the evidence for each is in `docs/re-frontier.md`.

- The executable plus 22 loaded code modules (`docs/overlays.md`) form the code corpus.
- The first resident call boundary is `0x80089344`; the IRQ resume `0x80088A2C` is a mid-function re-entry, not a
  function start.
- The model-table reset `0x80041F38` includes the delay-slot increment at `0x80041FFC`.
- The retail pad buffers are `0x800CF8A0` and `0x800CF8C8`; active-low Cross is `0xBF` and release is `0xFF` in the
  measured packet byte.
- The authored projection leaves are `0x80083CD4` and `0x80083CF4`, initialized to `256/120/160`.
- `.RAW` payloads use Traveller's Tales' `DecompressRAW` LZ format (not RNC); a 14-byte chunk header keeps both CRCs,
  and it decodes the whole retail corpus to exact lengths.
