# Issue 1: Upgrading Kernel on Liontron VEC-3588 Board

## What the GitHub issue says
- A user has a Liontron VEC-3588 single-board computer (based on the Rockchip RK3588 SoC) running Debian 11 with a 5.10.x kernel.
- They want to upgrade to a newer kernel (6.x) because newer GPU drivers (e.g., Collabora's Panfrost/Mali G610 support) and newer glibc versions require it.
- Concern: keeping Liontron's vendor customizations, drivers, and parameters that ship with the existing firmware.

### Comments in the issue
- **Project owner response (Dec 28, 2025):** The main work is adapting the device tree (DTB). The vendor does not provide source code for the kernel or DTB, so the DTB must be reverse-engineered from the compiled binary. This is active, ongoing work.
- **Issue author reply:** Thanks the maintainer and is following the work.

## Background (newbie-friendly)
- **Kernel version:** Linux 5.10 is an older long-term support kernel. Newer Rockchip RK3588 drivers and glibc releases target 6.x kernels, so upgrading can improve hardware support and software compatibility.
- **DTB (Device Tree Blob):** A DTB describes the board's hardware (pins, regulators, buses, clocks, etc.) so the kernel can drive it correctly. Vendors typically ship a DTB alongside the kernel. If the DTB source (`.dts`) is unavailable, it can be decompiled from the binary so it can be adapted for a new kernel.
- **Why DTB adaptation matters:** When switching to a newer kernel, hardware definitions and driver bindings often change. Adapting the DTB ensures power rails, GPIOs, clocks, and peripheral nodes (e.g., GPU, Wi‑Fi, storage) match what the new kernel expects.

## What is required to solve the issue
1. **Extract the existing DTB** from the vendor firmware (e.g., from `/boot` or the firmware image) and decompile it with tools like `dtc`.
2. **Compare with upstream RK3588 DT bindings** (from mainline or BSP trees) to adjust nodes, compatible strings, clocks, and regulators for kernel 6.x.
3. **Carry over vendor-specific tweaks** (GPIOs, power sequences, regulator voltages, panel timings, PCIe tunings, etc.) into the updated DT.
4. **Rebuild and test a kernel 6.x** with the adapted DTB, verifying boot, display, GPU, USB, PCIe, storage, networking, and power management.
5. **Iterate** based on boot logs (`dmesg`) to fix missing drivers or incorrect bindings.

## Constraints and challenges
- **Closed vendor sources:** No official kernel or DT source is available, so reverse engineering is necessary.
- **Driver gaps:** Some peripherals may still require out-of-tree drivers or firmware that need to be sourced from the vendor image.
- **Stability testing:** Extensive hardware testing is needed after the DTB is adapted to ensure the board operates reliably under kernel 6.x.

## Expected outcome
- A bootable kernel 6.x for the Liontron VEC-3588 with a reworked DTB that preserves vendor-specific hardware settings and enables newer drivers and glibc support.
