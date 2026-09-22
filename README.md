# NCVM

**ncvm** — CodeOS-minimal QEMU fork. Two tiny system emulators built from
QEMU v10.2.4, stripped to only what CodeOS needs:

| Binary              | Machine                | Purpose                     |
|---------------------|------------------------|-----------------------------|
| `bin/ncvm-x86_64`   | q35 only               | CodeOS desktop (x86_64)     |
| `bin/ncvm-aarch64`  | virt only              | CodeOS / Zircon ARM64       |

The x86_64 build boots the full CodeOS Limine ISO (SeaBIOS → Limine →
kernel → Qt6/HyperDE desktop) at 60fps under TCG. The aarch64 build boots
EDK2 on the virt machine.

## What is stripped

- Boards: only `q35` (x86_64) and `virt` (aarch64) remain — `pc`, `microvm`,
  `isapc` and dozens of ARM boards are compiled out via
  `configs/devices/*/codeos.mak` (`--without-default-devices`).
- Legacy device set removed (FDC, ne2k/pcnet/rtl8139, ac97/sb16, uhci/ohci,
  cirrus-vga, PIIX IDE, ...).
- Kept devices: e1000 + virtio-net, USB-EHCI/XHCI + HID (tablet/kbd), ICH9
  AHCI, std VGA (+EDID), ISA serial.
- ACPI_CXL pruned from the machine configs and guarded out of the ACPI
  builders (`patches/0001-codeos-strip-acpi-cxl.patch`) so the firmware
  tables build without the CXL machinery.

## Quickstart

Requires: `git`, `gcc`, `make`, `ninja`, `pkg-config`, `python3`, `glib2`
and `pixman` dev headers (Arch: `base-devel ninja python pkgconf glib2
pixman`).

```sh
git clone https://github.com/tech-for-everyone/NCVM.git
cd NCVM
bash build-codeos.sh          # clones QEMU v10.2.4 into src/, patches it,
                              # builds both targets into bin/, installs data
                              # to share/qemu
./bin/ncvm-x86_64 -machine q35 -m 1G -cdrom codeos-1-kernel.iso -boot order=d
```

- Reuse an existing QEMU checkout: `NCVM_QEMU_SRC=/path/to/qemu bash build-codeos.sh`
- System install: `bash build-codeos.sh install` (binaries → `/usr/local/bin`,
  firmware → `/usr/local/share/qemu`).
- Single target: `bash build-codeos.sh x86_64` (or `aarch64`).

## Layout

```
build-codeos.sh                            build + bootstrap script
configs/devices/<arch>-softmmu/codeos.mak  per-target device deps
patches/0001-codeos-strip-acpi-cxl.patch   ACPI_CXL removal + guards
src/qemu/                                  QEMU v10.2.4 checkout (gitignored)
build-<arch>/  install-<arch>/  bin/       build outputs (gitignored)
share/qemu                                 firmware data used by bin/ (gitignored)
```

The `codeos.mak` device configs and the ACPI patch are the source of
truth; the QEMU tree itself stays pristine upstream except for these.