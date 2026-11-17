# Integration tests

A built snap, a booted container, a visible window, and an accelerated window are separate results. devmode is not a strict-confinement result.

How to reach delia stays in the README. These notes do not repeat connection details.

## Bootstrap shell (pre-pivot)

Observed on delia against installed `ers` 0.1.0 revision `x1`. Not re-run for the pivot.

| Check | Result |
| --- | --- |
| `confinement: strict` and `base: core26` in `/snap/ers/current/meta/snap.yaml` | Pass |
| `snap run ers.bash` executable is `/snap/ers/x1/usr/bin/bash` | Pass |
| `command -v nix` fails and `/nix` is absent inside the shell | Pass |
| Host has no `nix` binary, no `/nix`, and no Nix accounts | Pass |
| Ubuntu Frame daemon and the Frame kiosk stay active | Pass |
| Install hook `chmod` of `/var/snap/ers/common` | Fail. AppArmor deny on `snap.ers.hook.install` |
| Shipped hook creates `/var/snap/ers/common/data` mode `1777` | The hook does this. It is not the store directory ADR 0004 named |
| POSIX ACL via `setxattr` under that hook profile | Mode became `0777` and the invoking user created a file. Not shipped in the hook |

Hyprland was not launched. No Wayland client was launched.

## NixOS image (WSL LXD 6.10.1)

| Command | Result |
| --- | --- |
| `lxc image list images: nixos` | Empty. `images` is `https://images.lxd.canonical.com` |
| `lxc image show images:nixos/26.05` | Not found |
| `lxc image show images:nixos/25.05` | Not found |
| `lxc image show lxc-images:nixos/26.05` | Not found. `lxc-images` is `https://images.linuxcontainers.org` |
| `lxc image show lxc-images:nixos/unstable` | Not found |
| `lxc image import` of `incus.tar.xz` then `rootfs.squashfs` for `nixos/26.05` build `20261117_01:01` | Imported. Fingerprint `301dcbd4f19c81b3ea3d7340553368bda00c68b0f107e790ffd31d08c0b714c5` |

The simplestreams product exists. Its metadata file type is `incus.tar.xz`, so the LXD client does not offer the alias. The same two files import.

| Check on instance `ers-nixos` | Result |
| --- | --- |
| `security.privileged` | `false` |
| `security.nesting` | `true` |
| `/etc/os-release` | NixOS 26.05 (Yarara), build `26.05.14561.b349185bb79c` |
| `systemctl is-system-running` | `running` |
| `nix --version` | `nix (Nix) 2.34.9` |
| `/root/write-probe` after `lxc restart` | Still present |
| `developer` after `nixos-rebuild switch` | uid 1000, group `wheel`. `sudo -n id` prints `uid=0(root)` |
| `/home/developer/home-probe` after `lxc restart` | Still present, owned by `developer` |
| `/proc/self/uid_map` | `0 1000000 1000000000` |
| First `sudo` in the few seconds after restart | PAM account error, then the next `sudo` succeeds |

This instance is on the WSL LXD.

## NixOS container on the host

Same image fingerprint `301dcbd4f19c81b3ea3d7340553368bda00c68b0f107e790ffd31d08c0b714c5`, imported by file and started as `ers-nixos`.

| Check | Result |
| --- | --- |
| `security.privileged` / `security.nesting` | `false` / `true` |
| `nix --version` | `nix (Nix) 2.34.9` |
| `systemctl is-system-running` | `running` |
| `developer` `sudo -n id` after restart | `uid=0(root)` |
| `/home/developer/home-probe` after restart | Still present, owned by `developer` |
| `/proc/self/uid_map` | `0 1000000 1000000000` |
| Ubuntu Frame, the kiosk, and `ers` | Still active. `ers` remains 0.1.0 revision `x1` |

## Pivot milestones

| Milestone | Result |
| --- | --- |
| M1 Hyprland under strict confinement, LXD stopped | Not run |
| M2 NixOS container: systemd, Nix, internal user, restart | Pass on the host. See above |
| M3 Client process in the container, window on the Hyprland display | Not run |
| M4 Accelerated client without display-control devices in the container | Not run |
