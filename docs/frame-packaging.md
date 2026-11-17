# Frame packaging kept for ers

Ubuntu Frame stays the packaging and session reference. Mir is not the runtime. The pin is `reference/ubuntu-frame` at `322a8fb67789b9fb831a5ab2cb486d3d5cba183f` on `core26`. This repository does not copy that tree into the ers sources.

The notes below are the read of that pin: which packaging, interfaces, graphics setup, and launch scripts ers keeps. The bootstrap snap in this repository ships only the pieces that belong to the bootstrap shell.

## Packaging

Kept, and used by the bootstrap snap:

| Frame packaging | ers |
| --- | --- |
| `confinement: strict` | `snap/snapcraft.yaml` |
| `base: core26` | `snap/snapcraft.yaml` |
| `compression: lzo` | `snap/snapcraft.yaml` |
| An install hook in `snap/hooks` | `snap/hooks/install` creates the snap's common data. snapd creates `$SNAP_COMMON` as root mode 0755, and the hook profile cannot chmod that directory. The hook makes `$SNAP_COMMON/data` mode 1777 so the user who runs ERS can write it. |

Frame builds `amd64`, `arm64`, `armhf`, and `riscv64`. ers builds `amd64`, which is the demonstration host. The other architectures are not kept.

Frame's install hook creates a diagnostic directory and sets daemon and launcher configuration. ers keeps the hook as the place that prepares common data. It does not keep the diagnostic directory or that configuration.

Not kept:

- The `mir-team/release` package repository
- Version and grade taken from the Mir package (`adopt-info`)
- Parts that build Mir, libcec, the Flutter launcher, fonts, icons, screenshots, or the Mir graphics platforms
- Apps `ubuntu-frame`, `launcher`, `daemon`, and `screenshot`
- Layouts that bind `/usr/share/icons` and `/usr/share/fonts`
- `MIR_SERVER_*`, `LIBINPUT_QUIRKS_DIR`, `XKB_CONFIG_ROOT`, `XDG_CURRENT_DESKTOP`, and the diagnostic path
- The `wayland` slot and the `ubuntu-frame-diagnostic` content slot
- `snap/local/plugs.json`
- Hooks `configure`, `connect-slot-gpu-2404`, `post-refresh`, and `unprepare-slot-gpu-2404`

## Interfaces

Kept for the compositor entry. They are not plugged on `ers.bash`. The bootstrap shell does not need them, and wiring them is later work.

| Interface | Why ers keeps it |
| --- | --- |
| `gpu-2604` | Content plug, `target: $SNAP/gpu-2604`, `default-provider: mesa-2604`. This is the graphics userspace plug Frame uses on core26. |
| `opengl` | Frame plugs this for compositing. It is the device access Hyprland needs for the GPU. |
| `hardware-observe` | Frame plugs this because libinput scans udev. Hyprland uses libinput for input devices. |

`home` is not a Frame plug. It is a separate boundary and is not part of this snap.

Not kept:

| Interface | Why ers does not keep it |
| --- | --- |
| `desktop-launch` | Desktop session integration for Frame's own clients. |
| `login-session-control` | Frame's user command uses it to run as a login on a virtual terminal. The Frame daemon does not plug it. ers is started with `snap run` and this shell does not take a virtual terminal. |
| `network-bind` | Frame uses it to operate as an X11 server. |
| `serial-port` | USB HDMI-CEC adapters. |
| `x11` | Nested on a host X11 session. Hyprland is the compositor. |
| `wayland` slot | Other snaps connecting to Frame. Programs inside ers share one execution context. |
| `wayplug` | Frame's launcher connecting to a Wayland slot. |
| `ubuntu-frame-diagnostic` | Frame's diagnostic screen. |

## Graphics setup

Kept for the compositor entry, and not part of the bootstrap shell:

- Plug `gpu-2604` as above.
- Put `bin/gpu-2604-wrapper` on the command chain of the graphical app. Frame puts that wrapper on every app, including the screenshot app. `ers.bash` does not use it. The bootstrap shell starts with no graphics stack and no Nix userspace generation.
- A part dumped from `https://github.com/canonical/gpu-snap.git`, then `gpu-2604-cleanup mesa-2604` during prime, priming only `bin/gpu-2604-wrapper`. That is how Frame avoids shipping the Mesa libraries the content snap provides.
- Do not set `LD_LIBRARY_PATH`, `LIBGL_DRIVERS_PATH`, `LIBVA_DRIVERS_PATH`, or `__EGL_VENDOR_LIBRARY_DIRS` in the snap. The wrapper sets them.
- No X11 `XErrorDB` layout.
- No Mir platform packages and no `MIR_SERVER_PLATFORM_PATH`.

Mesa from Nix is not the graphics userspace.

## Launch scripts

None of Frame's scripts in `scripts/bin/` are shipped.

| Script | Why ers does not keep it |
| --- | --- |
| `run-daemon` | Starts Frame as a system daemon. ers is not a daemon. |
| `run-frame` | Moves `XDG_RUNTIME_DIR` to its parent and clears `WAYLAND_DISPLAY` for Mir. |
| `run-user` | Copies Frame configuration into `$HOME`. |
| `run-launcher` | Exports Frame's launcher configuration. |
| `wayland-launch` | Waits for another snap's Wayland socket. |
| `run-client` | Points a client at the parent runtime directory. |
| `initialise-display-config` | Writes Frame's display configuration. |
| `screenshot` | Frame's screenshot tool. |

The launch script ers keeps is `bin/gpu-2604-wrapper`, from gpu-snap at build time, the same way Frame obtains it. It is not copied out of the Frame tree. It belongs to the compositor entry, not to this bootstrap snap.

## Bootstrap snap

`ers.bash` is bash shipped in the snap. The install hook prepares the snap's common data and makes `$SNAP_COMMON/data` writable by the user who runs ERS. Nix is not in the snap.
