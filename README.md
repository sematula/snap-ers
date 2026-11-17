# snap-ers

Edge Runner Shell (ERS) is a strictly confined Ubuntu snap that provides a Nix-managed Linux userspace and launches upstream Hyprland as the primary Wayland compositor.

The host stays a normal Ubuntu system. Snapd confines ERS. Nix manages the software inside that confinement. Hyprland manages the desktop. ERS only owns the integration that makes those pieces work together.

ERS is intended to run first on Ubuntu Server 26.04 LTS, and later on Ubuntu Core 26.

```text
Ubuntu Server / Ubuntu Core
    │
    ├── Kernel, device drivers, systemd, snapd
    │
    └── ERS (strictly confined snap)
         │
         ├── Minimal immutable bootstrap
         │    ├── Nix executable
         │    ├── Nix runtime configuration
         │    ├── Persistent Nix store
         │    └── ERS launcher and recovery utilities
         │
         └── Nix-managed userspace
              ├── Hyprland and Aquamarine
              ├── Shell and terminal
              ├── Desktop utilities
              ├── User applications
              └── User configuration
```

The first goal is a demonstration: upstream Nix can install, compile, and run software inside strict snap confinement, including a Nix-managed Hyprland desktop. ERS is not a new package manager, compositor, or security framework.

## Requirements

- Confinement is `strict`. AppArmor, seccomp, mount namespaces, and the declared snap interfaces are the security boundary. The host is trusted. ERS and everything running inside it are not.
- Nix is upstream Nix, in daemonless single-user mode. It is not installed on the host. The store, profiles, and desktop configuration persist across ERS restarts and ordinary snap refreshes. The bootstrap inside the snap must still be able to recover from a broken Nix generation.
- Initial Nix configuration is `experimental-features = nix-command flakes`, an empty `build-users-group`, and `sandbox = false`. Local builds are not sandboxed by Nix. Snap confinement still applies.
- One host Unix user is enough. Host accounts and SSH stay Ubuntu's job. Planned entry points are `snap run ers.bash` and `snap run ers.desktop`.
- Hyprland and Aquamarine come from Nix and run as the primary compositor, directly on the graphics and input devices. Ubuntu Frame is the packaging and session reference, not the runtime shell. Mir is not required.
- Software inside ERS shares one execution context. A Nix-installed program may break the ERS environment. It must not escape the snap or modify host-protected resources.

## Milestones

| Milestone | Result |
| --- | --- |
| M0 | Ubuntu Frame baseline: pinned upstream revision, build and interface notes, ERS packaging plan |
| M1 | Strictly confined snap runs upstream Nix, installs a package into persistent storage, and executes it |
| M2 | A local Nix derivation compiles and runs inside the snap |
| M3 | Nix-managed Hyprland runs as the Wayland compositor under strict confinement |
| M4 | A small declarative desktop: Hyprland, a terminal, a shell, a launcher, and one Wayland client |
| M5 | Persistence, generation rollback, snap refresh, and recovery from a broken desktop generation |

## Current state

The bootstrap is a strictly confined snap on the core26 base. `ers.bash` is a shell shipped in the snap. It starts without a Nix userspace generation. The install hook creates the snap's common data directory. snapd creates that directory as root mode 0755, and the hook cannot change that mode, so it makes `$SNAP_COMMON/data` writable by the user who runs ERS. There is no bundled Nix and no Hyprland session.

Which packaging, interfaces, graphics setup, and launch scripts from the pinned Ubuntu Frame reference ers keeps is recorded in `docs/frame-packaging.md`.

Ubuntu Frame is pinned in this repository so upstream security and feature changes can be reviewed without copying them into the ERS tree:

| | |
| --- | --- |
| Path | `reference/ubuntu-frame` |
| Upstream | https://github.com/canonical/ubuntu-frame |
| Branch | `core26` |
| Commit | `322a8fb67789b9fb831a5ab2cb486d3d5cba183f` (29 October 2026) |

The pin is staged and is not in a commit yet. History so far is the initial README commit.

The host is delia, Ubuntu Server 26.04.1, reached as `warden@10.10.82.165` with `ssh -i /home/sematula/.ssh/warden_rsa` from WSL. The account is warden. `iac-delia` provisions the host. This repository's job is ERS.
