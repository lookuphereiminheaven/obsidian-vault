###### BIOS
- Limited to one sector of the disk and needs a multi-stage bootloader
- Can start the bootloader from internal/external HDD, CD/DVD, USB Flash drive, Network server
- If booting from the HDD, the Master Boot Record will be used (1 sector)
###### UEFI
- Specifies a special disk partition for the bootloader. Called EFI System Partition (ESP)
- ESP is FAT and mounted on /boot/efi and bootloader files has .efi extensions
- /sys/firmware/efi 
###### Bootloader
- initializes the minimum hardware needed to boot the system. then, it locates and runs the OS
- GRUB can be used to run any specific program you need but it generally runs the OS
###### Kernel
- core of your operating system
- Your bootloader loads the kernel in the memory and runs it
- Needs info to run stored in `initrd` or `initramfs`
- You can also send parameters to the kernel during the boot using the Grub configs. For example, sending a 1 or S will result the system booting in single-user mode (recovery). Or you can force your graphics to work in 1024×768x24 mode by passing `vga=792` to the Kernel during the boot
###### dmesg
-  kernel saves it's own logs into the "Kernel Ring Buffer". after the compilation of the boot process, the syslog daemon collects the boot logs and stores them in `/var/log/dmesg`
- `dmesg` to show logs
- `journalctl -k` to check kernel logs
- `journalctl -b` to check boot logs
- `journalctl -u kernel` to show previous logs too
- systems keep logs at `/var/log/boot` or  `/var/log/boot.log` in Debian
`/var/log/messages`
- After `init` comes up `syslog daemon` will log messages with timestamps
- might be in `/var/log/syslog`
###### init
- SysVinit : based on Unix and old
- upstart : outdated
- Systemd : starts services in parallel
- `pstree` check hierarchy of processes
###### Systemd
- made around units
- 12 types of units :  automount, device, mount, path, scope, service, slice, snapshot, socket, swap, target & timer.
- use `systemctl` to work with units
- `journalctl` to see logs
- units are found in
  - `/etc/systemd/system/`
  - `/run/systemd/system/`
  - `/usr/lib/systemd/system/`
  - We can use these commands to work with services :
```
# systemctl stop sshd
# systemctl start sshd
# systemctl status sshd
# systemctl is-active sshd
# systemctl is-failed sshd
# systemctl restart sshd
# systemctl reload sshd
# systemctl daemon-reload sshd
# systemctl enable sshd
# systemctl disable sshd
```
