---
status: accepted
---

# NixOS container, ERS compositor

Strict confinement cannot give Nix a build-user pool or a store owner that matches upstream multi-user Nix. ERS stays a strictly confined snap and runs Hyprland. The userspace is an unprivileged NixOS system container on the upstream LXD snap. The snap and the container have independent lifecycles. A runtime Nix installation inside ers is retired.

Hyprland stays upstream. It is built with Nix when the snap is assembled and shipped inside the immutable snap, so the compositor runs while the container is stopped.

## Considered options

Adapting Nix to the snap failed on identity. A passwd line inside the snap is not a kernel uid. The install hook cannot chown `/var/snap/ers/common`. A snap service may `setuid` to `_daemon_` (uid 584792) and not to a `nixbld` pool, so host build users stay unused. The findings are in `docs/nix-internal-user.md`.

Classic confinement was rejected. ADR 0001 stands.

A root snap daemon that drops to `_daemon_` is one uid, and that uid does not own the common directory.

## Consequences

The integration boundary is the Wayland socket between Hyprland in ers and clients in the container. The container does not receive the LXD management socket. Container root is not host root. Host SSH stays the host administration path.

ADR 0002, ADR 0004, and the Nix-generation recovery role in ADR 0008 are superseded. ADR 0003 stands: Hyprland is the compositor, built at snap assembly rather than installed into a store inside ers. ADR 0005 stands: the compositor's graphics userspace is the `mesa-2604` content snap. ADR 0006 stands: the host home is the person's ordinary files. ADR 0007 stays proposed.
