# CachyOS Kernel (znver4 + ThinLTO) for Fedora 45+

This repository builds **`kernel-cachyos-znver4-lto`** — the CachyOS BORE kernel compiled with **Clang ThinLTO** and optimized specifically for **AMD Zen 4 / Zen 5 (`znver4`)** CPUs on **Fedora 45 and future Fedora releases**.

Based on [CachyOS/copr-linux-cachyos](https://github.com/CachyOS/copr-linux-cachyos) and inspired by [itotm/copr-linux-cachyos-znver4](https://github.com/itotm/copr-linux-cachyos-znver4).

## Kernel Details

- **Package Name**: `kernel-cachyos-znver4-lto`
- **Target OS**: Fedora 45+ (`fedora-45-x86_64`, automatically expanding to `fedora-46-x86_64`, `fedora-47-x86_64`, etc. as new Fedora releases branch on COPR)
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

## Automated Build & Future Fedora Version Support

[`.github/workflows/build-kernel-znver4-lto.yml`](.github/workflows/build-kernel-znver4-lto.yml) runs daily (and on manual `workflow_dispatch`):

1. Resolves the latest stable `CachyOS/linux` release tag (highest semver, ignoring LTS and RCs) and verifies matching `CachyOS/kernel-patches` exist.
2. Queries COPR's active chroot list for all `fedora-<N>-x86_64` chroots with `N >= MIN_FEDORA_VERSION` (default `45`). Whenever a new Fedora version branches on COPR (e.g. `fedora-46-x86_64`), the workflow **automatically enables that chroot** on your COPR project and submits a build for it—no manual repo edits required.
3. Skips quickly when both the kernel version and Fedora chroots are already up to date.
4. Patches [`sources/kernel-cachyos-bore/kernel-cachyos-lto.spec`](sources/kernel-cachyos-bore/kernel-cachyos-lto.spec), builds the `.src.rpm`, and submits it to Fedora COPR for all enabled chroots (each binary RPM automatically gets its Fedora dist tag via `%{?dist}`, e.g. `.fc45`, `.fc46`).
5. Commits the bumped spec back to the repository only after COPR accepts the build.

### Optional GitHub Repository Variables

The workflow automatically reads your COPR username from the `COPR_TOKEN` secret, so no variables are required if you use the defaults. You can override any of them under **Settings → Secrets and variables → Actions → Variables**:

| Variable | Default | Description |
| --- | --- | --- |
| `COPR_PROJECT` | `kernel-cachyos-znver4-lto` | Name of your COPR project |
| `COPR_USERNAME` | *(parsed from `COPR_TOKEN`)* | Your Fedora FAS / COPR username |
| `MIN_FEDORA_VERSION` | `45` | Automatically enable and build all active `fedora-<N>-x86_64` chroots from this version onward |
| `COPR_CHROOT` | *(unset — all enabled chroots)* | Set to pin builds to a single chroot (disables automatic future Fedora chroot sync) |

## Deployment Setup (GitHub + COPR)

1. **Push this repository to GitHub** (e.g. `UHDbits/fedora-cachyos-znver4`).
2. **Create a COPR project** at [copr.fedorainfracloud.org](https://copr.fedorainfracloud.org):
   - **Project name**: `kernel-cachyos-znver4-lto`
   - **Chroots**: enable `fedora-45-x86_64` *(future `fedora-46-x86_64`, etc. will be enabled automatically by the workflow; optionally also enable `fedora-rawhide-x86_64` with **Follow Fedora branching** checked if you want Rawhide too)*
   - **Enable internet access during builds**: checked (optional since the SRPM bundles all sources)
3. **Get your COPR API token** from [copr.fedorainfracloud.org/api/](https://copr.fedorainfracloud.org/api/) and copy the entire `[copr-cli]` block.
4. **Add the GitHub Actions secret**:
   - In your GitHub repo go to **Settings → Secrets and variables → Actions → Secrets → New repository secret**
   - **Name**: `COPR_TOKEN`
   - **Value**: paste the full `[copr-cli]` config block from step 3.
5. **Trigger the first build**:
   - Go to **Actions → Build kernel-cachyos-znver4-lto → Run workflow → Run workflow**.

## Installation on Fedora 45+

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
  "https://copr.fedorainfracloud.org/coprs/<your-copr-user>/kernel-cachyos-znver4-lto/repo/fedora-$(rpm -E %fedora)/<your-copr-user>-kernel-cachyos-znver4-lto-fedora-$(rpm -E %fedora).repo" \
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
