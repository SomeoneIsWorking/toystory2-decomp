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

The entries live in per-area arrays of `0x14`-byte records: cull-sphere centre s32 `+0x00/+0x04/+0x08`, radius u16
`+0x0C`, kind `+0xE`, area `+0xF`, group `+0x10`. `0x80020074(entries, view, eye)` walks them area by area (area
records of `0x34` at `0x800A8860`, byte `+0x10` the area's entry bytes) and fills `0x800BB4D8` with the visible
entries whose kind lacks `0x40` and `0x800C0AB0` with those that have it; it returns the two counts as low and high
halves. `0x80027724(count, &previousCount, list, previous)` merges each sorted list against the previous frame's
(`0x800A8C70` count `0x800A146C`, `0x800B19C8` count `0x800A14F4`): an entry that left has its packets released
through `0x80017C60` (the OT cursor `0x800A151C`), one that entered has its counters reset and packets taken by
`0x80023C14`, and the entries holding packets become the previous list.

The group's first nine s16 are the object's rotation matrix, which `0x8001F848(group)` (kinds 1/4/9/0xC with
nonzero angles) composes with the camera rotation. `0x8002DFFC(index, sx, sy, sz)`, `0x8002E2A4` and `0x8002E52C`
set it from an `AreaObjects` (`0x800A1274`: a count, then entry pointers) index: per-axis scales into instance
`+0x12/+0x14/+0x16`, then RotMatrix of the instance angles `+0x0C` into the group, each column scaled, stored
column by column. Their callers are update code (`0x80065CD0`, `0x800660B0`, `0x80066328`, `0x8005AE8C`,
`0x8006F938`, `0x80076334`..`0x8007678C`, `0x80077C68`, `0x80077D44`, `0x8007844C`, `0x8007855C`,
`0x8007D0E4`..`0x8007D4D8`).

In Andy's House, idle, one of the 157 visible instances moves: `0x8014FA24` (kind `0x49`), its position still and
its angle halfwords `+0x0C/+0x0E/+0x10` advancing by 40/80/80 a tick. Its group matrix (`0x801AFC4C`) is rewritten
once a tick by the draw section: a call of the world pass `0x8002A070` alone rewrites it, and with frame
interpolation on every replay of the section rewrites it to the same value. Which instruction stores it is not
recorded yet; under the dynarec a store watch reports the registers of the last synchronisation, not of the store.

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
