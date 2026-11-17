# snap-ers

Edge Runner Shell (ERS) is a strictly confined Ubuntu snap. It presents upstream Hyprland. The userspace is an unprivileged NixOS system container on the upstream LXD snap.

Ubuntu Core runs the machine. Snapd confines ERS. ERS presents the desktop. LXD isolates the userspace. NixOS manages the software inside that container.

ERS is intended to run first on Ubuntu Server 26.04 LTS, and later on Ubuntu Core 26. The picture, the milestones, and the boundaries are in `docs/architecture.md`. The in-snap Nix path this replaces is in `docs/migration-from-snap-nix.md`.

## Current state

`ers` is a strictly confined snap on the core26 base. `ers.bash` is a shell shipped in the snap. Hyprland is not packaged yet. An unprivileged NixOS 26.05 container named `ers-nixos` is running on the host LXD. There is no runtime Nix inside the snap.

Which packaging, interfaces, graphics setup, and launch scripts from the pinned Ubuntu Frame reference ers keeps is recorded in `docs/frame-packaging.md`.

Ubuntu Frame is pinned in this repository so upstream security and feature changes can be reviewed without copying them into the ERS tree:

| | |
| --- | --- |
| Path | `reference/ubuntu-frame` |
| Upstream | https://github.com/canonical/ubuntu-frame |
| Branch | `core26` |
| Commit | `322a8fb67789b9fb831a5ab2cb486d3d5cba183f` (29 October 2026) |

The pin is recorded on `legacy/in-snap-nix` at `8b9dfaf` and is on this branch. The in-snap Nix checkpoint is that same commit.

The host is delia, Ubuntu Server 26.04.1, reached as `warden@10.10.82.165` with `ssh -i /home/sematula/.ssh/warden_rsa` from WSL. The account is warden. `iac-delia` provisions the host. This repository's job is ERS.
