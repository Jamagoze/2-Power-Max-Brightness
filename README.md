# Double Tap Power → Max Brightness

A tiny Magisk module that sets the screen to **maximum brightness when you double tap the power button**.

It only *listens* to the power key. It never grabs, blocks, or remaps it, so the power menu, long press and power off all work exactly as normal.

## Features

- Double tap power → max brightness
- Safe: read-only key listener, nothing is intercepted
- No polling: uses a blocking `getevent` read, so it uses no CPU while idle
- Protected from being killed by the system's low-memory killer
- No logging, nothing written to storage
- Adjustable tap speed

## Requirements

- Rooted device with Magisk
- A device that exposes its screen backlight in `/sys/class/backlight/*`

Tested on Redmi Pad SE (Snapdragon 680), Android 14 / HyperOS.

## Install

1. Download `double_tap_power_brightness.zip` from the releases.
2. Open Magisk → Modules → Install from storage → pick the zip.
3. Reboot.

## Usage

Double tap the power button. The screen sets itself to max brightness a moment after it wakes.

## Configuration

Edit `/data/adb/modules/double_tap_power_brightness/config.txt`:

| Option | Default | Description |
|---|---|---|
| `INTERVAL_MS` | `500` | Max time between the two taps, in milliseconds |
| `APPLY_DELAY` | `0.6` | Seconds to wait before applying brightness (lets the screen finish waking) |

Reboot after changing it.

## How it works

1. `getevent -t` watches the input devices for the power key (`KEY_POWER`, code `0x74`).
2. Two presses within `INTERVAL_MS` count as a double tap. Timing uses the kernel's own event timestamps, so a slow wake-up can't turn a real double tap into a miss.
3. The module writes each backlight node's own `max_brightness` into its `brightness` file, then calls `cmd display set-brightness 1.0` so Android's display state stays in sync. This is repeated a few times to survive the system resetting brightness while the screen wakes.
4. The script lowers its own `oom_score_adj` so Android doesn't kill it in the background.

## Compatibility notes

- Power key detection should work on almost any rooted Android device.
- Setting brightness depends on the device having a standard backlight node in `/sys/class/backlight/`. If your device uses a different path, brightness may not change.
- If your device has its own double-press power shortcut (such as opening the camera), it will run alongside this module. Disable it in system settings if it gets in the way.

## Version history

### v1.1
- Fixed the module sometimes not working after a reboot. The old start-up check could mistake a reused process ID for a running copy and quit silently; the check is removed.
- The script and its listener are now protected from being killed by the system.
- Tap timing now uses kernel event timestamps instead of the time the script processed the event.

### v1.0
- Initial release: double tap power sets max brightness.
- Read-only `getevent` listener on all input devices, no polling, no logging.
- Configurable tap interval and apply delay.

## Uninstall

Remove the module in Magisk and reboot.

## License

MIT
