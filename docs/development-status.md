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

Not started. No image has been imported and no instance has been booted for this pivot.

Next action: import an unprivileged NixOS LXC image into the upstream LXD snap and boot it. Confirm systemd, Nix, an internal user, and a disk that survives restart. Leave ers unchanged while that proof runs.

## M3 — first client

Not started. The Wayland socket path is undecided. Try a bind that preserves file descriptors before a proxy or a relay.

## Later milestones

M4 through M6 have no results.
