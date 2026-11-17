---
status: superseded by ADR-0009
---

# Daemonless Nix inside the snap

ERS runs upstream Nix inside the snap, daemonless and single-user, with `nix-command` and flakes enabled and `sandbox = false`. Snap confinement is the sandbox. Nix is not installed on the host, and the multi-user daemon is not running.

Multi-user Nix and Nix's own sandbox are deferred to a later iteration. This is the initial configuration, not a ban on either.
