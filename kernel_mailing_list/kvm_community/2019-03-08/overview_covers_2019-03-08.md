

#### [RFC net-next v1 0/3] Support mlx5 mediated devices in host
##### From: Parav Pandit <parav@mellanox.com>

```c

Use case:
---------
A user wants to create/delete hardware linked sub devices without
using SR-IOV.
These devices for a pci device can be netdev (optional rdma device)
or other devices. Such sub devices share some of the PCI device
resources and also have their own dedicated resources.
A user wants to use this device in a host where PF PCI device exist.
(not in a guest VM.) A user may want to use such sub device in future
in guest VM.

Few examples are:
1. netdev having its own txq(s), rq(s) and/or hw offload parameters.
2. netdev with switchdev mode using netdev representor
3. rdma device with IB link layer and IPoIB netdev
4. rdma/RoCE device and a netdev
5. rdma device with multiple ports

Requirements for above use cases:
--------------------------------
1. We need a generic user interface & core APIs to create sub devices
from a parent pci device but should be generic enough for other parent
devices
2. Interface should be vendor agnostic
3. User should be able to set device params at creation time
4. In future if needed, tool should be able to create passthrough
device to map to a virtual machine
5. A device can have multiple ports
6. An orchestration software wants to know how many such sub devices
can be created from a parent device so that it can manage them in global
cluster resources.

So how is it done?
------------------
Kernel has existing mediated device infrastructure for lifecyle of such
sub devices provided by mdev driver.
Hence, these sub devices are created with help of mdev driver.

mlx5_core driver registers with mdev core to do so and exposes necessary
sysfs files.

Each creates sub device has unique uuid id assigned by the user.

mdev sub devices inherit their parent's dma parameters.

Each registered mdev has corresponding devlink instance. Through this
devlink instance, such device and it port(s) are managed.

In order to use mediated device in a VM or in host, user decides
which driver to use. Typically vfio_mdev is used to expose a mdev in a
guest VM. In current use case, mlx5 mediated devices are only usable
inside the host through mlx5_core driver binding to it.

Patchset summary:
-----------------
Patch-1 adds support to inherit dma params of parent device in child mdev.
Patch-2 registers with mdev core.
Patch-3 registers a mdev device driver to create actual netdev.

Summary of alternatives considered and discussed:
-------------------------------------------------
1. new subdev bus
   Fits the need but mdev simplifies it.
2. visorbus
   Very specific to Unisys s-Par devices.
3. platform devices
   Primarily meant for autonomous, SoC etc devices.
4. mfd devices
   Depends on platform device infra.
5. Directly creating netdev, rdma device instead of sub device
   Doesn't fit use case of passthrough mode.
6. creating subports of devlink instance
   Doesn't cover multiport rdma device usecase.

While discussion [1], [2] is still ongoing, v1 is posted to describe
how two use cases of using mdev in host or in guest via standard Linux
device driver model are addressed.

[1] https://www.spinics.net/lists/netdev/msg556552.html
[2] https://www.spinics.net/lists/netdev/msg556944.html

All patches are only a reference implementation to see framework in
works at devlink, sysfs, mdev and device model level. Once RFC looks good,
solid upstreamable version of the implementation will be done.

System view with one mdev:
--------------------------

$ ls -l /sys/bus/pci/devices/0000:05:00.0
[..]
drwxr-xr-x 3 root root        0 Mar  8 14:53 69ea1551-d054-46e9-974d-8edae8f0aefe
drwxr-xr-x 3 root root        0 Mar  8 15:41 infiniband
drwxr-xr-x 3 root root        0 Mar  8 15:41 mdev_supported_types
-rw-r--r-- 1 root root     4096 Mar  8 13:17 msi_bus
drwxr-xr-x 2 root root        0 Mar  8 15:41 msi_irqs
drwxr-xr-x 3 root root        0 Mar  8 15:41 net

ls -l /sys/bus/mdev/drivers
total 0
drwxr-xr-x 2 root root 0 Mar  8 13:39 mlx5_core
drwxr-xr-x 2 root root 0 Mar  8 14:53 vfio_mdev

ls -l /sys/bus/mdev/devices/
total 0
lrwxrwxrwx 1 root root 0 Mar  8 14:53 69ea1551-d054-46e9-974d-8edae8f0aefe -> ../../../devices/pci0000:00/0000:00:02.2/0000:05:00.0/69ea1551-d054-46e9-974d-8edae8f0aefe

Bind mdev to mlx5_core driver:
$ echo 69ea1551-d054-46e9-974d-8edae8f0aefe > /sys/bus/mdev/drivers/mlx5_core/bind

$ ls -l /sys/class/net/eth0/
-r--r--r-- 1 root root 4096 Mar  8 15:43 carrier_up_count
lrwxrwxrwx 1 root root    0 Mar  8 15:43 device -> ../../../69ea1551-d054-46e9-974d-8edae8f0aefe
-r--r--r-- 1 root root 4096 Mar  8 15:43 dev_id

$ devlink dev show
pci/0000:05:00.0
mdev/69ea1551-d054-46e9-974d-8edae8f0aefe

Changelog
---
v0->v1:
 - Removed subdev bus, instead using existing mdev bus which fits
   the need.
 - Dropped devlink patches which are not needed anymore due to use of
   mdev framework.
 - Updated SPDX license line in patches.
 - Added TODO to patches where more hardware specific code will be added.

Parav Pandit (3):
  vfio/mdev: Inherit dma masks of parent device
  net/mlx5: Add mdev sub device life cycle command support
  net/mlx5: Add mdev driver to bind to mdev devices

 drivers/net/ethernet/mellanox/mlx5/core/Kconfig    |   9 ++
 drivers/net/ethernet/mellanox/mlx5/core/Makefile   |   5 +
 drivers/net/ethernet/mellanox/mlx5/core/dev.c      |  18 ++++
 drivers/net/ethernet/mellanox/mlx5/core/main.c     |  22 ++++
 drivers/net/ethernet/mellanox/mlx5/core/mdev.c     | 120 +++++++++++++++++++++
 .../net/ethernet/mellanox/mlx5/core/mdev_driver.c  | 106 ++++++++++++++++++
 .../net/ethernet/mellanox/mlx5/core/mlx5_core.h    |  19 ++++
 drivers/vfio/mdev/mdev_core.c                      |   4 +
 include/linux/mlx5/driver.h                        |   5 +
 9 files changed, 308 insertions(+)
 create mode 100644 drivers/net/ethernet/mellanox/mlx5/core/mdev.c
 create mode 100644 drivers/net/ethernet/mellanox/mlx5/core/mdev_driver.c
```
