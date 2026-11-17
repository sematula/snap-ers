---
status: superseded by ADR-0009
---

# Bootstrap shell

ADR 0009 retires the Nix-generation recovery role. `ers.bash` remains a shell shipped in the snap.

`ers.bash` is shipped in the snap, not installed by Nix. It still starts when the current generation is broken, and from that shell the generation can be rolled back. `ers.desktop` starts Hyprland and does not repair a broken generation.

The userspace shell was the obvious meaning of `ers.bash`. It was rejected because a broken generation would take the recovery path with it.
