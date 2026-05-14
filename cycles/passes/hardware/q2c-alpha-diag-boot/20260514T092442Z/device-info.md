# Device Info

## Hardware

- **Platform**: DriveNets Q2C — NCP-1 packet forwarder (Broadcom Qumran2C silicon)
- **Form factor**: COM Express Type 7 module — ADLINK Express-VR7-R3.1 (per BIOS
  string + ADLINK SEMA EC driver); board version `A2`
- **SoC / CPU**: AMD Ryzen Embedded V3C18I 8-Core Processor (16 threads)
- **RAM**: ~64 GB (65,646,428 kB total)
- **Storage**: 2× NVMe SSD on PCIe (enumerated as BLK0/BLK1 at the UEFI shell)
- **Packet processor**: Broadcom Qumran2C ("Q2C") — PCI `01:00.0`, device `14e4:8820`
- **Secondary Broadcom device**: PCI `02:00.0`, device `14e4:9800`
- **BIOS**: American Megatrends Inc. `1.05.11.DN03` (04/20/2026)
- **Board ID / Serial**: not programmed — `board_id_get` returns `0x00` (default),
  DMI vendor/product/serial all read "To be filled by O.E.M." (manufacturing/bring-up state)

## Operating System

> **Not the production OS.** The device is running a RAM-based **diagnostics image**,
> not DriveNets DNOS. The production target is determined by the installer USB (see below).

- **OS**: Buildroot 2025.02.12 — "QTOOC" Alpha Diagnostics image
- **Image**: `qtooc.0.0.1-b2` (SERIES_NAME `QTOOC`, RELEASE_VERSION `0.0.1-b2`)
- **Built**: 2026-05-13 12:33 by `yushin`
- **Kernel**: 6.12.40 `#2 SMP PREEMPT_DYNAMIC` x86_64 (built 2026-05-13)
- **Hostname**: `alphadiags`
- **Root filesystem**: runs entirely from RAM (`rootfs`, 32 GB) — nothing persisted to disk
- **SSH**: dropbear
- **Login**: `root` (password `drivenets`)
- **Diag tooling**: `/alpha` tree — Broadcom SDK chip DBs (jericho2/3/4, qumran2/3/4,
  ramon), `cli`/`alphach`/`mgmt-app`, Quectel modem proxies, board TOMLs.
  The image's only board profile is "DN 3200R Switch" (board-101).

## Installer USB (attached)

`/dev/sda1` (label `BASEOS_INST`, vfat) is mounted at `/alpha/mnt/usb` and carries the
DriveNets production install media:

- `drivenets_baseos_2.2540007001.tar` — DriveNets BaseOS 2.2540007001
- `onie-installer-x86_64-drivenets_ncp1_1_s9700_53dx-r0` / `onie-updater-...`
- `package-s9700-53dx-24.04-basic-1.0.0-rootfs_*.tar.xz`
- `diag-installer-x86_64-ncp1_1-r0.sh`, `startup.nsh`

→ Production platform: **UfiSpace S9700-53DX**, DriveNets **NCP-1** (Network Cloud
Packet forwarder).

## Boot Configuration

- **Boot operation**: `connect_networks` (networks: `cross-eth`)
- **Boot chain**: AMI UEFI firmware → **UEFI Interactive Shell v2.2** (EDK II) →
  `startup.nsh` on USB FS0 → launches the `qtooc.0.0.1-b2` EFI-stub kernel directly.
  **No GRUB.** The boot device list at the EFI shell shows the SanDisk USB (FS0) plus
  2× NVMe SSDs (BLK0/BLK1) — the diag kernel is booted off the USB, NVMe untouched.
- **Init**: custom `/init` (`Run /init as init process`) — **no systemd**. `/init`
  pip-installs `diagpy` + `pytest` from `/alpha/whl`, loads the ADLINK SEMA EC
  drivers, runs a device scan, and drops to a shell.
- **Kernel cmdline**: `qtooc.0.0.1-b2 console=ttyS0,115200n8`
- **Connection**: hardware mode — USB serial `/dev/ttyUSB5` @ 115200; SSH over
  `cross-eth` (guest `10.10.129.183`, host `10.10.129.1`)
- **Boot detection patterns**: `"Press <DEL> or <ESC> to enter setup"`,
  `"skip startup.nsh or any other key"`

## Archive Note

This archive contains a **complete power-on boot cycle**, captured because the device
was rebooted mid-session (kernel uptime resets `68016s → 0` within `boot_id=2`). In
`console.txt` the boot runs lines ~619–1925: shutdown → AMI BIOS/POST → UEFI shell /
`startup.nsh` → kernel `6.12.40` → `/init` userspace → `Welcome to Buildroot`. The
journal opens and closes on interactive shell activity, with the full boot sequence
in between.
