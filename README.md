#### The Android kernel is based on an upstream Linux [Long Term Supported (LTS) kernel](https://www.kernel.org/category/lts). At Google, LTS kernels are combined with Android-specific patches to form Android Common Kernels (ACKs).

<div align="center">
  <img width="350" alt="android-logo" src="https://github.com/user-attachments/assets/b6fe1a38-1c8d-4421-82eb-6fba2631fbf9" />
</div>


ACKs that are 5.10 and higher are also known as *generic kernel images (GKI) kernels. GKI kernels support the separation of the hardware-agnostic generic core kernel code and GKI modules from hardware-specific vendor modules.

The interaction between the GKI kernel and vendor modules is enabled by the Kernel Module Interface (KMI) consisting of symbol lists identifying the functions and global data required by vendor modules. Figure 1 shows the GKI kernel and vendor module architecture:

GKI kernel and vendor module architecture.

<div align="center">
  <img width="450" alt="aosp-kernel" src="https://github.com/user-attachments/assets/078f4481-7d68-441f-8fff-6ef8826ae5b2" />
</div>

Figure 1. GKI kernel and vendor module architecture.

💡 Notes:
> Note: The GKI kernel, GKI module, and vendor module architecture is the result of a multi-year effort known as the Generic Kernel Image (GKI) project. For information about this project and its phases, refer to The Generic Kernel Image (GKI) project.
> Note: Kernels are referred to by their platform version followed by a kernel version. For example, android15-6.6 is kernel for Android 15 with a version of 6.6.

### Kernel glossary
Following are terms used throughout the kernel documentation.

### Kernel types
#### Android Common Kernel (ACK)
A kernel that is downstream of a LTS kernel and includes patches that are important to the Android community. These patches haven't been merged into Linux mainline or Long Term GKI kernels.
Kernel with versions of 5.10 and higher are also referred to as [Generic Kernel Image (GKI)](https://source.android.com/docs/core/architecture/kernel#gkik) kernels.

#### Android Open Source Project (AOSP) kernel
See [Android Common Kernel](https://source.android.com/docs/core/architecture/kernel#ack).

Android 12 features can't be backported to 4.19 kernels; the feature set would be similar to a device that launched with 4.19 on Android 11 and upgraded to Android 12.

<div align="center">
  <img width="500" alt="aosp-architecture" src="https://github.com/user-attachments/assets/799744d5-4feb-45cf-a7f4-c1a2d04c99ff" />
</div>

Figure 2. AOSP Architecture

#### Generic Kernel Image (GKI) kernel
Any 5.10 and higher ACK kernel(aarch64 only). The GKI kernel has these two parts:

- Generic kernel - The portion of the GKI kernel that is common across all devices.
- GKI modules - Kernel modules built by Google that can be dynamically loaded on devices where applicable. These modules are built as artifacts of the GKI kernel and are delivered alongside GKI as the `system_dlkm_staging_archive.tar.gz` archive. GKI modules are signed by Google using the kernel build time key pair and are compatible only with the GKI kernel that they're built with.

#### Kernel Module Interface (KMI) kernel
See [GKI kernel](https://source.android.com/docs/core/architecture/kernel#gkik).

#### Long Term Supported (LTS) kernel
A Linux kernel that's supported for 2 to 6 years. LTS kernels are released once per year and are the basis for each of Google's Android Common Kernels.

### Branch types
#### ACK KMI kernel branch
The branch for which GKI kernels are built. Branch names correspond to kernel versions, such as `android15-6.6`.

#### Android-mainline
The primary development branch for Android features. When a new LTS kernel is declared upstream, the corresponding new GKI kernelGKI kernel is branched from android-mainline.
Linux mainline :The primary development branch for the upstream Linux kernels, including LTS kernels.

### Other terms
#### Certified boot image
The kernel delivered in binary form `boot.img` and flashed onto the device. This image is considered certified because contains embedded certificates so Google can verify that the device ships with a kernel certified by Google.

#### Dynamically loadable kernel module (DLKM)
A module that can be dynamically loaded during device boot depending on the needs of the device. GKI and vendor modules are both types of DLKMs. DLKMs are released in `.ko` form and can be drivers or can deliver other kernel functionality.

#### GKI project
A Google project addressing kernel fragmentation by separating common core kernel functionality from vendor-specific SoC and board support into loadable modules.

💡 Notes:
> Generic Kernel Image (GKI): A boot image certified by Google that contains a GKI kernel built from an ACK source tree and is suitable to be flashed to the boot partition of an Android-powered device.

#### Kernel Module Interface (KMI)
An interface between the GKI kernel and vendor modules allowing vendor modules to be updated independently of the GKI kernel. This interface consists of kernel functions and global data that have been identified as vendor/OEM dependencies using per-partner symbol lists.
Vendor module
A hardware-specific module developed by a partner and that contains SoC and device-specific functionality. A vendor module is a type of dynamically loadable kernel module

---
### 🌰 Build custom kernel

Enter the [GitHub Actions workflow](https://github.com/azagramac/build-kernel-aosp-pixel/actions/workflows/build-kernel.yml)

<div align="center">
  <img width="550" height="874" alt="image" src="https://github.com/user-attachments/assets/329e729c-793d-4411-8c4a-0e9e89ea5b73" />
</div>

Select codename device, build mode, and LTO mode.

| LTO Mode | Global Optimization | kCFI Compatibility | Build Time | Memory Usage | Typical Use                          |
| -------- | ------------------- | ------------------ | ---------- | ------------ | ------------------------------------ |
| none     | No                  | ❌ No               | Very fast  | Low          | Debug / development                  |
| thin     | Partial / fast      | ✅ Yes              | Moderate   | Moderate     | Recommended for Android builds       |
| full     | Full                | ✅ Yes              | Slow       | High         | Release builds / maximum performance |

💡 Notes:
> * For AOSP / LineageOS, Pixel kernel builds where you want security (kCFI) and reasonable build times, use **`lto=thin`**.
> * You would only use **`lto=full`** if you want to maximize kernel performance and don’t mind longer build times.


Artifacts are available on [GitHub Releases](https://github.com/azagramac/build-kernel-aosp-pixel/releases)

---
#### 🖼 Kernel & Image files overview

When building a custom Android kernel, several images are generated. Here’s a breakdown of the main images and their purpose:

| Image               | Description                                                                                                         | How it’s generated                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `boot.img`        | Contains the **kernel** and **boot ramdisk** (initramfs for platform boot). This is used for platform startup.      | Generated by `make bootimage` or `build_<codename>.sh`                                          |
| `vendor_kernel_boot.img` | Contains **vendor kernel**, **vendor initramfs** (initramfs for vendor boot), optional **DTB**, and vendor modules. | Generated by AOSP with `TARGET_BOOTIMAGE_TARGETS=vendor_bootimage` or `build_<codename>.sh`     |
| `vdtb.img`         | Flattened **Device Tree Blob** for the kernel. Defines hardware configuration.                                      | Extracted from compiled kernel (`arch/arm64/boot/dts/vendor/*.dtb`) or generated via `mkdtb.py` |
| `dtbo.img`        | **Device Tree Overlay** for modular SoC configuration. Allows optional hardware/board overlays.                     | Generated from DT overlay source: `dtc -I dts -O dtb dtbo_*.dts -o dtbo.img`                    |
| `vendor_dlkm.img` | Vendor dynamically loadable kernel modules (DLKMs). Modules loaded by vendor boot.                                  | Built as part of GKI vendor modules in AOSP (`out/<codename>/obj/vendor_dlkm`)                  |
| `system_dlkm.img` | GKI modules shipped to system partition. Modules loaded by generic kernel.                                          | Built as part of GKI system modules (`out/<codename>/obj/system_dlkm`)                          |

💡 Notes:
> Note: `initramfs.img` is embedded inside `boot.img` (platform) and `vendor_boot.img` (vendor). You do not flash it separately.

---

### 📱 Flash custom kernel

##### If you haven't disabled verification, you need to do it before flashing the custom kernel. Here is the command to do so:
```bash
fastboot oem disable-verification
```

##### ⚠️ Wipe device (if flashing on top of a platform build)
Then you may need to wipe your device if there is a security patch level (SPL) downgrade associated with the new kernel. This process erases all of your personal data. Be sure to back up your data before wiping.
```bash
fastboot -w
```

##### Flash commands:
```bash
fastboot flash boot out/{{ codename }}/dist/boot.img
fastboot flash dtbo out/{{ codename }}/dist/dtbo.img
fastboot flash dtb out/{{ codename }}/dist/dtb.img
fastboot reboot fastboot
fastboot flash vendor_boot out/{{ codename }}/dist/vendor_kernel_boot.img
fastboot flash vendor_dlkm out/{{ codename }}/dist/vendor_dlkm.img
fastboot flash system_dlkm out/{{ codename }}/dist/system_dlkm.img
fastboot reboot
```
> Reboot into fastboot mode ensures the device is ready to accept vendor and system DLKMs.

💡 Notes:
- Do not flash `initramfs.img` directly; it is embedded inside `vendor_boot.img`.
- Reboot into fastboot before flashing vendor_dlkm or system_dlkm to ensure the device is ready.
- Order matters: Boot → DTBO/DTB → Vendor & System DLKM.

##### Enable serial logs (optional):
```bash
fastboot oem uart enable
fastboot oem uart config 3000000
```
Example:
```bash
screen -fn /dev/ttyUSB* 3000000
```

##### Restore the factory images
To restore your device back to the factory images, you can use [flash.android.com](https://flash.android.com/welcome).
