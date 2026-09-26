# MeshCore-Repeater-RegionSave

Patched fork of [MeshCore](https://github.com/meshcore-dev/MeshCore) firmware — fixes region settings not persisting after reboot on Repeater nodes.

## The bug

Regions set via CLI (`region put`, `region allowf`, `region denyf`, `region remove`) return `OK` but are never written to flash. On reboot the region table resets to the default `*^ F`, silently discarding the configuration.

## The fix

`_callbacks->saveRegions()` was defined but never called after mutating region commands in `src/helpers/CommonCLI.cpp`. This patch adds the call after each of the four commands above, matching the behavior already used by `region default`.

In effect, this makes `put`/`allowf`/`denyf`/`remove` do automatically what the existing manual `region save` command does — no extra step needed after each change.

Firmware version string is tagged `v1.17.1-saveregions` so the fix is confirmable via the `ver` CLI command after flashing.

## Supported boards

| Board | Firmware |
|-------|----------|
| Heltec WiFi LoRa 32 V3 | `meshcore_v3_saveregions.bin` / `meshcore_v3_merged.bin` |
| Heltec WiFi LoRa 32 V4 | `meshcore_v4_saveregions.bin` / `meshcore_v4_merged.bin` |

> Use **non-merged** to update without erasing settings.
> Use **merged** for a clean install on a new board.

## Flashing

1. Download the correct `.bin` for your board from [Releases](https://github.com/RemusArvay/MeshCore-Repeater-RegionSave/releases)
2. Flash via `esptool.py` or [observer.gessaman.com](https://observer.gessaman.com) (Custom Firmware)
3. After boot, verify with `ver` in the CLI — should report `v1.17.1-saveregions (Build: 14 Aug 2026)`

## Verifying the fix

```
region put ro *
region list allowed
<reboot>
region list allowed
```

The region should still be listed after reboot.

## Building from source

```bash
git clone https://github.com/RemusArvay/MeshCore-Repeater-RegionSave.git
cd MeshCore-Repeater-RegionSave
pio run -e Heltec_v3_repeater
pio run -e heltec_v4_repeater
```

## Credits

- Original firmware: [meshcore-dev/MeshCore](https://github.com/meshcore-dev/MeshCore)
- MeshCore project: [meshcore.io](https://meshcore.io)
