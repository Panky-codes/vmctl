# vmctl

:wrench: QEMU NVMe Testing Galore!

`vmctl` is a tool to rapidly getting preconfigured QEMU virtual machines up and
running.


## Getting Started

1. Clone the `vmctl` repository.

2. Make sure that that `ssh` and `socat` are available. Follow the standard
   procedure for your distribution to install those packages.

3. For ease of use, do a symlink in your path, say

       $ ln -s /path/to/vmctl/vmctl $HOME/bin/vmctl

4. Create a directory to hold your VMs and their configurations

       $ mkdir $HOME/vms; cd $HOME/vms

5. You probably want to use the `q35-base.conf` configuration file to base your
   own VMs on.

       $ ln -s /path/to/vmctl/examples/vm/q35-base.conf q35-base.conf

6. Start from an example and edit it as you see fit.

       $ cp /path/to/vmctl/examples/vm/nvme.conf .

7. Prepare a boot image. The `q35-base.conf` configuration will look a base
   image in `img/base.qcow2`. You can use [archbase][archbase] to build a lean
   Arch Linux base image or grab an [Ubuntu cloud image][ubuntu-cloud-image] if
   that's your vice.

   **Note** The example `nvme.conf` will define `GUEST_BOOT="img/nvme.qcow2"`.
   You do not need to provide that image - if it is not there `$GUEST_BOOT`
   will be a shallow image derived from `img/base.qcow2` by the `q35-base.conf`
   configuration. So, if you ever need to reset to the "base" state, just
   remove the `img/nvme.qcow2` image.

[archbase]: https://github.com/OpenMPDK/archbase
[ubuntu-cloud-image]: https://cloud-images.ubuntu.com


## Virtual Machine Configurations

In essence, a virtual machine configuration must provide the `QEMU_PARAMS`
array and do any required initialization of VM images. Typically,
`q35-base.conf` in combination with the `qemu_` helpers will "just work".


## Running Virtual Machines

To launch a VM, use `vmctl -c CONFIG run`. This will launch the VM specified in
the `CONFIG` config file in interactive mode such that the VM serial output is
sent to standard out. The QEMU monitor is multiplexed to standard out, so you
can access it by issuing `Ctrl-a c`.

### Tracing

The `--trace` (short: `-t`) option can be used to enable tracing inside QEMU.
The trace events will be sent to the `log/${VMNAME}/qemu.log` file (along with
any other messages written to standard error by the QEMU process). For example,
to enable all trace events for the NVMe device, but disabling anything related
to IRQs, use

    vmctl run -t 'pci_nvme,-pci_nvme_irq'

`vmctl` inserts an implicit `*`-suffix such that all traces with the given
prefix is traced.

### Custom kernel

Finally, the `--kernel-dir` (short: `-k`) can be used to point to a custom
Linux kernel to boot directly. This directory will be made available to the VM
as a p9 virtual file system with mount tag `kernel_dir`. If supported by the VM
being booted, this allows it to use kernel modules from that directory. The
image built by `archbase` has support for this built-in, but see
`contrib/systemd` for a systemd service that should be usable on most
distributions.

## License

`vmctl` is licensed under the GNU General Public License v3.0 or later.
