# Device Info

## Machine

- **QEMU machine**: `-M virt`
- **CPU**: `cortex-a72` (aarch64)
- **Memory**: 1024 MB
- **Boot path**: direct-kernel — `-kernel Image -append "console=ttyAMA0 earlycon=pl011,0x09000000 root=/dev/vda2 rw rootwait init=/usr/lib/systemd/systemd"`
- **Drive**: virtio-blk via `-drive file=sdcard.img,format=raw,if=virtio` (i.MX8M sdcard.img assembled by phase-10's image-combiner)
- **Network**: `-net nic -net tap,ifname=tap0,script=no,downscript=no`
- **Serial**: PL011 → PTY (`-serial pty`)

QEMU's stock `-M virt` defaults to `-cpu cortex-a15` (32-bit ARMv7); the `cortex-a72` override is required for aarch64 code to execute. Set automatically by `lab_mcp`'s `MACHINE_DEFAULT_CPU` (commit 8cedd07).

## Operating System

- **OS**: Debian GNU/Linux 13 (trixie)
- **Kernel**: 6.6.52-g5a0a5e71d2bd (NXP linux-imx `lf-6.6.y`, `imx_v8_defconfig`, aarch64, PREEMPT)
- **Hostname**: `debuerreotype` (default for mmdebstrap-built rootfs — not overridden)
- **Init**: systemd 257 (pinned via `init=/usr/lib/systemd/systemd` to bypass the kernel's `/sbin/init` symlink walk)

## Login Path

- **Service**: custom `serial-login.service` (runs `/sbin/agetty ttyAMA0` directly; bypasses the systemd-shipped `serial-getty@ttyAMA0.service` which `BindsTo=dev-%i.device` and times out 1m30s waiting for a udev event that never fires under QEMU virt)
- **Credentials**: `root` / `0penBmc` (set by the `setup-root.sh` chroot hook in the `imx8m-pipeline-test/debian` project)
- **Boot operation**: `login_shell` — see `lab_mcp/src/qemu_server/operations/login.py`

## Pipeline Origin

This boot cycle was produced by the platform-tier full-pipeline E2E test
[`tests/test_imx8m_pipeline.py::TestImx8mPipeline::test_full_pipeline_build_combine_boot`](https://github.com/firmwai/firmwai-platform/blob/phase-11-imx8m-pipeline/tests/test_imx8m_pipeline.py).

Build chain: `debian-trixie-arm64` (rootfs.tar.gz) + `imx8m-kernel` (kernel-artifacts.tar.gz) + `imx8m-boot-firmware` (boot-firmware-artifacts.tar.gz) → `imx8m-image` (sdcard.img) → `lab_mcp` QEMU.

## Why This Cycle Is Interesting

- First archived cycle that exercises lab_mcp commit `9623587` ("trigger BOOT_START on first-boot login transition"). The original `TerminalState` only emitted `BOOT_START` on `*→FIRMWARE` or `SHELL→LOGIN`; this cycle's `OFF→BOOTING→LOGIN` path would have stayed silent before, and `wait_for_event(["shell_ready"])` would have timed out.
- First archived cycle on `-M virt`. Different from `ast2600-evb` (no PL011-on-emulated-AST), `hardware/rpi4-linux-boot` (real PL011 over USB-serial, no virtio-blk), or any of the OpenBMC cycles.
- Boot-firmware artifacts are produced through the pipeline and validated as a packaging contract, but not used during this boot — phase-11 commits to `-M virt` direct-kernel; hardware-faithful boot-from-flash.bin is deferred. See [phase-11 leftovers Item 2 / Item 5](https://github.com/firmwai/builder/blob/phase-11-imx8m-pipeline/docs/exec-plans/active/debian-builder/phase-11-full-pipeline-e2e-leftovers.md).

## Reproduce

```bash
cd platform/
./scripts/platform-up.sh                                  # starts firmwai-minio, builder, firmwai-qemu
uv run pytest tests/test_imx8m_pipeline.py -v -s         # ~6 min with warm Docker layer cache
```
