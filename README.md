# cix-gpu-kmd

Out-of-tree Mali kernel driver (`mali_kbase`) for the CIX Sky1 / CD8180 SoC
with Mali-G720-Immortalis MC10 GPU (Valhall/Titan architecture).

Based on ARM Mali DDK r54p1 sources from the CIX cix-go-2025q4 SDK, with
patches to build against mainline kernels 6.15 through 7.0.

Part of the [Sky1 Linux](https://github.com/Sky1-Linux) project.

## Modules

The build produces three kernel modules:

| Module | Description |
|--------|-------------|
| `mali_kbase` | Main GPU kernel driver |
| `memory_group_manager` | Memory group manager |
| `protected_memory_allocator` | Protected memory allocator |

## Kernel Compatibility

| Kernel | Status | Notes |
|--------|--------|-------|
| 6.12 LTS | Builds | Vendor baseline |
| 6.15+ | Builds | hrtimer_setup, MALI_CFLAGS rename |
| 6.16+ | Builds | fence_value_str removal |
| 6.17+ | Builds | Requires PGTY_mali_gpu kernel patch (see below) |
| 6.18-6.19 | Builds | mm_get_unmapped_area signature change |
| 7.0 | Builds | Tested with linux-next |

### PGTY_mali_gpu kernel patch (6.17+)

Kernel 6.17 changed the page migration `movable_ops` API to a page-type-based
scheme. The driver requires a `PGTY_mali_gpu` page type to be defined in the
kernel. Sky1 Linux kernels carry this patch; if you are building against a
different kernel tree, you must apply it. The patch adds a single page type
constant to `include/linux/page-flags.h`.

## Building with DKMS

Install the source tree into the DKMS directory and build:

```bash
sudo cp -r . /usr/src/cix-gpu-kmd-1.0.0
sudo dkms add cix-gpu-kmd/1.0.0
sudo dkms build cix-gpu-kmd/1.0.0
sudo dkms install cix-gpu-kmd/1.0.0
```

Or install the pre-built DKMS deb package:

```bash
sudo apt install cix-gpu-kmd-dkms
```

After installation, load the driver:

```bash
sudo modprobe mali_kbase
```

## Manual Build

```bash
make KDIR=/lib/modules/$(uname -r)/build all
sudo make KDIR=/lib/modules/$(uname -r)/build modules_install
```

## Porting Patches

The following patches (on top of the vendor import) adapt the driver for
mainline kernels:

1. Fix DKMS build: pass KDIR to make invocations
2. Replace EXTRA_CFLAGS with MALI_CFLAGS (removed in kernel 6.15)
3. Fix Kbuild conditionals for out-of-tree module builds
4. Port to kernel 6.15+: hrtimer_setup and timer_delete_sync
5. Port to kernel 6.16+: remove fence_value_str callback
6. Port to kernel 6.17+: page migration movable_ops API
7. Port to kernel 6.19+: mm_get_unmapped_area signature change
8. Make IPA thermal initialization failure non-fatal (ACPI boot)
9. Fix Sky1 platform: remove vendor SCMI ifdefs, fix dev_pm_domain_detach args

## License

GPLv2. See `license.txt`.
