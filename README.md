<div align="center">

# Wild Kernels for Android devices running GKI 2.0 (5.10+)

[![License: GPL-3.0](https://img.shields.io/badge/license-GPL--3.0-blue.svg)](LICENSE)
[![Third-Party Notices](https://img.shields.io/badge/notices-THIRD__PARTY_NOTICES-lightgrey.svg)](THIRD_PARTY_NOTICES.md)

</div>

> [!CAUTION]
> Wild Kernels is not responsible for bricked devices or damage. By flashing, you assume all risk. Back up your data and understand the risks before flashing.

---

## About

Generic kernels built on [Google's GKI sources](https://android.googlesource.com/kernel/common/) with KernelSU and SUSFS for root hiding and detection evasion — broad compatibility, not guaranteed for every device.

---

## Features

When **BakaSU** and **NoMount** are selected together, supported 5.10–6.12 builds use the pinned NoMount Suite kernel stack. The matching `00_NoMount-Module` activation ZIP is published as a separate artifact and release asset; install it only with the matching BakaSU kernel build. Other root variants, and BakaSU on kernel versions the Suite patch stack does not support, keep the existing NoMount integration.

- **KernelSU / KernelSU-Next / BakaSU / FinaSU / Suki-SU Ultra / KowSU** — root implementations
- **susfs4ksu** — root hiding (incl. Ptrace Leak Fix, Unicode Fix)
- **NoMount Suite (BakaSU)** — pinned in-kernel engine plus matching activation module for supported GKI 5.10–6.12 builds
- **NoMount / Mountify** — existing metamodule path for other root variants and unsupported Suite kernel versions
- **Baseband Guard** — partition protection
- **Networking** — WireGuard, BBR, IPSet, CIFS
- **TMPFS** — xattr / POSIX ACLs
- **BPF** — BTF / eBPF / FUSE-BPF
- **Performance** — incl. NTSync
- **DroidSpaces** — container runtime
- **Thermal & Power Efficiency (`android15-6.6` fork addition)** — KMI/ABI-safe idle wakeup reduction (`CONFIG_WQ_POWER_EFFICIENT_DEFAULT=y`, lazy RCU enabled by default)

> [!TIP]
> Full documentation: [Wiki](https://github.com/WildKernels/GKI_KernelSU_SUSFS/wiki)

---

## Fork Customizations (`ampereforest` / `wild-trsy`)

This fork adds a strictly **KMI/ABI-safe thermal and power-efficiency config fragment** scoped to **`android15-6.6`** (`6.6.x-android15`) builds, designed to cut unnecessary idle CPU wakeups and standby/light-use heat with **zero frequency capping and zero performance loss**.

### What Was Added

| Path | Purpose |
| --- | --- |
| [`.github/actions/thermal-efficiency/thermal-efficiency.fragment`](.github/actions/thermal-efficiency/thermal-efficiency.fragment) | Defines the two KMI-safe kernel config options appended to `wild_defconfig.fragment` |
| [`.github/actions/thermal-efficiency/action.yml`](.github/actions/thermal-efficiency/action.yml) | Composite GitHub Action that applies the fragment only when `inputs.version == 'android15-6.6'` |
| [`.github/workflows/build.yml`](.github/workflows/build.yml) | Invokes **Apply Thermal & Power Efficiency Configs** right after `Apply Device-Specific Patches` |

### What Each Config Option Does

| Config | Source Default Changed | What It Does |
| --- | --- | --- |
| `CONFIG_WQ_POWER_EFFICIENT_DEFAULT=y` | `wq_power_efficient = true` (`kernel/workqueue.c`) | Allows workqueues flagged `WQ_POWER_EFFICIENT` to be scheduled as unbound work on an **already-awake CPU** instead of waking a deep-idle core. Reduces idle core wakeups and heat while leaving latency-critical bound workqueues and CPU max frequencies completely untouched. |
| `# CONFIG_RCU_LAZY_DEFAULT_OFF is not set` | `enable_rcu_lazy = true` (`kernel/rcu/tree.c`) | Stock `android15-6.6` GKI already compiles in `CONFIG_RCU_LAZY=y` but leaves it disabled at boot (`CONFIG_RCU_LAZY_DEFAULT_OFF=y`). Clearing `DEFAULT_OFF` activates lazy RCU batching by default on offloaded `rcu_nocbs` CPUs, coalescing non-urgent `call_rcu()` callbacks to avoid frequent idle wakeups. Urgent/synchronous RCU paths (`call_rcu_hurry()`, `synchronize_rcu()`, memory reclaim flushes) remain immediate with no throughput or UI latency regression. |

### KMI/ABI Safety & CI Verification

- **Zero KMI/ABI drift:** Both options only change static `bool` initializers inside `.c` files — no struct layouts, headers, or exported symbols are altered.
- **Verified in GitHub Actions:** Confirmed on `6.6.x-android15` (`wild-trsy`, `4k`, `KernelSU` + `SUSFS` + `DroidSpaces`, [Release `r24`](https://github.com/ampereforest/GKI_KernelSU_SUSFS/releases/tag/r24)):
  - `ABI gate: OK - 9317 symbols compared against stock, 0 differ` (`6.6.118-android15-wild-trsy`)
  - Built `.config` verified: `CONFIG_WQ_POWER_EFFICIENT_DEFAULT=y` and `# CONFIG_RCU_LAZY_DEFAULT_OFF is not set`
- **Runtime overrideable:** If ever needed for testing, `wq_power_efficient` can be toggled live via `/sys/module/workqueue/parameters/power_efficient` (or disabled at boot with `workqueue.power_efficient=N` / `rcutree.enable_rcu_lazy=N`).

---

## Build Your Own Kernel

Fork the repository and follow **[Build Your Own Kernel](docs/build-from-fork.md)** to select one kernel family, patch level, root implementation, and feature set in GitHub Actions.

---

## Installation

See **[Installation Guide](https://github.com/WildKernels/GKI_KernelSU_SUSFS/wiki/Installation)**.

---

## Our Projects

| Device | Repository | Description |
| -------- | ------------ | ------------- |
| **Multi** | [GKI_KernelSU_SUSFS](https://github.com/WildKernels/GKI_KernelSU_SUSFS) | Google GKI sources — built to be generic and work across many devices |
| **Pixel** | [Sultan_KernelSU_SUSFS](https://github.com/WildKernels/Sultan_KernelSU_SUSFS) | Custom kernels for specific Pixel devices — built from Sultan sources |
| **Samsung** | [Samsung_KernelSU_SUSFS](https://github.com/WildKernels/Samsung_KernelSU_SUSFS) | Built from Samsung sources and manifest |
| **OnePlus** | [OnePlus_KernelSU_SUSFS](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS) | Built from OnePlus sources and manifest |

---

## Special Thanks

**These amazing people and projects make this possible:**

- **KernelSU** — [tiann](https://github.com/tiann/KernelSU)
- **KernelSU-Next** — [rifsxd](https://github.com/KernelSU-Next/KernelSU-Next)
- **KernelSU-Next SUSFS Fork** — [pershoot](https://github.com/pershoot/KernelSU-Next)
- **BakaSU** (formerly ReSukiSU) — [BakaSU](https://github.com/Baka-SU/BakaSU)
- **FinaSU** — [LyraVoid](https://github.com/LyraVoid/FinaSU)
- **Suki-SU Ultra** — [SukiSU Ultra](https://github.com/sukisu-ultra/sukisu-ultra)
- **KowSU** (KOWX712's KernelSU fork) — [KowSU](https://github.com/KOWX712/KernelSU)
- **Magic-KSU** — [5ec1cff](https://github.com/5ec1cff/KernelSU)
- **SUSFS** — [simonpunk](https://gitlab.com/simonpunk/susfs4ksu)
- **SUSFS Module** — [sidex15](https://github.com/sidex15)
- **NoMount** — [maxsteeel](https://github.com/maxsteeel/nomount)
- **DroidSpaces-OSS** — [ravindu644](https://github.com/ravindu644/Droidspaces-OSS)
- **Baseband-guard (BBG)** — [vc-teahouse](https://github.com/vc-teahouse/Baseband-guard)
- **Kernel Patches** — [WildKernels/kernel_patches](https://github.com/WildKernels/kernel_patches)
- **AnyKernel3** — [osm0sis](https://github.com/osm0sis/AnyKernel3)
- **Sultan Kernels (Pixel)** — [kerneltoast](https://github.com/kerneltoast)
- **Device Boot Fix** — [Boot fix commit](https://github.com/Anything-at-25-00/android_kernel_common_android12-5.10/commit/2476d262b597fe8af82cfb7aaf96676f51c6b4ed)

**Contributors to this repository:**

[![Contributors](https://contrib.rocks/image?repo=WildKernels/GKI_KernelSU_SUSFS)](https://github.com/WildKernels/GKI_KernelSU_SUSFS/graphs/contributors)

Have an idea or improvement in mind? Contributions are always welcome — feel free to open a pull request or share your thoughts!

---

## Community

<div align="center">

[![Telegram Group](https://img.shields.io/badge/Telegram-%40WildKernelsTG-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/WildKernelsTG)
[![Telegram DM](https://img.shields.io/badge/Telegram-%40TheWildJames-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/TheWildJames)

</div>

Need help? Open an issue in this repository or reach out on Telegram. Please ask in the [WildKernelsTG group](https://t.me/WildKernelsTG) first for general issues. DMs to [@TheWildJames](https://t.me/TheWildJames) are always open — use for priority / very important, or if you just want to talk and learn.

---

## Donations

> [!IMPORTANT]
> **Kind note:** A donation is truly just a gift — not a payment for support, features, or priority. It doesn't unlock anything extra on our side and doesn't change how we help you; everyone gets the same community support whether you donate or not. Think of it as a kind “thank you” to help keep development going — not a transaction. If you do choose to give, we're genuinely grateful, but please never feel obligated.

- PayPal: [bauhd@outlook.com](mailto:bauhd@outlook.com)
- Card: <https://buy.stripe.com/5kQ28sdi08Nr0Xc2fU5os00>
- LTC: `MVaN1ToSuks2cdK9mB3M8EHCfzQSyEMf6h`
- BTC: `3BBXAMS4ZuCZwfbTXxWGczxHF4isymeyxG`
- ETH: `0x2b9C846c84d58717e784458406235C09a834274e`
- Patreon: <https://patreon.com/WildKernels>
