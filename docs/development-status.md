# Development status

Branch `pivot/nixos-lxd`. The pre-pivot tree is `8b9dfaf` on `legacy/in-snap-nix`. Neither branch is pushed.

## M0 — inventory

Done in this branch.

- Working: strict `ers` 0.1.0 on delia, `ers.bash` from the snap, no host Nix, Frame packaging notes, Frame pin.
- Not working: Hyprland, runtime Nix, a writable Nix store inside the snap, any NixOS container.
- UID failure: the snap cannot `setuid` to a build-user pool, and the install hook cannot chown `/var/snap/ers/common`. Recorded in `docs/nix-internal-user.md`.
- Reuse plan: `docs/migration-from-snap-nix.md`.
- Architecture: `docs/architecture.md` and ADR 0009.

## M1 — compositor

Not started. Nothing in the snap is Hyprland. Do not treat the bootstrap shell as a compositor result.

## M2 — NixOS container

Not booted. No instance has been imported.

The WSL LXD remote `images` (`https://images.lxd.canonical.com`) has no `nixos/26.05` and no `nixos/25.05`. A remote `lxc-images` pointing at `https://images.linuxcontainers.org` was added on that same LXD. It also has no `nixos/26.05` and no `nixos/unstable`. The Linux Containers HTML index lists those builds for LXC and Incus. This LXD client is not offered them.

Next action: build an image with nixos-generators (`-f lxc` and `-f lxc-metadata`) and import that. Leave ers unchanged while that proof runs. Do not write a custom image builder.

## M3 — first client

Not started. The Wayland socket path is undecided. Try a bind that preserves file descriptors before a proxy or a relay.

## Later milestones

M4 through M6 have no results.
