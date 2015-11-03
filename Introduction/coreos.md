

## # **CoreOS Introduction**

CoreOS is an Open-Source and lightweight operating system based on Linux kernel and designed for providing infrastructure for clustered deployment, while focusing on automation, ease of security, high availability and scalability. CoreOS provide minimal functionality required for deploying application inside software container such as docker or rkt together with built-in mechanism for service discovery and configuration sharing.

CoreOS provide no package manager as a way to distribute payload applications, requiring instead all applications to run inside their containers. CoreOS instance uses the underlying operating-system-level-virtualization features of the Linux kernel to create and configure multiple containers that perform isolated Linux systems. This way, resource partitioning between containers is performed through multiple isolated userspace instances, instead of using a hypervisor and providing full-flegged virtual machines. This approach relies on the linux kernel's functionality. which provides namespace isolation and abilities to limit, account and isolate resource usge (CPU, memory, disk I/O, etc. ) for the collections of processess.