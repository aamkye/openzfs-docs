.. highlight:: sh

Preparation
======================

.. contents:: Table of Contents
   :local:

#. Disable Secure Boot. ZFS modules can not be loaded if Secure Boot is enabled.
#. Download a variant of `AlmaLinux Minimal Live ISO
   <https://repo.almalinux.org/almalinux/9.1/live/x86_64/>`__
   and boot from it.
#. Connect to the Internet.
#. Set root password or ``/root/.ssh/authorized_keys``.
#. Start SSH server::

    echo PermitRootLogin yes >> /etc/ssh/sshd_config
    systemctl restart sshd

#. Connect from another computer::

    ssh root@192.168.1.19

#. Target disk

   List available disks with::

    find /dev/disk/by-id/

   If using virtio as disk bus, use ``/dev/disk/by-path/``.

   Declare disk array::

    DISK='/dev/disk/by-id/ata-FOO /dev/disk/by-id/nvme-BAR'

   For single disk installation, use::

    DISK='/dev/disk/by-id/disk1'

#. Set partition size:

   Set swap size, set to 1 if you don't want swap to
   take up too much space::

    INST_PARTSIZE_SWAP=4

   Root pool size, use all remaining disk space if not set::

    INST_PARTSIZE_RPOOL=

#. Temporarily set SELinux to permissive in live environment::

    setenforce 0

   SELinux will be enabled on the installed system.

#. Add ZFS repo and install ZFS inside live system::

    dnf install -y https://zfsonlinux.org/epel/zfs-release-2-2$(rpm --eval "%{dist}").noarch.rpm
    rpm -e --nodeps zfs-fuse || true
    source /etc/os-release
    export VERSION_ID
    dnf config-manager --disable zfs
    dnf config-manager --enable zfs-kmod
    dnf install -y zfs
    modprobe zfs

#. Install partition tool and arch-install-scripts::

    dnf install -y epel-release
    dnf install -y gdisk dosfstools cryptsetup
    dnf download arch-install-scripts
    rpm -i --nodeps arch-install-scripts*.rpm
    dnf remove -y epel-release
