# Nix store in SNAP_COMMON

The Nix store and profiles live in the snap's common data (`/var/snap/ers/common`). An install hook creates that directory and makes it writable by the user who runs ERS. Refresh leaves it in place. Removing the snap deletes it.

The per-revision data directory was rejected because a refresh would copy the store. The host home was rejected because the store is not the user's files, and a snap layout cannot bind a per-user directory to the store path.

`snap revert` does not restore this directory. A broken generation is rolled back inside the store.
