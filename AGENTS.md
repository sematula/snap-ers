## Agent skills

### Issue tracker

Issues and specs for Edge Runner Shell live as GitHub issues in sematula/snap-ers. See `docs/agents/issue-tracker.md`.

### Triage labels

Five default roles: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one root `GLOSSARY.md` and `docs/adr/`. See `docs/agents/domain.md`.

### Architecture

The userspace is an unprivileged NixOS system container on the upstream LXD snap. ers is the strict Hyprland shell. Read `docs/architecture.md` before changing the snap, Nix, LXD, or the Wayland connection. The retired in-snap Nix path is `docs/migration-from-snap-nix.md`.

### Snap build

Build ers on the WSL machine with `snapcraft pack --use-lxd` for amd64 when `lxc` can reach the LXD socket. When it cannot, use `snapcraft remote-build` on Launchpad. Copy the tree onto the WSL filesystem first when the checkout is under `/mnt`. Install the built snap on delia. Leave the build toolchain off delia.
