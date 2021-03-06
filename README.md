# SDSC "mlnx-ofed" roll

## Overview

This roll bundles the Mellanox&reg; OFED Linux distribution for installation on a
Rocks&reg; cluster.

> #### IMPORTANT NOTE
>
> **The Mellanox OFED Linux software must be obtained from Mellanox&reg;
> directly as this roll only wraps the software into a Rocks&reg; roll for
> installation into a Rocks&reg; cluster.**

For more information about the various packages included in the Mellanox OFED
Linux software stack and to download the MLNX_OFED_LINUX software archive
for your system please visit their official web pages:

- [Mellanox OFED Linux (MLNX_OFED)][mlnx_ofed_linux]

[mlnx_ofed_linux]: http://www.mellanox.com/page/products_dyn?product_family=26&mtag=linux_sw_drivers


## Requirements

To build/install this roll you must have root access to a Rocks&reg; development
machine (e.g., a frontend or development appliance).

If your Rocks development machine does **not** have Internet access you must
download the appropriate MLNX_OFED_LINUX source file(s) using a machine that does
have Internet access and copy them into the `src/mlnx-ofed-linux` directory on your
Rocks development machine.


## Dependencies

This roll wraps up the basic Mellanox OFED Linux installation process into a
Rocks&reg; roll which can be used to install all nodes in a cluster. Part of this
process includes building kernel modules for the running kernel and adding the
produced RPMs to the roll. Otherwise, binary RPMs provided by Mellanox are
added unchanged to the roll.

To build updated kernel modules the following packages are required on the
host building the roll...

- perl
- pciutils
- python
- gcc-gfortran
- libxml2-python
- tcsh
- libnl.i686
- libnl
- expat
- glib2
- tcl
- libstdc++
- bc
- tk
- gtk2
- atk
- cairo
- numactl
- pkgconfig
- ethtool

These RPMs are generally included on Rocks&reg; frontend and development servers.

*NOTE: The roll build process will **UNINSTALL** all Mellanox OFED Linux RPMs
from the build server during the roll build. **DO NOT** build this roll on a
frontend which is acting as your subnet manager.*


## Alternate Versions of MLNX_OFED_LINUX

This roll is used to build the Mellanox OFED Linux stack that is **in production**
on SDSC HPC systems. As such, specific versions of [MLNX_OFED_LINUX][mlnx_ofed_linux]
sources that support our installed hardware and target OS are used in the build
process.

If your hardware or OS require a different version of [MLNX_OFED_LINUX][mlnx_ofed_linux]
it is straightforward to change the roll to build that version.

Follow these steps...

- Identify the version of [MLNX_OFED_LINUX][mlnx_ofed_linux] you would like to
build/install on your system
- Download the `tgz` version of the [MLNX_OFED_LINUX][mlnx_ofed_linux] software
stack and place in the `src/mlnx-ofed-linux` directory
- Generate an updated entry for the `binary_hashes` file in the `src/mlnx-ofed-linux`
directory using [this script][gen_hash].

[gen_hash]: https://raw.githubusercontent.com/sdsc/skeleton-roll/master/gen_hash.sh

For example...

```shell
% cd src/mlnx-ofed-linux
% curl -LO https://raw.githubusercontent.com/sdsc/skeleton-roll/master/gen_hash.sh
% sh ./gen_hash.sh MLNX_OFED_LINUX-*.tgz | tee -a binary_hashes
    252193073  84799d79649f745b8ecb7e1031ab3f42602f626b  MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-x86_64.tgz
```

- Modify the `version.mk` file in the `src/mlnx-ofed-linux` directory as/if
necessary
- Modify the `.mlnx` file in the `src/mlnx-ofed-linux` directory as/if
necessary
- Modify the `version.mk` file at the root of the repository as/if
necessary. The utility scripts, `kern.sh` and `mlnx.sh` can be used to
obtain the correct values to use for the `ROLLNAME` makefile variable.


## Conflicts with OS Provided Infiniband RPMs

It is common to update a Rocks system using the `mlnx-ofed-roll` to a newer
version of `CentOS` and/or apply system updates using an `Updates-CentOS` roll.

In either of these cases it is possible to create a conflict between distro
provided Infiniband RPMs and those included by the `MLNX_OFED_LINUX` version
provided by a specific build of this roll source.

Any other roll providing Infiniband related RPMs can create a similar conflict.

An example of this conflict and it's resolution can be found in the `CONFLICTS.md`
file in this repository.


## Building

To build the mlnx-ofed-roll, execute these instructions on a Rocks development
machine (e.g., a frontend or development appliance):

```shell
% make default 2>&1 | tee build.log
% grep "RPM build error" build.log
```

If nothing is returned from the grep command then the roll should have been
created as... `mlnx-ofed-*.iso`. If you built the roll on a Rocks frontend then
proceed to the installation step. If you built the roll on a Rocks development
appliance you need to copy the roll to your Rocks frontend before continuing
with installation.


## Installation

To install, execute these instructions on a Rocks frontend:

```shell
% rocks add roll mlnx-ofed-$(ROLLNAME)*.iso
% rocks enable roll mlnx-ofed-$(ROLLNAME)
% cd /export/rocks/install
% rocks create distro
% rocks run roll mlnx-ofed-$(ROLLNAME) | bash
```
...where $(ROLLNAME) may be unique for your build and depends on the running
kernel of your build host and the version of MLNX_OFED_LINUX that you built.

On the most recent version of OS/MLNX_OFED_LINUX deployed to Comet this would
be...

```shell
% rocks add roll mlnx-ofed-4.6-1.0.1.1-3.10.0-957.27.2.el7*.iso
% rocks enable roll mlnx-ofed-4.6-1.0.1.1-3.10.0-957.27.2.el7
% cd /export/rocks/install
% rocks create distro
% rocks run roll mlnx-ofed-4.6-1.0.1.1-3.10.0-957.27.2.el7 | bash
```


## Configuration

The mlnx-ofed roll will likely need configuration for your system in order
to work properly. Specifically, the creation and/or modification of the files...

    /etc/infiniband/openibd.conf
    /etc/infiniband/connectx.conf
    /etc/modprobe.d/mlx4_core.conf

...will be unique for each system.
