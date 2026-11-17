# Strict confinement

ERS is a snap with confinement `strict` on a trusted Ubuntu host. AppArmor, seccomp, mount namespaces, and the declared snap interfaces are the security boundary. ERS and everything running inside it are not trusted.

Classic confinement, devmode, and installing Nix on the host were rejected. Each one would let ERS or its software bypass that boundary.
