# Development status

Branch `pivot/nixos-lxd`. The pre-pivot tree is `8b9dfaf` on `legacy/in-snap-nix`. Neither branch is pushed.

## M0 — inventory

Done in this branch.

- Working: strict `ers` 0.1.0 on delia, `ers.bash` from the snap, no host Nix, Frame packaging notes, Frame pin.
- Not working: Hyprland, runtime Nix, a writable Nix store inside the snap.
- UID failure: the snap cannot `setuid` to a build-user pool, and the install hook cannot chown `/var/snap/ers/common`. Recorded in `docs/nix-internal-user.md`.
- Reuse plan: `docs/migration-from-snap-nix.md`.
- Architecture: `docs/architecture.md` and ADR 0009.

## M1 — compositor

Not started. Nothing in the snap is Hyprland. Do not treat the bootstrap shell as a compositor result.

## M2 — NixOS container

Issue #13. A container is running on the WSL LXD, not on the host.

LXD 6.10.1 does not present `nixos/26.05` from either image remote. The simplestreams product `nixos:26.05:amd64:default` is published, and its metadata file type is `incus.tar.xz`. Importing that metadata tarball and `rootfs.squashfs` by hand worked. Build `20261117_01:01`. Fingerprint `301dcbd4f19c81b3ea3d7340553368bda00c68b0f107e790ffd31d08c0b714c5`. Alias `nixos-26.05`.

Instance `ers-nixos`: `security.privileged=false`, `security.nesting=true`. NixOS 26.05 (Yarara), Nix 2.34.9, systemd `running`. `/root/write-probe` was still there after `lxc restart`.

The container's `/etc/nixos/configuration.nix` imports `virtualisation/lxc-container.nix` and `incus.nix`, `stateVersion` 26.05. That is the image to keep.

The Proxmox wiki's `nixos.proxmoxLXC` tarball is a different artifact. Its sample container is privileged (`--unprivileged=0`, `proxmoxLXC.privileged = true`), turns Nix's sandbox off, and the Plasma section binds the host display and input into the container. None of that is this container.

Still open on #13: an internal user with sudo, and the same boot on the host's LXD.

## M3 — first client

Not started. The Wayland socket path is undecided. Try a bind that preserves file descriptors before a proxy or a relay.

## Later milestones

M4 through M6 have no results.
