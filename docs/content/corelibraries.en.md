# LinuxToys Scripting API

LinuxToys supports full Bash scripts for software and features that require more control than a [repository listing](repositorylists.html). These scripts can take advantage of LinuxToys' compatibility detection, package-management abstraction, filesystem transaction tracking, systemd helpers, bootloader management, user interaction facilities, and automatic rollback system.

This document covers the scripting interface for regular LinuxToys `.sh` scripts.

Repository lists are a separate feature with their own documentation and are intentionally not covered here.

---

## Basic Script Structure

A LinuxToys script is a normal Bash script with a metadata header at the beginning:

```bash
# name: Example Application
# version: 1.0
# description: Installs and configures Example Application.
# icon: example.svg
# compat: debian, ubuntu, fedora, arch
# reboot: no
# revert: yes

pkg_install example-package

info "$finishmsg"
```

LinuxToys reads metadata only from the initial comment block. Once the parser reaches the first line that does not begin with `#`, header parsing stops.

For that reason, all LinuxToys headers must appear before the executable part of the script.

You generally **do not need to source LinuxToys libraries yourself**. When LinuxToys launches a script, its library loader examines the script, determines which split libraries are needed, exports the appropriate library flags, and sources the main `linuxtoys.bash` library before sourcing the actual script.

The resulting execution is conceptually similar to:

```bash
export SCRIPT_DIR="/path/to/linuxtoys"
export FS_OPS=1
export PACKAGE_OPS=1
# ...

source "$SCRIPT_DIR/libs/linuxtoys.bash"
source "/path/to/script.sh"
```

The real preamble is constructed automatically by `library_loader.py`.

---

## Display Metadata Headers

### `name`

```bash
# name: Example Application
```

Sets the user-facing name of the script.

This name is displayed in LinuxToys and is also used when a child script called through `call_script` is registered as a transaction.

If no name is provided, normal scripts default to `No Name`; local user scripts may fall back to their filename.

For scripts distributed with LinuxToys, always provide a name.

---

### `version`

```bash
# version: 1.4
```

Provides version metadata for the script.

The normal parser defaults this field to `N/A`.

This is the version of the LinuxToys script/feature definition, not necessarily the version of the software it installs.

---

### `description`

```bash
# description: Installs the Example graphics configuration utility.
```

Provides the user-facing description shown by LinuxToys.

`name` and `description` are also eligible for LinuxToys' translation lookup when translations are supplied to the parser.

---

### `icon`

```bash
# icon: example.svg
```

Specifies the icon associated with the feature.

Without one, the parser defaults to:

```text
application-x-executable
```

---

### `repo`

```bash
# repo: https://github.com/example/example
```

Associates an upstream repository with the feature.

If omitted, this field is empty.

---

## Operating-System Compatibility

### `compat`

```bash
# compat: debian, ubuntu, fedora
```

Restricts a script to systems matching at least one supplied compatibility key.

Multiple keys are comma-separated.

For example:

```bash
# compat: arch, cachy, manjaro
```

means:

> Show/run this script if the machine matches Arch, CachyOS, or Manjaro.

The header also supports exclusions with `!`:

```bash
# compat: debian, !deepin
```

LinuxToys separates the supplied entries into inclusion and exclusion sets. If inclusion keys exist, at least one must match. Any matching exclusion key then makes the script incompatible.

A blacklist-only rule is therefore also valid:

```bash
# compat: !ubuntu
```

This permits the script everywhere compatible with LinuxToys except Ubuntu.

### Useful OS keys

Current compatibility detection includes Linux families and derivatives such as:

```text
debian
ubuntu
arch
cachy
manjaro
fedora
rhel
suse
ostree
ublue
solus
zorin
deepin
pika
```

Do not perform your own `/etc/os-release` parsing simply to distinguish the common supported families. The shell library exposes matching functions for runtime branching as described later in this document.

---

## GPU Compatibility

### `gpu`

Restrict a script according to graphics hardware or compute capability:

```bash
# gpu: amd
```

Supported special values are:

```text
amd
intel
nvidia
rocm
xe
```

They correspond to LinuxToys compatibility keys:

```text
gpu-amd
gpu-intel
gpu-nvidia
gpu-rocm
gpu-xe
```

Multiple GPU targets can be supplied:

```bash
# gpu: amd, intel
```

Any matching target is sufficient.

### `rocm`

```bash
# gpu: rocm
```

Requires hardware considered ROCm-capable by LinuxToys.

### `xe`

```bash
# gpu: xe
```

Requires an Intel GPU considered compatible with the Intel Compute Runtime/Xe path.

LinuxToys' compatibility layer separately generates `gpu-rocm`, `gpu-xe`, and hybrid-GPU keys where appropriate.

---

## CPU Compatibility

### `cpu`

Restricts a feature according to CPU vendor:

```bash
# cpu: amd
```

or:

```bash
# cpu: intel
```

Multiple values may be specified:

```bash
# cpu: amd, intel
```

The recognized specific values map to `cpu-amd` and `cpu-intel`. Other values are treated as requiring a generic CPU key.

---

## Desktop Environment Compatibility

### `desktop`

Restricts a script according to the active desktop:

```bash
# desktop: gnome
```

Currently recognized specific values are:

```text
gnome
plasma
other
```

Multiple desktops can be allowed:

```bash
# desktop: gnome, plasma
```

These map internally to:

```text
desktop-gnome
desktop-plasma
desktop-other
```

Use this header when a feature fundamentally depends on a desktop environment. If the script merely needs to perform slightly different operations on different desktops, it can instead branch at runtime.

---

## systemd Compatibility

### `systemd`

Require systemd:

```bash
# systemd: yes
```

An empty header has the same meaning:

```bash
# systemd:
```

Require a non-systemd system instead:

```bash
# systemd: no
```

LinuxToys also automatically detects some implicit systemd dependencies. If a script calls `pkg_flat` or a function matching `sysd_*` and does not have an explicit `systemd` header, LinuxToys treats it as requiring systemd.

This means a script such as:

```bash
sysd_enable example.service
sysd_start example.service
```

does not normally need to manually add:

```bash
# systemd: yes
```

although explicitly stating the requirement can make the script easier for humans to understand.

---

## Wayland/X11 Compatibility

### `wayland`

For a Wayland-only script:

```bash
# wayland: yes
```

or:

```bash
# wayland: true
```

For an X11-only script:

```bash
# wayland: no
```

or:

```bash
# wayland: false
```

If this header is absent, LinuxToys assumes the feature is neutral and can run on either session type.

---

## Hybrid-GPU Compatibility

### `hybridgpu`

For a feature that should exist **only** on hybrid NVIDIA + Intel/AMD systems:

```bash
# hybridgpu: only
```

To prevent it from appearing on hybrid-GPU machines:

```bash
# hybridgpu: no
```

The compatibility implementation also supports compatibility-key filtering for hybrid systems:

```bash
# hybridgpu: gpu-intel
```

or exclusions:

```bash
# hybridgpu: !gpu-amd
```

The whitelist/blacklist is evaluated only when the machine has the `hybridgpu` compatibility key.

For simple cases, prefer `only`, `no`, or no header at all.

---

## Device-ID Compatibility

### `deviceids`

Restrict a script to machines containing particular PCI or USB IDs:

```bash
# deviceids: a69c
```

Several IDs may be supplied:

```bash
# deviceids: 8086, 7d55, 8086:7d55
```

LinuxToys normalizes IDs by removing a leading `0x` and checks them against detected PCI and USB vendor IDs, product/device IDs, and combined `vendor:product` IDs.

Thus these are equivalent forms where applicable:

```text
8086
0x8086
```

For maximum precision, use a combined vendor/product pair when one is available:

```bash
# deviceids: 1234:abcd
```

---

## Container Compatibility

### `nocontainer`

To prevent a script from running inside any container:

```bash
# nocontainer
```

or:

```bash
# nocontainer:
```

To prevent it only in containers based on selected distributions:

```bash
# nocontainer: debian, ubuntu
```

To make a feature available **only inside containers**:

```bash
# nocontainer: invert
```

It can also be limited to particular container environments:

```bash
# nocontainer: invert, debian
```

The supported behavior is:

```text
# nocontainer
    Host: allowed
    Container: blocked

# nocontainer: fedora
    Host: allowed
    Fedora container: blocked
    Other container: allowed

# nocontainer: invert
    Host: blocked
    Container: allowed

# nocontainer: invert, debian
    Host: blocked
    Debian container: allowed
    Other container: blocked
```

### Automatic Flatpak/AppImage guardrail

You do not need to manually add `nocontainer` merely because your script calls:

```bash
pkg_flat ...
pkg_appimage ...
flatpak_in_lib
```

LinuxToys automatically treats scripts using these facilities as incompatible with containers.

It also detects statically visible AppImage/Flatpak URLs passed through `pkg_fromurl`.

The automatic Flatpak/AppImage restriction takes precedence over explicit container metadata.

---

## Localization Filtering

### `localize`

A script can be restricted to selected locales:

```bash
# localize: pt
```

or:

```bash
# localize: pt, es
```

If `localize` is absent, the script is shown for all locales.

If present, the user's detected locale must be in the comma-separated list.

This header is intended for functionality that genuinely only makes sense in certain locales. It is not needed merely because the name or description requires translation.

---

## Reboot Requirement

### `reboot`

For a feature that requires a reboot after successful installation:

```bash
# reboot: yes
```

For normal scripts:

```bash
# reboot: no
```

`no` is the default.

For changes that require rebooting only on rpm-ostree/Universal Blue systems:

```bash
# reboot: ostree
```

LinuxToys interprets `ostree` as a reboot requirement only when the current compatibility set contains `ostree` or `ublue`.

---

## Reversion and Automatic Rollback

### `revert`

LinuxToys maintains a transaction map for script operations and can use that information to undo changes.

The default is:

```bash
# revert: yes
```

Omitting the header also defaults to `yes`.

### Full reversion

```bash
# revert: yes
```

Allows both:

* automatic rollback after an unsuccessful installation;
* manual removal through LinuxToys.

### No reversion

```bash
# revert: no
```

Disables both automatic and manual reversion.

Use this only when LinuxToys cannot safely determine how to undo the operation.

### Internal/manual custom removal

```bash
# revert: internal
```

Keeps automatic rollback support, but indicates that manual removal requires the script's own internal process instead of normal LinuxToys transaction-based removal.

### Conditional reversion

Reversion can be limited by compatibility keys:

```bash
# revert: ubuntu, debian
```

or excluded on specific targets:

```bash
# revert: !ostree
```

The supported modes and whitelist/blacklist syntax are defined by the compatibility layer.

For most well-behaved LinuxToys scripts, leave reversion enabled.

---

## Header Example

A fairly complete normal script header might look like:

```bash
# name: Example GPU Tool
# version: 1.2
# description: Installs and configures Example GPU Tool.
# icon: example-gpu.svg
# repo: https://github.com/example/example-gpu-tool
# compat: fedora, rhel, arch, cachy
# gpu: amd
# cpu: amd, intel
# desktop: gnome, plasma
# systemd: yes
# wayland: yes
# nocontainer
# reboot: no
# revert: yes
```

Do not add restrictions merely because they are available. A missing compatibility header normally means that dimension is unrestricted.

For example, software that works on all GPUs should have **no `gpu` header**, not:

```bash
# gpu: amd, intel, nvidia
```

---

## Automatic Library Loading

LinuxToys' Bash library is split into several modules.

The main library always loads:

```text
sysinfo.bash
```

Other modules are conditionally sourced according to automatically generated flags:

```text
FS_OPS       -> fsops.bash
PACKAGE_OPS  -> packages.bash
BOOT_OPS     -> boot.bash
MISC_OPS     -> misc.bash
SYSD_OPS     -> sysd.bash
```

The loader statically scans the script for functions belonging to those modules. It also resolves transitive library dependencies.

Therefore a script can simply contain:

```bash
pkg_install curl
prep_create /etc/example.conf
```

without manually writing:

```bash
source "$SCRIPT_DIR/libs/packages.bash"
source "$SCRIPT_DIR/libs/fsops.bash"
```

### Dynamically generated function calls

The scanner is deliberately conservative but is not a full Bash parser. If a function name is generated dynamically, the loader may not be able to infer it.

For example:

```bash
operation="pkg_install"
"$operation" example
```

For such unusual cases, the script may explicitly set the corresponding flag before sourcing `linuxtoys.bash` when executed outside the normal loader path. The loader documentation explicitly identifies this as the escape hatch for computed function names.

For normal LinuxToys scripts, direct function calls are strongly preferred.

---

## Error and Message Functions

The main library provides UI-aware messaging functions.

### `info`

```bash
info "Installation completed."
```

Displays an informational dialog when GUI interaction is available and falls back to terminal output otherwise.

The normal completion message can therefore be written as:

```bash
info "$finishmsg"
```

---

### `warn`

```bash
warn "This feature requires manual configuration."
```

Displays a warning.

---

### `error`

```bash
error "Optional component could not be configured."
```

Reports an error but returns control to the script.

Use this for a failure that should **not** terminate the entire installation.

---

### `die`

```bash
die "Failed to install Example."
```

Reports a fatal error and terminates the script with exit status `1`.

Typical usage:

```bash
some_command || die "some_command failed"
```

Legacy aliases remain available:

```text
fatal      -> die
nonfatal   -> error
zeninf     -> info
zenwrn     -> warn
zenask     -> question
sudo_rq    -> askpass sudo
zenpass    -> askpass password
```

New code should generally use the newer names except where an existing LinuxToys convention makes the legacy alias clearer.

---

## Asking the User a Question

### `question`

```bash
if question "Example Installer" \
    "Would you like to enable the optional component?"; then
    ...
fi
```

The signature is:

```bash
question TITLE TEXT [WIDTH] [HEIGHT]
```

Default dimensions are:

```text
360 x 300
```

In GUI mode the function uses Zenity. When an interactive terminal is available as a fallback, it asks for a `y/N` response.

Example:

```bash
if question "Example" "Install development tools?" 400 250; then
    pkg_install example-devel
fi
```

---

## Privilege Authentication

### `askpass`

To make sure sudo credentials are available:

```bash
askpass
```

or explicitly:

```bash
askpass sudo
```

Legacy scripts may use:

```bash
sudo_rq
```

LinuxToys first checks whether existing sudo authorization is still valid. If authentication is required, it uses the appropriate GUI or CLI password path.

In many cases developers do not need to call this directly because library helpers invoke `sudo` themselves. It is useful before a sequence of privileged commands when authentication should happen predictably before the operation begins.

Do not implement custom password dialogs or pipe arbitrary captured passwords into `sudo`.

---

## Runtime OS Detection

`sysinfo.bash` is always available.

The primary distribution helpers are:

```bash
is_arch
is_cachy
is_fedora
is_ostree
is_debian
is_ubuntu
is_suse
is_solus
is_zorin
is_rhel
is_deepin
is_manjaro
is_systemd
```

They are designed for ordinary Bash conditionals:

```bash
if is_fedora; then
    ...
elif is_arch; then
    ...
elif is_debian || is_ubuntu; then
    ...
fi
```

Prefer these helpers over repeatedly parsing `/etc/os-release`.

### Example

```bash
if is_debian || is_ubuntu; then
    pkg_install foo
elif is_fedora || is_rhel; then
    pkg_install foo
elif is_arch || is_cachy; then
    pkg_install foo
fi
```

In this particular example, the branches are unnecessary if the package has the same name everywhere:

```bash
pkg_install foo
```

Use distribution checks only when the actual procedure differs.

---

## Runtime Hardware Detection

The system-information library also exposes:

```bash
is_nvidia
is_intel
is_icr_capable
is_amd
amd_dgpu
rocm_apu
is_rocm_capable
has_rebar
is_hybridgpu
```

For example:

```bash
if is_nvidia; then
    pkg_install nvidia-settings
fi
```

or:

```bash
if is_rocm_capable; then
    ...
fi
```

Intel Compute Runtime capability is exposed through:

```bash
is_icr_capable
```

while ROCm hardware eligibility is exposed through:

```bash
is_rocm_capable
```

The library also exposes `has_rebar` for checking whether a GPU has a BAR allocation larger than the traditional 256 MiB window, and `is_hybridgpu` for NVIDIA + Intel/AMD hybrid systems.

Use headers when incompatible hardware should prevent the script from appearing at all. Use runtime detection when the script supports several hardware configurations but needs different commands for each.

---

## Package Management

One of the most important LinuxToys abstractions is the package-management library.

Avoid manually writing large blocks such as:

```bash
if is_debian; then
    sudo apt install ...
elif is_fedora; then
    sudo dnf install ...
elif is_arch; then
    sudo pacman -S ...
fi
```

when the package name is identical.

Use:

```bash
pkg_install package-name
```

instead.

---

### `pkg_install`

```bash
pkg_install curl git
```

Installs one or more native packages.

Before installing, LinuxToys checks which requested packages are already installed and skips them.

The function currently abstracts package installation across Debian/Ubuntu, Arch/CachyOS/Manjaro, Fedora/RHEL, rpm-ostree systems, openSUSE, and Solus. Successful installations are appended to the transaction map so LinuxToys can later undo them.

### Options

`pkg_install` recognizes special LinuxToys options before the package arguments.

#### `--ignore-appends`

```bash
pkg_install --ignore-appends foo
```

Installs without recording the normal package transaction.

Use sparingly. This deliberately opts out of normal revert tracking for that operation.

#### `--ostreecheck`

```bash
pkg_install --ostreecheck foo
```

On rpm-ostree systems, installation is performed and LinuxToys then handles the pending deployment state.

#### `--allowerasing`

```bash
pkg_install --allowerasing foo
```

Enables DNF's `--allowerasing` path where applicable.

---

## Checking Whether Packages Are Installed

### `pkg_exists`

```bash
pkg_exists foo bar
```

Populates two arrays:

```bash
pkg_found
pkg_notfound
```

Example:

```bash
pkg_exists foo

if [[ ${#pkg_found[@]} -gt 0 ]]; then
    echo "foo is installed"
fi
```

For ordinary installation code, calling `pkg_exists` yourself is usually unnecessary because `pkg_install` already performs this check.

---

## Removing Native Packages

### `pkg_remove`

```bash
pkg_remove foo bar
```

Removes installed packages using the native package manager.

A compatibility alias also exists:

```bash
pkg_rm foo
```

Most installation scripts should not manually remove their own packages during normal LinuxToys uninstallation; package installations recorded in the transaction map can be handled by the revert system.

`pkg_remove` is useful when package removal is itself part of the installation procedure.

---

## Flatpak Installation

### `pkg_flat`

```bash
pkg_flat org.example.Application
```

Ensures Flatpak/Flathub support exists, installs or updates the specified application, checks the result, and records newly installed Flatpaks in the transaction map.

By default LinuxToys prefers the user's Flatpak installation where appropriate.

To request the system installation path:

```bash
pkg_flat --skip-user org.example.Application
```

Because Flatpak applications cannot be installed from inside the containers LinuxToys targets, the use of `pkg_flat` automatically marks the script as container-incompatible.

---

## Installing a Downloaded Package

### `pkg_fromfile`

```bash
pkg_fromfile ./example.deb
```

or:

```bash
pkg_fromfile ./example.rpm
```

Installs a local package using the appropriate package manager for the current distribution.

The helper also handles `.flatpak` files.

This is useful when the script downloads or builds a package before installing it.

---

## Installing Directly from a URL

### `pkg_fromurl`

```bash
pkg_fromurl \
    "https://example.org/releases/example-amd64.deb"
```

Multiple URLs are accepted:

```bash
pkg_fromurl \
    "https://example.org/foo.deb" \
    "https://example.org/foo-data.deb"
```

The function creates a temporary download directory, downloads each URL with `curl`, then passes the resulting file to either `pkg_appimage` or `pkg_fromfile` depending on its type.

This is generally preferable to manually combining:

```bash
wget ...
sudo dpkg -i ...
```

because LinuxToys retains its normal package-management and transaction behavior.

---

## Installing the Latest GitHub/Codeberg Release

### `pkg_fromrelease`

For projects that publish installable packages in GitHub or Codeberg releases:

```bash
pkg_fromrelease \
    "https://github.com/example/example"
```

An optional asset glob can narrow the result:

```bash
pkg_fromrelease \
    "https://github.com/example/example" \
    "*desktop*"
```

The helper queries the latest stable release, detects the machine architecture and native package format, filters unsuitable assets, and selects an installable release package before passing it to `pkg_fromurl`.

Recognized release formats include AppImage, Flatpak, and the appropriate native package format.

On x86-64, 32-bit x86 release assets may be used as a fallback only when a suitable 64-bit or architecture-neutral candidate does not exist.

---

## AppImage Integration

### `pkg_appimage`

```bash
pkg_appimage ./Example.AppImage
```

Integrates one or more AppImages into the user's system and records them for reversion.

The function also contains update handling: if the current LinuxToys registry indicates that this script previously installed an AppImage, LinuxToys resolves the previous integrated filename and removes it before integrating its replacement.

On systemd systems, LinuxToys currently performs AppImage integration through Gear Lever. On other systems it uses its manual integration path.

AppImage installers are automatically excluded from container environments.

---

## npm and Bun Packages

### `pkg_npm`

```bash
pkg_npm package-name
```

Installs npm if required, configures the global user path where necessary, skips packages already globally installed, and records installed npm packages.

---

### `pkg_bun`

```bash
pkg_bun package-name
```

Ensures Bun exists or updates it, configures the relevant user PATH, installs missing global packages, and records them.

---

## Filesystem Operations and Reversion

For files that LinuxToys may need to restore later, use the filesystem preparation helpers rather than editing them blindly.

These functions are central to automatic rollback.

---

### `prep_create`

Before creating a new file:

```bash
prep_create /etc/example/example.conf

sudo tee /etc/example/example.conf >/dev/null <<'EOF'
option=true
EOF
```

`prep_create`:

1. creates missing parent directories if required;
2. creates the file;
3. records:

```text
created /etc/example/example.conf
```

in the transaction map.

If the file unexpectedly already exists, LinuxToys switches to `prep_edit` rather than destroying the original.

---

## Preparing an Existing File for Modification

### `prep_edit`

```bash
prep_edit /etc/example/example.conf

sudo sed -i 's/foo/bar/' /etc/example/example.conf
```

Before editing, LinuxToys makes a `.bak` copy and records that the file was edited.

If the expected file does not exist, it records a warning and falls back to treating the file as newly created.

This is preferable to:

```bash
sudo cp /etc/example/example.conf /etc/example/example.conf.bak
```

because the transaction system understands `prep_edit`.

---

## Preparing a File or Directory for Removal

### `prep_rm`

```bash
prep_rm /etc/example/obsolete.conf
```

Rather than immediately deleting the target, LinuxToys moves it to a `.bak` path and records the removal.

This makes the operation reversible.

---

## Creating Directories

### `prep_dir`

```bash
prep_dir /etc/example
```

Creates the directory only if it does not already exist and records newly created directories.

---

### `prep_dir_edit`

Before modifying an existing directory as a unit:

```bash
prep_dir_edit /etc/example
```

LinuxToys backs it up as:

```text
/etc/example.bak
```

and marks it as edited.

---

## Temporary Directories

### `prep_tmp`

```bash
prep_tmp
```

Selects and enters LinuxToys' temporary working directory, falling back through the configured temporary location, `/tmp/linuxtoys`, and finally:

```text
$HOME/.cache/linuxtoys/tmp
```

---

### `prep_tmp_noram`

```bash
prep_tmp_noram
```

Explicitly uses:

```text
$HOME/.cache/linuxtoys/tmp
```

This is useful for larger downloads/builds where RAM-backed temporary storage would be inappropriate.

---

## Privilege-Aware Copy and Move Helpers

### `copy_`

```bash
copy_ source destination
```

or:

```bash
copy_ -r source-directory destination
```

LinuxToys first attempts the copy as the current user and retries through `sudo` if necessary. A missing source or failed copy is treated as fatal.

---

### `move_`

```bash
move_ source destination
```

Works similarly for moves, with automatic sudo fallback.

---

## systemd Helpers

When a script needs to manage systemd services, use the `sysd_*` functions.

### System services

```bash
sysd_enable example.service
sysd_start example.service
```

Available operations:

```text
sysd_enable
sysd_disable
sysd_start
sysd_stop
```

LinuxToys performs a daemon reload where needed and records each operation in the transaction map.

Multiple services can be supplied:

```bash
sysd_enable example.service example.timer
sysd_start example.service example.timer
```

---

### User systemd Services

For user services:

```bash
sysd_enable_usr example.service
sysd_start_usr example.service
```

Available functions are:

```text
sysd_enable_usr
sysd_disable_usr
sysd_start_usr
sysd_stop_usr
```

These invoke:

```text
systemctl --user
```

rather than the system service manager.

---

## Bootloader and initramfs Management

### `bootloader_upd`

```bash
bootloader_upd
```

Updates the installed bootloader using the method appropriate for the current distribution.

LinuxToys currently handles GRUB variants and, where applicable, Limine, systemd-boot, `sdboot-manage`, and Solus' `clr-boot-manager`. rpm-ostree systems skip the normal bootloader regeneration path.

Use this instead of embedding distribution-specific bootloader commands throughout a script.

---

### `initramfs_upd`

```bash
initramfs_upd
```

Regenerates initramfs images using the appropriate implementation:

```text
Debian/Ubuntu      update-initramfs
Arch/CachyOS       dracut or mkinitcpio
Fedora/RHEL/SUSE   dracut
```

The operation is recorded in the transaction map.

---

## Kernel Arguments

### `kargs_upd`

For rpm-ostree systems:

```bash
kargs_upd "example.option=1"
```

Multiple arguments may be passed:

```bash
kargs_upd \
    "example.option=1" \
    "another.option=2"
```

Each is added with `rpm-ostree kargs --append` and recorded.

---

### `grubbyargs_upd`

```bash
grubbyargs_upd "example.option=1"
```

Ensures `grubby` exists and adds each argument to all kernels.

---

## Secure Boot

### `secureboot_check`

```bash
secureboot_check
```

Checks whether Secure Boot is active and performs LinuxToys' appropriate preparation for supported distributions.

The implementation currently includes Fedora/RHEL/rpm-ostree, Ubuntu and Debian paths. Fedora-family systems can delegate to LinuxToys' module-signing script, while Ubuntu/Debian paths work with MOK enrollment.

For Debian procedures intended to use the Ubuntu-style MOK location:

```bash
secureboot_check --ubuntumok
```

Use this helper before installing unsigned kernel modules rather than reimplementing the LinuxToys Secure Boot flow inside every driver installer.

---

## Miscellaneous Transaction Helpers

Some operations do not fit the package/filesystem model but still need to be recorded.

### `shell_change`

```bash
shell_change /bin/zsh
```

Changes the current user's shell and records the operation.

---

### `distrobox_created`

```bash
distrobox_created example-container
```

Records a Distrobox creation in the transaction map.

The actual container creation is performed separately.

---

### `swapfile_created`

```bash
swapfile_created /swapfile
```

Records a created swapfile for reversion.

---

### `rclone_mount`

```bash
rclone_mount myremote /mnt/example
```

Mounts an rclone remote as a daemon and records the resulting mount.

---

## Flatpak Overrides

### `flatpak_override`

Example:

```bash
flatpak_override user fs "$HOME/Games" org.example.Application
```

General form:

```text
flatpak_override SCOPE TYPE SETTING APPLICATION
```

Scope:

```text
user
system
```

Accepted type aliases include:

```text
fs      -> filesystem
name    -> talk-name
dbus    -> talk-dbus
```

and the direct types:

```text
share
env
runtime
device
socket
filesystem
talk-name
talk-dbus
```

The function checks that the target Flatpak exists and records the override for reversion.

---

## Calling Another LinuxToys Script

### `call_script`

LinuxToys scripts can reuse other LinuxToys scripts:

```bash
call_script rpmfusion
```

The `.sh` suffix is optional:

```bash
call_script rpmfusion.sh
```

LinuxToys looks for the named script in its synchronized script cache, verifies that it is compatible with the current machine, and only then executes it. An incompatible child is skipped with return status `2`.

Arguments can be forwarded:

```bash
call_script child-script --some-option value
```

### Child transaction behavior

Called scripts receive their own transaction map.

If the child succeeds:

1. its transaction is committed independently;
2. the child is saved in the registry;
3. the parent transaction records that the child was called.

If the child fails, its uncommitted transaction operations are merged into the parent's transaction so normal top-level rollback can undo those changes.

This allows complex functionality to be decomposed into reusable LinuxToys features without losing rollback safety.

Prefer:

```bash
call_script paru
```

over duplicating another LinuxToys script's complete installation logic.

---

## Optional Helper Library

Some less fundamental utilities are kept in the additional helper library.

Load it with:

```bash
summon_helpers
```

LinuxToys then sources its helper library.

Currently exposed utilities include:

```text
fetch_from_mirror
multilib_chk
clinfo_chk
enable_debian_nonfree
enable_debian_backports
```

as well as legacy compatibility wrappers including:

```text
chaotic_aur_lib
rpmfusion_chk
pip_lib
flatpak_in_lib
```

The legacy wrappers delegate to normal LinuxToys scripts through `call_script`.

### Example

```bash
summon_helpers

if is_debian; then
    enable_debian_nonfree
fi
```

---

## Optimizer Helper Library

Specialized optimization helpers may be loaded with:

```bash
summon_optimizers
```

The current optimizer library mostly provides reusable calls into existing LinuxToys optimization scripts, plus specialized functionality such as NVIDIA CDI setup.

Examples of currently exposed functions include:

```text
nvidia_ctkpatch
cachyos_sysd_lib
sboost_lib
preempt_lib
dsplitm_lib
psave_lib
earlyoom_lib
zswap_lib
wayland_proton_lib
intel_xe_lib
free_mem_fix
dnsmasq_lib
fix_intel_gtk
pp_ondemand
```

Most legacy optimizer wrappers now simply call the corresponding LinuxToys script.

For new code, prefer `call_script` directly when no additional abstraction is needed.

---

## The Transaction Map

LinuxToys' reversion model revolves around the transaction map.

A transaction map is initialized lazily when something first needs to record an operation. LinuxToys creates the file with mode `600` and appends operations through `_append_transmap`.

Developers normally should **not** manually write to the transaction map.

Instead, use transaction-aware functions such as:

```text
pkg_install
pkg_flat
pkg_fromfile
pkg_appimage

prep_create
prep_edit
prep_rm
prep_dir
prep_dir_edit

sysd_enable
sysd_start

bootloader_upd
initramfs_upd

shell_change
flatpak_override
```

These functions both perform the operation and tell LinuxToys how the system changed.

This is one of the most important rules when developing a full LinuxToys script:

> If LinuxToys already provides a transaction-aware helper for an operation, prefer it over the equivalent raw shell command.

For example, prefer:

```bash
prep_edit /etc/example.conf
sudo sed -i 's/foo/bar/' /etc/example.conf
```

over:

```bash
sudo cp /etc/example.conf /etc/example.conf.bak
sudo sed -i 's/foo/bar/' /etc/example.conf
```

and prefer:

```bash
pkg_install foo
```

over:

```bash
sudo dnf install -y foo
```

Doing so makes error recovery and later uninstallation substantially more reliable.

---

## Error Handling and Rollback-Friendly Scripting

Commands whose failure means that installation cannot safely continue should terminate through `die`/`fatal`.

Recommended:

```bash
some_command || die "Failed to configure Example."
```

Not recommended:

```bash
some_command
```

when the remainder of the script assumes the command succeeded.

Likewise, do not hide failures unnecessarily:

```bash
some_command || true
```

unless failure is genuinely acceptable.

A good LinuxToys script should leave the transaction log in a state that accurately represents everything successfully changed before a failure occurs.

---

## Example: Cross-Distribution Application Installer

```bash
# name: Example Server
# version: 1.0
# description: Installs and configures Example Server.
# icon: example-server.svg
# repo: https://github.com/example/example-server
# compat: debian, ubuntu, fedora, rhel, arch, cachy, suse
# systemd: yes
# nocontainer
# reboot: no
# revert: yes

# Install common dependencies.
pkg_install curl

# Package names differ between families.
if is_debian || is_ubuntu; then
    pkg_install example-server example-utils

elif is_fedora || is_rhel; then
    pkg_install example-server example-tools

elif is_arch || is_cachy; then
    pkg_install example-server example-tools

elif is_suse; then
    pkg_install example-server
fi

# Preserve an existing configuration or register a new one.
config="/etc/example/example.conf"

if [[ -f "$config" ]]; then
    prep_edit "$config"
else
    prep_create "$config"
fi

sudo tee "$config" >/dev/null <<'EOF'
enabled=true
port=1234
EOF

# Enable and start the service using transaction-aware helpers.
sysd_enable example.service
sysd_start example.service

info "$finishmsg"
```

This script automatically causes LinuxToys to load the package, filesystem and systemd modules because those functions are referenced directly.

---

## Example: Latest Upstream Release

For a project whose upstream publishes ready-made packages:

```bash
# name: Example Desktop
# version: 1.0
# description: Installs the latest stable version of Example Desktop.
# icon: example.svg
# repo: https://github.com/example/example
# compat: debian, ubuntu, fedora, rhel, arch, cachy
# reboot: no
# revert: yes

pkg_fromrelease "https://github.com/example/example"

info "$finishmsg"
```

LinuxToys performs distribution-format and architecture selection automatically.

If the release has several similarly valid assets:

```bash
pkg_fromrelease \
    "https://github.com/example/example" \
    "*desktop*"
```

can narrow the selection.

---

## Example: Driver Installer

```bash
# name: Example Kernel Driver
# version: 1.0
# description: Installs the Example hardware driver.
# icon: driver.svg
# compat: debian, ubuntu, fedora, rhel, arch, cachy
# deviceids: 1234:abcd
# systemd: yes
# nocontainer
# reboot: yes
# revert: yes

secureboot_check

if is_debian || is_ubuntu; then
    pkg_install dkms linux-headers-amd64
elif is_fedora || is_rhel; then
    pkg_install dkms kernel-devel
elif is_arch || is_cachy; then
    pkg_install dkms linux-headers
fi

prep_tmp_noram

git clone https://github.com/example/example-driver.git \
    || die "Failed to download Example driver."

cd example-driver \
    || die "Failed to enter driver source directory."

sudo ./install.sh \
    || die "Failed to install Example driver."

initramfs_upd

info "$rebootmsg"
```

The exact package names in a real driver script should of course reflect the distributions actually supported by that driver.

---

## Choosing Headers vs Runtime Checks

Use a **header** when incompatible machines should never be offered the feature:

```bash
# gpu: nvidia
```

Use a **runtime function** when the script supports multiple systems but needs different behavior:

```bash
if is_nvidia; then
    ...
elif is_amd; then
    ...
fi
```

A useful rule is:

```text
Header        -> Can this machine use the feature at all?
Runtime check -> How should the feature be installed on this machine?
```

For example:

```bash
# compat: debian, ubuntu, fedora, arch
```

combined with:

```bash
if is_debian || is_ubuntu; then
    ...
elif is_fedora; then
    ...
elif is_arch; then
    ...
fi
```

is appropriate when the software supports all four families but requires different procedures.

---

## Recommended Practices

When writing a LinuxToys script:

1. **Keep all headers at the top of the file.** Header parsing stops at the first non-comment line.

2. **Use compatibility headers instead of manually aborting unsupported systems.** Users should not be offered a feature that LinuxToys already knows is incompatible.

3. **Use `pkg_install` instead of calling package managers directly** whenever package names permit it.

4. **Use `prep_create`, `prep_edit`, `prep_rm`, `prep_dir`, and `prep_dir_edit` for persistent filesystem changes.** These make rollback possible.

5. **Use `sysd_*` helpers for service changes.**

6. **Use `bootloader_upd` and `initramfs_upd` instead of duplicating distribution-specific boot logic.**

7. **Use `secureboot_check` for driver/module installers that need Secure Boot handling.**

8. **Use `call_script` when LinuxToys already implements a prerequisite as another script.** Do not copy its installer into yours.

9. **Use `die` for fatal failures.** A script should not continue after a prerequisite operation has failed.

10. **Leave `revert` enabled whenever possible.** LinuxToys' transaction-aware functions are designed around reversible installation.

11. **Do not add unnecessary compatibility restrictions.** The absence of a GPU, CPU, desktop, Wayland or similar header means that dimension is unrestricted.

12. **Do not manually source split core libraries.** Normal scripts are inspected by `library_loader.py` and receive the modules they use automatically.

---

## Core Function Quick Reference

| Area                       | Functions                                                                                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Messages                   | `info`, `warn`, `error`, `die`, `question`                                                                                                                       |
| Authentication             | `askpass`, `sudo_rq`                                                                                                                                             |
| OS detection               | `is_arch`, `is_cachy`, `is_fedora`, `is_ostree`, `is_debian`, `is_ubuntu`, `is_suse`, `is_solus`, `is_zorin`, `is_rhel`, `is_deepin`, `is_manjaro`, `is_systemd` |
| Hardware                   | `is_nvidia`, `is_intel`, `is_icr_capable`, `is_amd`, `amd_dgpu`, `rocm_apu`, `is_rocm_capable`, `has_rebar`, `is_hybridgpu`                                      |
| Native packages            | `pkg_exists`, `pkg_install`, `pkg_remove`, `pkg_rm`                                                                                                              |
| Portable/external packages | `pkg_flat`, `pkg_fromfile`, `pkg_fromurl`, `pkg_fromrelease`, `pkg_appimage`, `pkg_appimage_rm`                                                                  |
| Language package managers  | `pkg_npm`, `pkg_bun`                                                                                                                                             |
| Filesystem                 | `prep_create`, `prep_edit`, `prep_rm`, `prep_dir`, `prep_dir_edit`, `prep_tmp`, `prep_tmp_noram`, `copy_`, `move_`                                               |
| systemd                    | `sysd_enable`, `sysd_disable`, `sysd_start`, `sysd_stop`, `sysd_enable_usr`, `sysd_disable_usr`, `sysd_start_usr`, `sysd_stop_usr`                               |
| Boot                       | `bootloader_upd`, `initramfs_upd`, `kargs_upd`, `grubbyargs_upd`, `secureboot_check`                                                                             |
| Miscellaneous              | `shell_change`, `distrobox_created`, `rclone_mount`, `swapfile_created`, `flatpak_override`                                                                      |
| Composition                | `call_script`                                                                                                                                                    |
| Optional libraries         | `summon_helpers`, `summon_optimizers`                                                                                                                            |

---

## Full Header Quick Reference

| Header           | Purpose                    | Example                            |
| ---------------- | -------------------------- | ---------------------------------- |
| `name`           | User-facing feature name   | `# name: Example`                  |
| `version`        | Script metadata version    | `# version: 1.0`                   |
| `description`    | User-facing description    | `# description: Installs Example.` |
| `icon`           | Feature icon               | `# icon: example.svg`              |
| `repo`           | Upstream repository        | `# repo: https://github.com/...`   |
| `compat`         | OS compatibility           | `# compat: fedora, arch, !manjaro` |
| `gpu`            | GPU compatibility          | `# gpu: amd, intel`                |
| `cpu`            | CPU compatibility          | `# cpu: amd`                       |
| `desktop`        | Desktop compatibility      | `# desktop: gnome, plasma`         |
| `systemd`        | Init-system requirement    | `# systemd: yes`                   |
| `wayland`        | Session requirement        | `# wayland: yes`                   |
| `hybridgpu`      | Hybrid GPU filtering       | `# hybridgpu: only`                |
| `deviceids`      | PCI/USB hardware filtering | `# deviceids: 1234:abcd`           |
| `nocontainer`    | Container filtering        | `# nocontainer: fedora`            |
| `localize`       | Locale filtering           | `# localize: pt, es`               |
| `reboot`         | Reboot requirement         | `# reboot: yes`                    |
| `revert`         | Reversion policy           | `# revert: yes`                    |
| `negates`        | Hide superseded scripts    | `# negates: old-script`            |
| `optimized-only` | Optimization-set filtering | `# optimized-only:`                |

---

## Final Minimal Template

For a new full LinuxToys installer, this is a good starting point:

```bash
# name: Application Name
# version: 1.0
# description: Installs and configures Application Name.
# icon: application.svg
# repo: https://github.com/vendor/application
# compat: debian, ubuntu, fedora, arch
# reboot: no
# revert: yes

# Install dependencies/packages.
pkg_install package-name

# Persistent files should be registered before modification.
config="/etc/application/application.conf"

if [[ -f "$config" ]]; then
    prep_edit "$config"
else
    prep_create "$config"
fi

# Perform configuration.
sudo tee "$config" >/dev/null <<'EOF'
example=true
EOF

info "$finishmsg"
```

From there, add only the compatibility headers and library functions genuinely required by the feature.

The central design principle is simple:

> **Let LinuxToys handle portability, compatibility and transaction tracking wherever an existing helper can do so. Keep the script focused on the software-specific procedure.**
