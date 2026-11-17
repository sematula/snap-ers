# Architecture

Ubuntu Core runs the machine. Snapd confines ERS. ERS presents the desktop. LXD isolates the userspace. NixOS manages the software inside that container.

```text
Ubuntu Server 26.04 / Ubuntu Core 26
│
├── snapd
│   ├── ers (strict)
│   │   ├── Hyprland + Aquamarine
│   │   ├── Wayland session
│   │   └── launcher into the container
│   ├── lxd (upstream snap)
│   │   └── unprivileged NixOS system container
│   └── microk8s (later)
└── host SSH
```

ers and the NixOS container have independent lifecycles. Hyprland is built with Nix when the snap is assembled and shipped in the snap. The container owns Nix, systemd, accounts, home directories, and applications. ERS does not reproduce those.

The container is unprivileged. Its root is not host root. The LXD management socket is not exposed to the container or to ERS. For this stage, the host administrator creates the container and its devices. Host administration stays on the host SSH service.

A Wayland connection lets a container client use the protocols Hyprland exposes. It does not grant the client the snap's AppArmor profile, and it does not grant the container the LXD API.

## Milestones

| Milestone | Issue | Result |
| --- | --- | --- |
| M0 | | Existing work preserved, UID blocker recorded, this architecture written down |
| M1 | #12 | Strictly confined ers runs Hyprland with no container required |
| M2 | #13 | Unprivileged NixOS container boots, runs Nix, and keeps its disk across restart |
| M3 | #14 | A Wayland client inside the container appears on the Hyprland display |
| M4 | #15 | A hardware-accelerated client renders without the container owning the display |
| M5 | #16 | A person can work in the container from the Hyprland session |
| M6 | #17 | Each layer restarts and recovers without taking the others with it |

M3 is the feasibility gate for the pivot. A built snap, a booted container, a visible window, and an accelerated window are four separate results.

## Wayland boundary

Preferred path, in order:

1. A bind of the Wayland socket into the container that keeps ancillary file descriptors and shared memory.
2. An LXD unix-socket proxy, only if it carries those file descriptors.
3. An existing Wayland relay. A custom transport is not part of this stage.

GPU render nodes may be passed into the container. Display-control devices stay with Hyprland unless a later test shows a client cannot render without them. Software rendering is an intermediate step, not the end of M4.

Session bus, portals, PipeWire, clipboard, and notifications wait until a client is on screen.

## Deferred

Container broker, multi-user login, MicroK8s, remote desktop, `hsudo`, and an ERS package manager. The stage in front of us is one compositor, one container, and one graphical connection.

## Decisions

ADR 0009 records the pivot. ADR 0001, ADR 0003, ADR 0005, and ADR 0006 still apply. ADR 0007 stays proposed.
