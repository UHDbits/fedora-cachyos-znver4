# CachyOS Kernel (znver4 + ThinLTO) for Fedora

This repository provides **`kernel-cachyos-znver4-lto`** — the CachyOS BORE kernel compiled with **Clang ThinLTO** and optimized specifically for **AMD Zen 4 / Zen 5 (`znver4`)** CPUs on **Fedora**, automatically updating for the latest kernel and Fedora releases.

Based on [CachyOS/copr-linux-cachyos](https://github.com/CachyOS/copr-linux-cachyos) and inspired by [itotm/copr-linux-cachyos-znver4](https://github.com/itotm/copr-linux-cachyos-znver4).

## Kernel Details

- **Package Name**: `kernel-cachyos-znver4-lto`
- **Target OS**: Fedora `x86_64` (automatically tracks the latest Fedora releases)
- **Scheduler**: BORE with `sched-ext` (`scx`) support
- **Microarchitecture**: `znver4` (`-march=znver4`, requires an AMD Zen 4 or Zen 5 CPU)
- **Compiler**: Clang / LLVM with **ThinLTO**
- **Tickrate**: 1000 Hz
- **SELinux & Secure Boot IMA**: Enabled by default for Fedora compatibility

## Features

- Cachy Sauce scheduler and kernel tweaks
- BORE CPU scheduler with `sched-ext` support
- AMD P-State Preferred Core & CPB enhancements
- Latest ZSTD patchset & BBRv3 TCP congestion control
- `v4l2loopback` and `NTSync` built in
- `dkms-clang.patch` included for building out-of-tree modules with Clang

## Installation

### Fedora Workstation / Standard Fedora

```bash
# Enable the COPR repository
sudo dnf copr enable uhdbits/kernel-cachyos-znver4-lto

# Install the kernel and matching devel headers
sudo dnf install kernel-cachyos-znver4-lto kernel-cachyos-znver4-lto-devel-matched

# Allow SELinux module loading
sudo setsebool -P domain_kernel_load_modules on
```

### Fedora Atomic / Silverblue / Kinoite / Bazzite

```bash
sudo curl -fsSL \
  "https://copr.fedorainfracloud.org/coprs/uhdbits/kernel-cachyos-znver4-lto/repo/fedora-$(rpm -E %fedora)/uhdbits-kernel-cachyos-znver4-lto-fedora-$(rpm -E %fedora).repo" \
  -o /etc/yum.repos.d/_copr-kernel-cachyos-znver4-lto.repo

sudo rpm-ostree override remove kernel kernel-core kernel-modules kernel-modules-core kernel-modules-extra \
  --install kernel-cachyos-znver4-lto
sudo systemctl reboot
```

### Always Boot the Latest CachyOS Kernel by Default

By default, Fedora boots whichever kernel was updated most recently. To keep the stock Fedora kernel installed as a fallback while always defaulting to the latest CachyOS kernel:

```bash
sudo dnf install libdnf5-plugin-actions
sudo mkdir -p /etc/dnf/libdnf5-plugins/actions.d
sudo tee /etc/dnf/libdnf5-plugins/actions.d/cachy-default.actions << 'EOF'
post_transaction:kernel*:in::/usr/bin/sh -c /usr/bin/grubby\ --set-default=/boot/$(ls\ /boot\ |\ grep\ vmlinuz.*cachy\ |\ sort\ -V\ |\ tail\ -1)
EOF
```

## Build System

New builds are triggered automatically via GitHub Actions and compiled on [Fedora COPR](https://copr.fedorainfracloud.org/coprs/uhdbits/kernel-cachyos-znver4-lto/):

- Checks for the latest stable `CachyOS/linux` release daily
- Automatically enables newly branched Fedora releases on COPR
- Spec file: [`sources/kernel-cachyos-bore/kernel-cachyos-lto.spec`](sources/kernel-cachyos-bore/kernel-cachyos-lto.spec)

## Credits

- [CachyOS](https://cachyos.org)
- [CachyOS/linux](https://github.com/CachyOS/linux)
- [CachyOS/copr-linux-cachyos](https://github.com/CachyOS/copr-linux-cachyos)
- [itotm/copr-linux-cachyos-znver4](https://github.com/itotm/copr-linux-cachyos-znver4)
