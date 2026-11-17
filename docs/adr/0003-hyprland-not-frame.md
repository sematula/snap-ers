# Hyprland is the compositor

Hyprland and Aquamarine come from Nix and are the Wayland compositor, on the graphics and input devices. Ubuntu Frame is the packaging and session reference. Mir is not the runtime.

Frame was the obvious shell to ship, because this repo already pins it and it is a strictly confined snap. ERS is a demonstration that a Nix-managed desktop can run inside that kind of confinement, so the compositor has to be Hyprland.
