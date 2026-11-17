# Host home is for the user's files

ERS plugs `home`. Programs inside the snap may read and write the invoking user's non-hidden host-home files. Dotfiles for ERS stay in the snap home. `personal-files` is not plugged until a specific host dotfile is named.

Which directory is the snap home does not change this boundary. That directory is provisional.

The host home was rejected as the place for the Nix store. It is the place an ERS program reads and writes the user's ordinary files.
