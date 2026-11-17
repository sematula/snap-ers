# Snap-internal Nix user

Upstream Nix has no mode that is one dedicated non-root Unix user, living only inside the ers snap, which Nix then runs as. Adding such a user would leave the kernel uid of the Nix process unchanged, so it would not make `/var/snap/ers/common` writable. Do not implement it.

The Nix store and profiles are the userspace of Edge Runner Shell (ERS). They stay in the common data directory of the ers snap. Under `snap run ers.bash` the process that can write that directory is the invoking host uid. On this demonstration that account is warden, and only once the directory grants write to that uid.

## Given constraints

These are inputs to the judgement, settled with the design of this demonstration.

snapd creates `/var/snap/ers/common` as root:root mode 0755. The install hook runs as root. Its AppArmor profile denies chmod and chown of that directory inode. A POSIX access ACL `setxattr` on that same inode, under the same profile, changed the mode to 0777, and the invoking user could then create a file there.

`snap run ers.bash` runs as the invoking host user. Strict confinement of ers uses the host user namespace. Nix stays daemonless and single-user. The multi-user daemon and Nix's own sandbox are deferred. Nix is not installed on the host. Host accounts stay the host's job, and one host Unix user is enough. The parent spec says that user should not have to create accounts inside the snap.

The stable Nix manual checked here is Nix 2.34.9. The introduction and the multi-user chapter at <https://nixos.org/manual/nix/stable/> match the same pages at <https://nix.dev/manual/nix/stable/>. Those pages do not differ.

## Modes Nix documents

### Single-user

Single-user installation is `--no-daemon`. `/nix` is owned by the invoking user. The manual's example, run as root, is `mkdir /nix` and `chown alice /nix`. The installation chapter states that installing Nix as root is not supported by this script. This is the default on Linux without systemd, and on Linux with SELinux.

<https://nix.dev/manual/nix/stable/installation/installing-binary>

<https://nix.dev/manual/nix/stable/installation/>

When `build-users-group` is empty, builds run as the uid of the Nix process: the uid of the caller when `NIX_REMOTE` is empty, or the uid of the daemon when `NIX_REMOTE` is `daemon`. The default is `nixbld` when Nix is running as root, and empty otherwise. A non-root single-user Nix therefore builds and writes the store as whoever invoked the command.

<https://nix.dev/manual/nix/2.34/command-ref/conf-file>

### Multi-user and the daemon

Multi-user installation is `--daemon`. The manual says this creates system users and a system service for the Nix daemon. The store and database are owned by some privileged user (usually `root`). An unprivileged user's store operations are forwarded to a Nix daemon running as that owner. The manual says to start the daemon as root (`nix-daemon`) and to set `NIX_REMOTE=daemon` for clients. `nix-daemon` is described as the multi-user support daemon that runs store operations on behalf of unprivileged users. The newer `nix daemon` command is described the same way, as a daemon for non-root clients, and its examples are run as root.

<https://nix.dev/manual/nix/stable/installation/installing-binary>

<https://nix.dev/manual/nix/stable/installation/multi-user>

<https://nix.dev/manual/nix/stable/command-ref/nix-daemon>

<https://nix.dev/manual/nix/2.34/command-ref/new-cli/nix3-daemon>

The second-stage multi-user script for Nix 2.34.9, which is the version the stable manual documents, creates the system group `nixbld` (default gid 30000) and `nixbld1` through `nixbldN` (default 32 users, names from `nixbld%d`, uids from 30001) with `groupadd` and `useradd`. It looks them up with `getent group` and `getent passwd`. It creates `/nix/store` as that group, mode 1775. If `/nix` already exists, it chowns that tree to `root:nixbld`. After copying store paths in, it chowns those paths to root (`0:0`) and clears their write bits. It writes `build-users-group = nixbld`. Those build users are host accounts. The script's only `useradd` is this pool. It does not create one non-root account for the daemon to run as.

<https://github.com/NixOS/nix/blob/2.34.9/scripts/install-multi-user.sh>

<https://github.com/NixOS/nix/blob/2.34.9/scripts/install-systemd-multi-user.sh>

The downloader the manual links, <https://nixos.org/nix/install>, currently unpacks Nix 2.35.6 and executes that tarball's `install`. The 2.35 multi-user chapter uses the same wording as 2.34.9: privileged owner usually root, build users `nixbld1`, `nixbld2`, and so on, daemon started as root.

<https://nix.dev/manual/nix/2.35/installation/multi-user>

### Build users

The build users are a pool of special uids, usually named `nixbld1`, `nixbld2`, and so on, all in the group `nixbld` and in no other group. The manual's Linux example is `groupadd -r nixbld` and a `useradd` loop. Nix runs at most one build under a given build user at a time. There cannot be more concurrent builds than there are build users.

`build-users-group` is the group of those accounts. The manual says builds should not be performed by the Nix account, because a caller could then modify the store and database through a crafted builder, and should not be performed by the calling user, because that user could influence the build result. If the group is non-empty, builds run as members of the group, listed in `/etc/group`. Those accounts should not be used for any other purpose. The store should be owned by the Nix account, grouped as that build group, mode `1775`, so build users can create files and cannot delete them. The phrase "the Nix account" is not given a username on that page. The multi-user chapter is the page that names the privileged owner, and it says usually `root`.

`allowed-users` and `trusted-users` list who may connect to the daemon and who is trusted by it. They do not choose the uid of the Nix process.

<https://nix.dev/manual/nix/stable/installation/multi-user>

<https://nix.dev/manual/nix/2.34/command-ref/conf-file>

### Settings that allocate more build uids

The `nix.conf` reference for 2.34.9 has no setting that names one non-root Unix user as the identity Nix runs as. The settings that deal with uids are the build pool and two experimental ways to size that pool.

`auto-allocate-uids` (experimental, default false) selects uids for builds in place of the users in `build-users-group`. On Linux those uids start at 872415232 (`0x34000000`). `start-id` is that first id. `id-count` is how many to use, default 8388608. The `uid-range` system feature, included on Linux when `auto-allocate-uids` is enabled, lets a build run in a user namespace where it is uid 0 and has 65536 uids. That is a range of build uids. It belongs to Nix's sandbox, which this demonstration defers.

<https://nix.dev/manual/nix/2.34/command-ref/conf-file>

Release 2.34 adds `libexec/nix-nswrapper`, which runs the Nix daemon in an unprivileged user namespace on Linux. Build-user uids and gids must be allocated in `/etc/subuid` and `/etc/subgid`. The release note says this can run the daemon with full sandboxing without executing as root, and that Nixpkgs gained `nix.daemonUser` and `nix.daemonGroup` for it. The Nix manual does not define those Nixpkgs settings. The note still requires a build-user pool in host files, and it is a daemon. Both are outside this demonstration.

<https://nix.dev/manual/nix/stable/release-notes/rl-2.34>

The same release notes point at a beta Rust installer, <https://artifacts.nixos.org/nix-installer>, source <https://github.com/NixOS/nix-installer>, described as a modified Determinate Nix Installer. Those notes do not describe a singular non-root Nix user. The installation chapter still documents two flags, `--daemon` and `--no-daemon`. Determinate Systems' own documentation was not used.

## A snap-local passwd entry

A line in a passwd file inside the snap filesystem does not create a uid the store writer can run as.

`/etc/passwd` is a text file that describes login accounts. Each line holds a name and a numeric UID. Utilities such as `ls` use it to map ids to names. The file is not the process credential.

<https://man7.org/linux/man-pages/man5/passwd.5.html>

Each process has integer user ids (`uid_t`): real, effective, saved, and, on Linux, the filesystem user id. Linux uses the filesystem user id for file permission checks. A child inherits those ids from its parent. They change when the process calls `setuid` or a related call.

<https://man7.org/linux/man-pages/man7/credentials.7.html>

`setuid` changes the effective uid, and on Linux it also sets the filesystem uid. If the caller lacks `CAP_SETUID` in its user namespace, the call fails with `EPERM` unless the target uid is already the real uid or the saved set-user-ID. If the uid is not valid in that user namespace, the call fails with `EINVAL`.

<https://man7.org/linux/man-pages/man2/setuid.2.html>

A process is a member of exactly one user namespace. A namespace created with `unshare` or `clone` `CLONE_NEWUSER` starts with no user-id mapping. The mapping is a range of uids written once to `/proc/<pid>/uid_map`, from that namespace onto its parent. A passwd line does not write that file. In the host user namespace, which is the namespace ers runs in, the credential is the host uid `snap run` was started with.

<https://man7.org/linux/man-pages/man7/user_namespaces.7.html>

`snap run ers.bash` therefore stays warden. Single-user Nix, with an empty `build-users-group`, writes the store as that same uid. `/var/snap/ers/common` is root:root mode 0755, so warden's filesystem uid fails the owner write check until the directory grants write to that uid. The ACL experiment is that grant: mode 0777, then the invoking user created a file. A different name in the snap's passwd file does not change the uid the permission check sees.

Nix's documented way to obtain build users is `useradd` and `groupadd` on the system, which updates the host account database. That is a host account, and this demonstration leaves host accounts to the host.

snapd does document one non-root identity a snap can drop to, and it is a different mechanism from a passwd line the snap writes itself. A `system-usernames` entry asks snapd to allocate a user. The page says `snap_daemon` is deprecated and replaced with `_daemon_`, uid 584792, and that the `snap_daemon: shared` stanza exposes uid 584788 to `getpwnam` and to `setuid`. The service still starts as root, from systemd, and then drops. That uid is allocated by snapd in the host user namespace. It is not a Nix run-as mode. `snap run ers.bash` stays the invoking user, Nix has no setting that calls `setuid` to that account, the common directory is not owned by that uid, and the install hook cannot chown the directory to it.

<https://snapcraft.io/docs/explanation/snap-development/system-usernames/>

## Judgement

Do not implement a dedicated snap-internal Nix user. It does not fix the write to `/var/snap/ers/common`.

The writer that remains is the invoking uid, warden, once that directory is writable by that uid. That is the single-user identity upstream Nix already uses. The store stays in `/var/snap/ers/common`. The daemon, the `nixbld` pool, `auto-allocate-uids`, and `nix-nswrapper` stay deferred with the rest of multi-user Nix.
