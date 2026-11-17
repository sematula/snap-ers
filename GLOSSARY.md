# Edge Runner Shell

ERS is a strictly confined snap on a Ubuntu host. This repository exists to build that product.

## Language

**Edge Runner Shell (ERS)**:
The product: a strictly confined Ubuntu snap that integrates a Nix-managed userspace with a Hyprland desktop.
_Avoid_: snap-ers, ers

**ers**:
The snap that delivers ERS.
_Avoid_: snap-ers, ERS

**Host**:
The Ubuntu system that runs ERS. The host is trusted. ERS and everything inside it are not.
_Avoid_: warden, machine

**delia**:
The host for this effort. An Ubuntu Server 26.04 machine, provisioned by the iac-delia repository.
_Avoid_: warden

**warden**:
The Unix account on delia that builds and runs ERS.
_Avoid_: delia, the host

**snap-ers**:
The git repository that contains ERS.
_Avoid_: ERS, ers, Edge Runner Shell

**Bootstrap**:
The immutable part of the ers snap.
_Avoid_: userspace

**Userspace**:
The persistent, Nix-managed part of ERS.
_Avoid_: bootstrap, the host

**Host home**:
The Unix account's home directory on the host. ERS may read and write its non-hidden files.
_Avoid_: snap home, $HOME

**Snap home**:
The home directory inside ERS. The dotfiles tool writes here.
_Avoid_: host home
