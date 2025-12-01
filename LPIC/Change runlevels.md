predefined states that determine which system services are operational
Systemd
- groups of services
- possible to isolate any target
- `rescue` Local file systems are mounted, no networking, and root-user only
- `emergency` Only the root file system and in read-only mode, No networking and root-user only
- `halt` stops all processes and halts CPU activities
- `poweroff` like halt but also sends ACPI shutdown signal
- Configs in
  - `/etc/systemd/`
  - `/usr/lib/systemd/`
SysV
- Debian 
  - 0-Shutdown
  - 1-Single-user mode
  - 2-Multi-user mode with graphics
  - 6-Reboot
