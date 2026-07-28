# Celestial Resizer

**A Fabric mod that lets resource packs choose how big the sun and the moon are.**

Minecraft 1.21.11 · Author: **Mego** · License: MIT · Client-side only

---

## Why this mod exists

Minecraft draws the sun and the moon as flat quads whose on-screen size is baked into the client as two plain numbers. A pack can ship a beautiful 512×512 `sun.png`, but the game still squeezes it onto the same small quad — so the sun never actually looks any bigger.

There has never been a vanilla way to change that. This mod intercepts those two numbers and hands the decision to your resource pack. That is all it does.

**With no pack installed the mod does nothing at all.** Sizes stay exactly vanilla, so it is safe to ship alongside packs that do not use it.

---

## Quick start

Two files. That is the whole integration.

```
your-pack/
├── pack.mcmeta
└── assets/
    └── celestialresizer/
        └── celestial_bodies.json
```

**`assets/celestialresizer/celestial_bodies.json`**

```json
{
  "sun":  { "scale": 2.5 },
  "moon": { "scale": 1.5 }
}
```

**`pack.mcmeta`** — note the format *range*, see [pack.mcmeta rules](#packmcmeta-rules)

```json
{
  "pack": {
    "description": "My pack",
    "min_format": 75,
    "max_format": 75
  }
}
```

Enable the pack, or press `F3 + T` if it is already enabled. The sky updates immediately.

---

## `celestial_bodies.json` reference

The file has two optional keys, `sun` and `moon`. Each takes an object with two
optional fields:

| Field   | Type  | Meaning |
|---------|-------|---------|
| `scale` | float | Multiplier applied to the vanilla size. `1.0` = vanilla, `2.0` = twice as wide. |
| `size`  | float | Absolute half-size in world units. **Takes priority over `scale`** when both are present. |

Everything is optional and independent:

```json
{ "sun": { "scale": 4.0 } }
```

This is a complete, valid file. The moon is left untouched.

### `scale` vs `size`

Use **`scale`** unless you have a reason not to. It is relative, so it reads the same way to anyone looking at your pack ("twice as big") and it does not require you to memorize vanilla's numbers.

Use **`size`** when you want an exact figure — for example to make the sun and moon precisely equal, which `scale` cannot express because they start from different values.

```json
{
  "sun":  { "size": 40.0 },
  "moon": { "size": 40.0 }
}
```

### Understanding the numbers

`size` is a **half-size**: the quad extends that far in each direction from its center, so the body is twice that wide.

| | vanilla `size` | width across |
|---|---|---|
| Sun  | `30.0` | 60.0 |
| Moon | `20.0` | 40.0 |

So `"sun": { "size": 30.0 }` and `"sun": { "scale": 1.0 }` are the same thing.

Handy conversions for the sun:

| Look | `scale` | equivalent `size` |
|---|---|---|
| Half | `0.5` | `15.0` |
| Vanilla | `1.0` | `30.0` |
| Double | `2.0` | `60.0` |
| Demo pack | `2.5` | `75.0` |
| Huge | `5.0` | `150.0` |

### Values you can rely on

- Both fields accept decimals: `0.75`, `1.5`, `12.345`.
- Results are clamped to `0 … 100000`. Enormous values will not crash the game but they will look like a wall of texture, not a sun.
- `0` renders nothing — a legitimate way to hide a body completely.
- Negative values and `NaN` are treated as `0`.
- A missing key, a missing field, or an unreadable file falls back to vanilla.
- **Malformed JSON never crashes the game.** The mod logs a warning and uses vanilla sizes, so a typo costs you a reload, not a session.

---

## `pack.mcmeta` rules

This trips up almost everyone on modern versions, so it is worth stating plainly.

Since snapshot 25w31a (Minecraft 1.21.9+), a pack declaring a format above `64` **must** use a `min_format` / `max_format` range. The old single `pack_format` number is no longer sufficient. If you get it wrong, the game does not warn you in-game. It drops your pack and logs:

```
Couldn't load ... pack metadata: Pack declares support for version newer than 64, but is missing mandatory fields min_format and max_format
Removed resource pack ... because it is no longer compatible
```

For Minecraft 1.21.11 the resource pack format is **75**:

```json
{
  "pack": {
    "description": "My pack",
    "min_format": 75,
    "max_format": 75
  }
}
```

- **Omit `pack_format` entirely** unless your pack also supports formats below `65` — in which case it must be present *as well as* the range.
- Widen `max_format` if you want the pack to keep loading on later versions.

Note the mod itself targets 1.21.11.

---

## Your workflow

1. Enable your pack once.
2. Edit `celestial_bodies.json` in the pack folder.
3. Press `F3 + T` in-game.
4. Look at the sky.

Step 2–4 takes about two seconds, so tuning a value is fast. You do not need to disable and re-enable the pack, and you do not need to restart Minecraft.

> Working from a `.zip`? Unzip it into `resourcepacks/` as a folder while you are iterating — folders reload in place, zips generally do not.

### Confirming it took effect

Every reload the mod writes one line to `.minecraft/logs/latest.log`:

```
[Render thread/INFO] (CelestialResizer) Applied celestial body sizes: sun=75.0 moon=30.0 (vanilla 30.0 / 20.0)
```

Search the log for `CelestialResizer`. It reports the **final** numbers, so it tells you directly whether your file was read and what it produced. If it says `sun=30.0 moon=20.0`, your values did not land.

---

## Recipes

**Cinematic close sun**
```json
{ "sun": { "scale": 3.0 } }
```

**Distant, cold star** — a small sun for a barren or sci-fi world
```json
{ "sun": { "scale": 0.35 }, "moon": { "scale": 0.5 } }
```

**Looming moon** — big moon, ordinary sun
```json
{ "moon": { "scale": 4.0 } }
```

**Twin bodies** — identical sizes, only `size` can do this
```json
{ "sun": { "size": 35.0 }, "moon": { "size": 35.0 } }
```

**Sunless sky** — hide the sun, keep the moon
```json
{ "sun": { "size": 0.0 } }
```

**Subtle realism** — the real sun and moon are about half a degree wide; vanilla is far larger. Shrinking both a little reads as more grounded:
```json
{ "sun": { "scale": 0.6 }, "moon": { "scale": 0.6 } }
```

---

## Working with textures

Sizing is completely independent of texture resolution. The quad is stretched to whatever `sun.png` / `moon_phases.png` you provide, so:

- A bigger `scale` does **not** improve sharpness by itself. If you are scaling up, ship a higher-resolution texture too, or the result will look blurry (or blocky, depending on filtering).
- A high-resolution texture at vanilla size looks the same as before. Resolution and size are separate knobs — this mod only provides the second one.
- `moon_phases.png` keeps its 4×2 phase grid. Scaling the moon scales whichever phase tile is currently shown; it does not change the phase cycle.

Textures live in the usual vanilla places, unchanged by this mod:

```
assets/minecraft/textures/environment/sun.png
assets/minecraft/textures/environment/moon_phases.png
```

---

## What this mod does not do

Being explicit so you do not waste time looking for options that are not there:

- **It does not move the bodies.** Position, path, and speed across the sky are untouched.
- **It does not scale the glow.** The bright atmospheric halo around the sun and the sunrise/sunset coloring are drawn separately and keep their vanilla size. A very large sun will visibly outgrow its halo.
- **It does not change lighting.** Daylight level, block light, and mob spawning are entirely unaffected — this is a purely visual change.
- **It does not touch stars, clouds, or the End sky.**
- **It has no per-dimension or per-biome control.** One setting applies globally.
- **It adds no config screen.** The resource pack *is* the configuration.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Pack vanishes from the enabled list | `pack.mcmeta` uses the old single `pack_format` | Use `min_format` / `max_format` — see [above](#packmcmeta-rules) |
| Log says `sun=30.0 moon=20.0` | Your file was not found or not read | Check the path is exactly `assets/celestialresizer/celestial_bodies.json` |
| No `CelestialResizer` line in the log at all | The mod is not installed or not loading | Confirm the jar is in `mods/` alongside Fabric API |
| Log shows `Could not read ...` | Malformed JSON | Validate the file — trailing commas and smart quotes are the usual culprits |
| Nothing changes after `F3 + T` | Editing inside a `.zip` | Unzip to a folder while iterating |
| Body disappeared | Value resolved to `0` | Check for a negative number or a stray `"size": 0` |
| Sun looks blurry when enlarged | Texture resolution unchanged | Ship a higher-resolution `sun.png` |

If a pack lower in the stack also ships `celestial_bodies.json`, the **topmost** enabled pack wins, exactly like any other resource.

---

## Compatibility

- **Client-side only.** Not needed on servers, and it does nothing if installed on one. Works in singleplayer and on any multiplayer server.
- **Shader packs** (Iris and similar) often replace sky rendering wholesale. If a shader draws its own sun, it will override this mod.
- **Other sky mods** that rewrite the same rendering path may conflict. Mods that only change textures will not.

**Requirements**

| | |
|---|---|
| Minecraft | 1.21.11 |
| Fabric Loader | 0.18.1+ |
| [Fabric API](https://modrinth.com/mod/fabric-api) | 0.141.3+1.21.11 |
| Java | 21 |
