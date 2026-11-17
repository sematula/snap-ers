# Mesa from the content snap

Hyprland comes from Nix. The graphics userspace is the `mesa-2604` content snap, connected through the `gpu-2604` interface, the same plug Ubuntu Frame uses. ERS owns the wrapper that points Hyprland at that stack.

Shipping Mesa from Nix was rejected. A second driver stack inside the snap would fight the graphics confinement Ubuntu already uses for this base.
