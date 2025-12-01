Sysfs
  - Sysfs is a pseudo file system that exports information about various kernel subsystems, hardware devices, and associated device drivers from the kernel's device model to user space through virtual files.
  - Sysfs is mounted under the `/sys mount point`.
udev
- udev (userspace `/dev`) is a device manager that primarily manages device nodes in the `/dev` directory.
- udev also handles all user space events raised when hardware devices are added into the system or removed including firmware loading as required by certain devices.
dbus
- D-Bus is a message bus system
- In addition to inter-process communication, D-Bus helps coordinate process lifecycle; It makes it simple and reliable to code a "single instance" application or daemon and to launch applications and daemons on demand when their services are needed.
proc directory
- This is where the Kernel keeps its settings and properties
-  It's created on ram and files might have write access
- IRQs (interrupt requests)
- I/O ports (locations in memory where CPU can talk with devices)
- DMA (direct memory access, faster than I/O ports)
- Processes
- Network Settings
- `/proc/sys/net/ipv4`which controls real-time networking configurations
- All changes will be reverted after a boot. You have to write into config files in `/etc/` to make these changes permanent
`lspci`
- Shows PCI devices connected
`lsusb`
- Shows all the USB devices connected
`lshw`
- shows hardware
`lsblk`
- To list devices that can read from or write to blocks of data
Loadable Kernel Modules
- `lsmod` for inspecting modules at `/lib/modules`
- `modprobe` for manage
- `rmmod` for removing
- `insmod` for installation
- Loadable kernel modules (.ko files) are object files that are used to extend the kernel of the Linux Distribution. They are used to provide drivers for new hardware like IoT expansion cards that have not been included in the Linux Distribution
