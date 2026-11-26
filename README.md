# CachyOS Kernel (znver4 + ThinLTO) for Fedora 45

This repository builds **`kernel-cachyos-znver4-lto`** — the CachyOS BORE kernel compiled with **Clang ThinLTO** and optimized specifically for **AMD Zen 4 / Zen 5 (`znver4`)** CPUs on **Fedora 45 (`fedora-45-x86_64`)**.

Based on [CachyOS/copr-linux-cachyos](https://github.com/CachyOS/copr-linux-cachyos) and inspired by [itotm/copr-linux-cachyos-znver4](https://github.com/itotm/copr-linux-cachyos-znver4).

## Kernel Details

- **Package Name**: `kernel-cachyos-znver4-lto`
- **Target OS**: Fedora 45 (`fedora-45-x86_64`)
- **Scheduler**: BORE with `sched-ext` (`scx`) support
- **Microarchitecture**: `znver4` (`CONFIG_MZEN4=y` -> `-march=znver4` / `-Ctarget-cpu=znver4`, requires AMD Zen 4 or Zen 5)
- **Compiler**: Clang / LLVM with **ThinLTO** (`CONFIG_LTO_CLANG_THIN=y`)
- **Tickrate**: 1000 Hz
- **SELinux & Secure Boot IMA**: Enabled by default for Fedora compatibility (includes the `rpm-ostree` 2026.1 `depmod` fix)

## Features

- Cachy Sauce scheduler and kernel tweaks
- BORE CPU scheduler + `sched-ext` support
- AMD P-State Preferred Core & CPB enhancements
- Latest ZSTD patchset & BBRv3 TCP congestion control
- `v4l2loopback` and `NTSync` built in
- `dkms-clang.patch` included for building out-of-tree modules with Clang

## Automated Build System

[`.github/workflows/build-kernel-znver4-lto.yml`](.github/workflows/build-kernel-znver4-lto.yml) runs daily (and on manual `workflow_dispatch`):

1. Resolves the latest stable `CachyOS/linux` release tag (highest semver, ignoring LTS and RCs) and verifies matching `CachyOS/kernel-patches` exist.
2. Checks whether the version is already built on COPR (skipping quickly when up to date).
3. Patches [`sources/kernel-cachyos-bore/kernel-cachyos-lto.spec`](sources/kernel-cachyos-bore/kernel-cachyos-lto.spec), builds the `.src.rpm`, and submits it to Fedora COPR for full compilation in the `fedora-45-x86_64` chroot.
4. Commits the bumped spec back to the repository only after COPR accepts the build.

### Optional GitHub Repository Variables

The workflow automatically reads your COPR username from the `COPR_TOKEN` secret, so no variables are strictly required if you use the default project name (`kernel-cachyos-znver4-lto`) and chroot (`fedora-45-x86_64`). You can override any of them under **Settings → Secrets and variables → Actions → Variables**:

| Variable | Default | Description |
| --- | --- | --- |
| `COPR_PROJECT` | `kernel-cachyos-znver4-lto` | Name of your COPR project |
| `COPR_USERNAME` | *(parsed from `COPR_TOKEN`)* | Your Fedora FAS / COPR username |
| `COPR_CHROOT` | `fedora-45-x86_64` | Target COPR chroot |

## Deployment Setup (GitHub + COPR)

1. **Push this repository to GitHub** (e.g. `UHDbits/fedora-cachyos-znver4`).
2. **Create a COPR project** at [copr.fedorainfracloud.org](https://copr.fedorainfracloud.org):
   - **Project name**: `kernel-cachyos-znver4-lto`
   - **Chroots**: enable `fedora-45-x86_64`
   - **Enable internet access during builds**: checked (optional since the SRPM bundles all sources)
3. **Get your COPR API token** from [copr.fedorainfracloud.org/api/](https://copr.fedorainfracloud.org/api/) and copy the entire `[copr-cli]` block.
4. **Add the GitHub Actions secret**:
   - In your GitHub repo go to **Settings → Secrets and variables → Actions → Secrets → New repository secret**
   - **Name**: `COPR_TOKEN`
   - **Value**: paste the full `[copr-cli]` config block from step 3.
5. **Trigger the first build**:
   - Go to **Actions → Build kernel-cachyos-znver4-lto → Run workflow** and click **Run workflow**.

## Installation on Fedora 45

Replace `<your-copr-user>` with your Fedora COPR username:

```bash
# Enable your COPR repository
sudo dnf copr enable <your-copr-user>/kernel-cachyos-znver4-lto

# Install the kernel and matching devel headers
sudo dnf install kernel-cachyos-znver4-lto kernel-cachyos-znver4-lto-devel-matched

# Allow SELinux module loading
sudo setsebool -P domain_kernel_load_modules on
```

### Fedora Atomic / Silverblue / Kinoite / Bazzite

```bash
sudo curl -fsSL \
  "https://copr.fedorainfracloud.org/coprs/<your-copr-user>/kernel-cachyos-znver4-lto/repo/fedora-45/<your-copr-user>-kernel-cachyos-znver4-lto-fedora-45.repo" \
  -o /etc/yum.repos.d/_copr-kernel-cachyos-znver4-lto.repo

sudo rpm-ostree override remove kernel kernel-core kernel-modules kernel-modules-core kernel-modules-extra \
  --install kernel-cachyos-znver4-lto
sudo systemctl reboot
```

### Always Boot the Latest CachyOS Kernel by Default

```bash
sudo dnf install libdnf5-plugin-actions
sudo mkdir -p /etc/dnf/libdnf5-plugins/actions.d
sudo tee /etc/dnf/libdnf5-plugins/actions.d/cachy-default.actions << 'EOF'
post_transaction:kernel*:in::/usr/bin/sh -c /usr/bin/grubby\ --set-default=/boot/$(ls\ /boot\ |\ grep\ vmlinuz.*cachy\ |\ sort\ -V\ |\ tail\ -1)
EOF
```

## Credits

- [CachyOS](https://cachyos.org)
- [CachyOS/linux](https://github.com/CachyOS/linux)
- [CachyOS/copr-linux-cachyos](https://github.com/CachyOS/copr-linux-cachyos)
- [itotm/copr-linux-cachyos-znver4](https://github.com/itotm/copr-linux-cachyos-znver4)
