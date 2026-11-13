# Frame loop, resident update and the world pass

How `main` (`0x8007A9E8`) builds and shows one gameplay frame. The resident loop runs at two display fields a logic
frame.

## The deferred display `0x8003A888`

Reached through `0x80021028` once per field. It draws the OT the previous update built, in order:

1. profiler mark;
2. `0x8003DEC4(buffer)`: screen bars into the OT, the env's clear colour;
3. `0x8003E498(buffer)`, a VRAM copy (effect mode 1), or `0x8003EFD4`, OT links (mode 2); the effect runs while
   `gp+0x588 + gp+0x610 != 0`, the copy when either mode is 1;
4. `0x8002DC2C`: the colour tint into the OT;
5. DrawOTag of the buffer's OT head (`+0x2B4 + 0x57B * 4`, ClearOTagR);
6. the pad decode `0x8003B33C`, the buffer swap and ClearOTagR of the next buffer.

The effect and tint calls take no arguments.

## The resident update

`0x8007B254` (the play loop; `0x8007B850`, the orbit camera, while `0x800A10F4` is set) ends with:

| step | what it does |
|---|---|
| `0x80078780` | logic: palette-cycle counters, timers, exit fade countdown, the fade step `0x800775CC` |
| `0x80074F80` | HUD and pause menu: logic (state machine, sound starts `0x80072C7C`) and HUD packets into the OT |
| `0x8007863C` | builds the draw list `0x800A4298` from the active list `0x800B18A8` plus the player `0x800B2188` |
| `0x80046A88` | the draw-mode packet |
| `0x8002A070` | the world pass |

The orbit update calls the pause menu `0x800742C4` after the world pass.

## The world pass `0x8002A070`

Camera into the GTE (`0x80083C84(0x1F800394)`), sky sprites `0x8001FB64`, the per-view matrix
(`0x8001E8D4(0x800A1618, 0x1F8003B4)`), visibility lists `0x80027724` (into `0x800BB4D8` and `0x800C0AB0`), world
objects `0x8002622C`, effects (`0x80044C18`, `0x80046E50`, `0x80048214`), actors `0x80025E44` (sorts the draw list,
then `0x8002518C` per actor), the VRAM probe `0x80040160`, profiler marks `0x8003E938`, DrawSync `0x80085A54`.

Besides drawing it sorts the draw list in place, poses animated actors (`0x800426E0` writes the part table
`0x800A9618`), steps the OT write cursor `0x800A151C`, consumes a streaming list (`+0x18` of the nearest area), and
calls DrawSync and GetRCnt. It links into a table the display cleared, so it rebuilds its OT entries each run.

`0x80040160` reads eight VRAM pixels back (StoreImage at `0x3C0,0x7F/0x7D`) and writes them again while `0x800A1568`
is set for the buffer. The profiler mark `0x8003E938` reads root counter 1.

### World objects `0x8002622C(list, count, ot, pass)`

Each entry of a visibility list points at a record whose byte `+0xE` selects the draw (`& 0x6F`), byte `+0xF` its
area, and `+0x10` a group: per-buffer packet pointers at `+0x18 + buffer * 4` (buffer `0x800A11E0`), per-buffer
skip counters at `+0x26 + buffer`, and at `+0x14` the instance: position s32 `+0x00/+0x04/+0x08`, angles s16
`+0x0C/+0x0E/+0x10`, mesh pointer `+0x14` (`+0x1C` for kinds 9/0xC and 0xA). Kinds 1/4 and 9/0xC draw
through `0x800100E4`, 2 through the hierarchical drawer `0x8002AC40`, 3 through `0x8002B6F0`, 0xA through
`0x8002AC40` with a matrix scaled by s16 `+0x12` (widened by `512 / 320`) and `+0x14`. The depth bias table is `0x80097DC8`, indexed by byte `+0x12 & 0xF` (`+0x18` for
9/0xC).

## The camera `0x8002C848`

Leaves the camera's MATRIX at `0x1F800394` (RotMatrix of its angles, the x row widened by `512 / 320`, rows 0 and 1
scaled by the shake `gp+0x9C4`), its position at `0x1F800374..7C` and that position `>> 5` at `0x1F800384..8C`, and
publishes the angles as three s16 at `0x800A1618`. The world pass reads the matrix and the published angles (the
sky and the screen effect). `0x80065CD0` places the camera afresh; `0x80068E2C` switches its source.

## Actor and part records

The actor pool is 64 records of `0x98` at `0x800C2EA0`; slot i is bound to placement `0x800B57A8 + i * 0x20`. The
player is the same layout at `0x800B2188`. `0x8002518C` reads position s32 `+0x00/+0x04/+0x08`, angles s16
`+0x0C/+0x0E/+0x10`, model `+0x14`, animation indices s16 `+0x12/+0x16` and their 16.16 frame words `+0x18/+0x1C`.
`0x800426E0` poses the model between frame `>> 16` and the next by the low half, the last frame towards frame 0, so
a frame word turns round the animation's frame count (`+4 & 0x7FFF` of the animation header; the model table
`0x800C7268` holds one pointer per model, whose animation table starts at `+8`). `0x8006A9FC` resets an actor.

The part table `0x800A9618` holds `0x6C`-byte records: offset s32 `+0x00/+0x04/+0x08` and a rotation of nine s16 from
`+0x58`, the last word's high half another field. A model's parts run from the index its `+4` holds to the one its
`+6` holds. An actor with an animation has its parts posed in the draw; for one without (the player), its update
poses them.
