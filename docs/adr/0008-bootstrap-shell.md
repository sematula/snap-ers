# Bootstrap shell

`ers.bash` is shipped in the snap, not installed by Nix. It still starts when the current generation is broken, and from that shell the generation can be rolled back. `ers.desktop` starts Hyprland and does not repair a broken generation.

The userspace shell was the obvious meaning of `ers.bash`. It was rejected because a broken generation would take the recovery path with it.
