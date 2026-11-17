# Migration from in-snap Nix

The previous architecture put Nix, the store, and Hyprland inside ers. ADR 0009 moves Nix into an unprivileged NixOS system container and keeps Hyprland in the snap. This file is the inventory. The code it names is still in the tree.

The checkpoint of that work, before these pivot notes, is `8b9dfaf` on `legacy/in-snap-nix`.

## What already works

On delia, `ers` 0.1.0 revision `x1` is installed confinement `strict`, base `core26`. `snap run ers.bash` is `/snap/ers/x1/usr/bin/bash`. That shell has no `nix` and no `/nix`. The host has no Nix and no Nix accounts. Ubuntu Frame and the Frame kiosk were left running. Evidence is in `docs/integration-tests.md`.

`docs/frame-packaging.md` records which Ubuntu Frame packaging, interfaces, and launch scripts ers keeps. The Frame pin is `reference/ubuntu-frame` at `322a8fb67789b9fb831a5ab2cb486d3d5cba183f`.

## What does not work

Hyprland is not packaged and has not been launched. There is no `ers.desktop`. There is no Nix binary in the snap. There is no NixOS container.

The in-snap store writer was not finished. snapd creates `/var/snap/ers/common` as `root:root` mode `0755`. The install-hook profile cannot `chmod` or `chown` that directory. A child `$SNAP_COMMON/data` mode `1777` is what the shipped hook creates, and it is the wrong directory. An access ACL applied under the hook profile did make the directory mode `0777`, and that ACL is not in the shipped hook. Details are in `docs/nix-internal-user.md`.

## UID blocker

Nix's build pool is a set of host uids a privileged daemon `setuid`s to, one uid per concurrent build. Inside the strict snap:

- A passwd entry does not create a kernel uid.
- `ers.bash` runs as the invoking host user and cannot `setuid`.
- A snap service starts as root and may drop only to `_daemon_` (uid 584792).
- Host `nixbld` accounts, if a bootstrap script created them, are still not a `setuid` target the snap is allowed to use.

That is why the store and the build users move into the NixOS container, where NixOS can use its normal accounts under LXD's uid map.

## Treatment

| Existing work | Treatment |
| --- | --- |
| `snap/snapcraft.yaml` strict core26, `ers.bash` | Retain and adapt |
| ADR 0001 strict confinement | Retain |
| `docs/frame-packaging.md` and the Frame pin | Retain |
| Hyprland packaging and startup | Not started. Build with Nix at snap assembly and ship the result |
| Graphics and device access | Retain the Frame notes. Plugs are not connected on `ers.bash` yet |
| `docs/nix-internal-user.md` | Retain as the record. No runtime Nix in ers |
| `/nix/store` layout inside the snap | Not implemented. Do not add it |
| `$SNAP_COMMON/data` mode 1777 | Historical workaround. Remove when the hook no longer serves a Nix store |
| UID and build-user workarounds | Findings retained. Do not continue them inside the snap |
| Shell and terminal launchers | `ers.bash` stays a shipped shell. A desktop terminal runs in the container |
| ADRs 0002, 0004, 0008 | Superseded by ADR 0009. Files kept |
| Issues #4, #6, #7, #9, #10 | In-snap Nix path. Do not implement |
| Issue #2 | The strict shell is in place. The common-directory Nix writer is retired |
| Issue #5, ADR 0007 | Snap home stays proposed |
| Issue #8 | Hyprland remains the goal. It is assembled into the snap, not installed by a runtime Nix |

## Issues

#11 is the spec. #12 is Hyprland in the snap. #13 is the NixOS container. #14 is the first Wayland client and is blocked by #12 and #13. #15, #16, and #17 follow #14.

#1, #4, and #6–#10 are closed. #2 is closed: the strict shell is in place, and the common-directory Nix writer is withdrawn. #3 stays closed. #5 stays open, in triage, and is not on the path to #14.
