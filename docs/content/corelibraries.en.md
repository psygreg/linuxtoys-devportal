# LinuxToys Core Libraries

LinuxToys procedure scripts are intentionally small. Common work such as package management, filesystem changes, service control, privilege escalation, system detection, dialogs, and transaction tracking is provided by the LinuxToys shell libraries. It also leverages its Core Libraries for standard installation procedures like *AppStream* apps - that's what makes it work even on *Arch Linux* and derivatives, which don't play well with *PackageKit*.

**You do not need to source these libraries manually.**

When LinuxToys launches a procedure, its library loader inspects the script before execution, detects the library functions it uses, follows dependencies between libraries, sets the required library flags, and sources the LinuxToys core automatically. `sysinfo.bash` is always available; the other modules are loaded only when required.

This makes a normal procedure look like this:

```bash
pkg_install curl git
prep_create "$HOME/.config/example/config"
info "$finishmsg"
```

There is no `source`, import block, or library initialization boilerplate in the script.

## How libraries are loaded

Before a procedure is executed, LinuxToys performs conservative static detection of the function names referenced by the script. It also inspects statically referenced `run_list_hook` scripts and follows transitive dependencies between the core libraries.

The loader maps the optional modules to internal flags:

| Library | Internal flag |
| --- | --- |
| `fsops.bash` | `FS_OPS` |
| `packages.bash` | `PACKAGE_OPS` |
| `boot.bash` | `BOOT_OPS` |
| `misc.bash` | `MISC_OPS` |
| `sysd.bash` | `SYSD_OPS` |
| `helpers.bash` | `HELPERS_OPS` |
| `optimizers.bash` | `OPTIMIZER_OPS` |

It then sources `linuxtoys.bash`, which always loads `sysinfo.bash` and conditionally loads the selected modules. The systemd library is loaded only when systemd is actually in use.

The detection intentionally errs toward loading too much rather than too little. Computed function names cannot be inferred statically; in the unusual case where a script dynamically constructs a library function name, the appropriate `*_OPS` flag can still be set explicitly before the core is sourced.

The old `summon_helpers` and `summon_optimizers` calls remain supported for compatibility, but new scripts should simply call the functions they need.

## The core library

`linuxtoys.bash` provides functionality shared by essentially every procedure: localized messages, dialogs, privilege escalation, nested script execution, runner integration, and transaction tracking.

### Privilege escalation

Use:

```bash
sudo_ command arguments...
```

instead of calling `sudo` directly.

`sudo_` validates authentication immediately before privilege escalation and uses LinuxToys' graphical or terminal authentication flow as appropriate.

Usually you should not call `askpass` yourself. Library helpers such as package, filesystem, boot and service operations already use `sudo_` when elevation is required.

### Information, warnings and errors

```bash
info "Operation completed."
warn "This configuration is unusual."
error "The optional operation failed."
die "The installation cannot continue."
```

`info` and `warn` display a graphical dialog when appropriate and fall back to terminal output. `error` reports a non-fatal error and returns failure. `die` reports a fatal error and terminates the procedure.

The legacy aliases `zeninf`, `zenwrn`, `nonfatal`, and `fatal` remain available, but new code should prefer the current names.

### Questions

```bash
if question "$msg006" "Proceed with installation?"; then
    # accepted
else
    # cancelled
fi
```

The arguments are:

```text
question TITLE TEXT [WIDTH] [HEIGHT]
```

The default size is 360×300. When graphical dialogs are unavailable, LinuxToys automatically falls back to a terminal `[y/N]` prompt.

### Single-choice selection

```bash
choice=$(radioselect "Option A" "Option B" "Option C") || exit 1
```

`radioselect` presents a radio-list dialog and prints the selected value to stdout. In terminal mode, the same operation becomes a numbered selection.

### Multiple-choice selection

```bash
mapfile -t choices < <(
    listselect "Feature A" "Feature B" "Feature C"
)
```

`listselect` allows multiple selections and prints one selected value per line.

### Calling another LinuxToys procedure

```bash
call_script flathub
```

Use `call_script` when an existing LinuxToys procedure should be reused rather than duplicated.

Nested scripts are launched through the same library-loading machinery, so their own dependencies are detected automatically.

### List hooks

Repository-list hooks can be invoked with:

```bash
run_list_hook path/to/hook.sh
```

Statically referenced hooks are also inspected by the library loader, so functions used by those hooks participate in automatic dependency detection.

## Localized reusable message variables

LinuxToys loads the language library automatically according to the current locale. Procedure scripts can therefore reuse common translated message variables instead of embedding English strings for generic dialogs.

Prefer these variables when the meaning matches. They keep common interactions consistent and already localized across the LinuxToys language libraries.

| Variable | English meaning | Typical use |
| --- | --- | --- |
| `$msg006` | `Installer` | Generic installer/question dialog title |
| `$finishmsg` | `Operations completed.` | Generic successful completion |
| `$msg022` | `Reboot to take effect.` | Short reboot notice |
| `$rebootmsg` | `Installation complete. Reboot for changes to take effect.` | Successful installation requiring reboot |
| `$msg024` | `Repository enabled successfully.` | Repository setup completion |
| `$cancelmsg` | `Cancel` | Cancel button/choice text |
| `$incompatmsg` | `Your operating system is not compatible.` | Generic OS incompatibility |
| `$abortmsg` | `Operation cancelled by the user.` | User cancellation |
| `$msg276` | `Understood.` | Generic acknowledgement |
| `$msg277` | `Select versions to install` | Version-selection prompt |
| `$notdomsg` | `Nothing to do.` | Successful no-op |
| `$rmmsg` | `You already have $LT_PROGRAM installed. Do you wish to remove it?` | Generic removal prompt after setting `LT_PROGRAM` |
| `$outofspace` | `Insufficient available storage space for operation.` | Storage-space failure |
| `$hwincompat` | `Your hardware is not compatible with this feature. Operation cancelled.` | Generic hardware incompatibility |
| `$parumsg` | Message explaining that `paru` is required and will be installed | AUR helper setup |
| `$gearlevermsg` | Message explaining that Gear Lever is required and will be installed | AppImage integration setup |

For example:

```bash
if ! is_ubuntu; then
    die "$incompatmsg"
fi

if question "$msg006" "This feature changes the boot configuration. Proceed?"; then
    # operation
else
    info "$abortmsg"
    exit 0
fi

info "$finishmsg"
```

Do not reuse a message merely because it is convenient if its wording does not accurately describe the operation. Application-specific or highly contextual messages should still have their own translation entry.

## System information and compatibility

`sysinfo.bash` is always loaded. Its helpers can therefore be used without triggering an optional module.

### Distribution detection

```bash
is_arch
is_cachy
is_manjaro
is_fedora
is_ostree
is_debian
is_ubuntu
is_suse
is_solus
is_zorin
is_rhel
is_deepin
is_steamos
```

These are shell predicates:

```bash
if is_fedora || is_ostree; then
    # Fedora-family handling
fi
```

Use these helpers instead of repeatedly parsing `/etc/os-release` yourself.

### Init system

```bash
is_systemd
```

### GPU and hardware helpers

```bash
is_nvidia
is_intel
is_amd
is_hybridgpu
is_icr_capable
is_rocm_capable
has_rebar
```

These helpers encapsulate LinuxToys' current hardware-detection rules.

## Filesystem operations

Filesystem helpers integrate changes with LinuxToys' transaction map so that supported operations can be reverted automatically.

### Editing an existing file

```bash
prep_edit /etc/example.conf
sudo_ sed -i 's/old/new/' /etc/example.conf
```

`prep_edit` preserves the previous file state before modification.

### Creating a file

```bash
prep_create /etc/example.conf
```

If the target unexpectedly already exists, LinuxToys treats it as an edit instead of silently overwriting untracked data.

### Removing a file or directory

```bash
prep_rm /etc/example.conf
```

The target is moved to a backup and recorded for reversion.

### Directories

```bash
prep_dir /etc/example
prep_dir_edit /etc/example
```

`prep_dir` creates a missing directory and records it. `prep_dir_edit` preserves an existing directory before modification.

### Temporary working directory

```bash
prep_tmp
```

This selects LinuxToys' preferred temporary workspace and changes into it.

For operations that should avoid the normal temporary location:

```bash
prep_tmp_noram
```

### Copying and moving

```bash
copy_ source destination
copy_ -r directory destination
move_ source destination
```

These helpers attempt the operation normally first and escalate only when necessary.

## Package management

The package library abstracts distribution-specific package managers and records supported package operations for reversion.

### Native packages

```bash
pkg_install curl git
```

LinuxToys chooses the appropriate native package manager for the current distribution and skips packages that are already installed.

Useful options include:

```bash
pkg_install --no-recommends package
pkg_install --allowerasing package
pkg_install --ostreecheck package
pkg_install --ignore-appends package
pkg_install --bypass package
```

`--no-recommends` maps to APT's no-recommends behavior where applicable. `--bypass` disables runner locking for that transaction and is intended for operations that must remain interactive.

Remove native packages with:

```bash
pkg_remove package
```

`pkg_rm` remains as an alias.

### Flatpak

```bash
pkg_flat org.example.App
```

Use the package helpers rather than invoking Flatpak installation directly when you want LinuxToys to manage scope, prerequisites and transaction tracking.

### Package files

```bash
pkg_fromfile ./package.deb
pkg_fromfile ./package.rpm
pkg_fromfile ./package.pkg.tar.zst
pkg_fromfile ./application.flatpak
```

For a system-scope Flatpak file:

```bash
pkg_fromfile --skip-user ./application.flatpak
```

### Direct URLs

```bash
pkg_fromurl "https://example.com/package.deb"
```

Multiple URLs can be supplied as fallbacks. Portable modes include:

```bash
pkg_fromurl --tar "https://example.com/application.tar.gz"
pkg_fromurl --bin "https://example.com/application"
```

### Tarballs and standalone binaries

```bash
pkg_tarball archive.tar.gz
pkg_binary ./application
```

These install into LinuxToys' user-level application area and participate in the package helpers' managed flow.

### Building from source

```bash
pkg_make "https://example.com/project.git"
```

A custom installation command can be supplied:

```bash
pkg_make --command "make install-user" "https://example.com/project.git"
```

`pkg_make` also supports release tarballs, direct tarball URLs, declared build dependencies and uninstall mode.

### Release-based applications

```bash
pkg_fromrelease "https://github.com/example/project"
```

This is the high-level helper for selecting and installing suitable release assets from supported Git hosting.

### AppImages

```bash
pkg_appimage ./Application.AppImage
```

Removal is available through:

```bash
pkg_appimage_rm ...
```

AppImage integration uses LinuxToys' managed AppImage flow rather than treating the file as an arbitrary executable.

### JavaScript package managers

```bash
pkg_npm package-name
pkg_bun package-name
```

Use these helpers when the software is intentionally distributed through the corresponding ecosystem.

## systemd operations

The systemd module is loaded only on systemd systems when one of its functions is required.

System services:

```bash
sysd_enable example.service
sysd_start example.service
sysd_stop example.service
sysd_disable example.service
```

User services:

```bash
sysd_enable_usr example.service
sysd_start_usr example.service
sysd_stop_usr example.service
sysd_disable_usr example.service
```

These helpers perform the required daemon reload where appropriate and record service changes in the transaction map.

## Boot and initramfs operations

### Update the bootloader

```bash
bootloader_upd
```

LinuxToys chooses the appropriate supported bootloader update path for the current distribution and environment.

### Update initramfs

```bash
initramfs_upd
```

This selects `update-initramfs`, `mkinitcpio`, or `dracut` as appropriate.

### Kernel arguments

For rpm-ostree systems:

```bash
kargs_upd "argument=value"
```

For systems using `grubby`:

```bash
grubbyargs_upd "argument=value"
```

### Secure Boot

```bash
secureboot_check
```

An Ubuntu-like MOK flow on Debian can be requested with:

```bash
secureboot_check --ubuntumok
```

## Miscellaneous helpers

### Flatpak permission overrides

```bash
flatpak_override user filesystem xdg-download org.example.App
```

The arguments are:

```text
flatpak_override SCOPE TYPE SETTING TARGET
```

Supported scopes are `user` and `system`. Shorthands `fs`, `name`, and `dbus` map to filesystem, talk-name, and talk-dbus respectively.

### Add an executable to the user's PATH

```bash
path_link /path/to/application
```

LinuxToys creates a managed link in `~/.local/bin` and ensures the directory is available in the user's shell environment.

For repository applications, the LinuxToys application ID can be used as the command name:

```bash
path_link --useappid /path/to/application
```

### Desktop shortcut

```bash
desktop_shortcut "/path/to/application"
```

The helper uses the application metadata supplied by LinuxToys to create the desktop entry.

An explicit WM class can be supplied:

```bash
desktop_shortcut --class ExampleApp "/path/to/application"
```

### Change the user's shell

```bash
shell_change /usr/bin/zsh
```

### Transaction markers for special resources

Some external resources have lightweight helpers used to record their creation:

```bash
distrobox_created container-name
swapfile_created /swapfile
rclone_mount remote /mount/point
```

## Helpers library

`helpers.bash` contains reusable setup and capability helpers that do not belong to the more general modules.

Current helpers include:

```bash
fetch_from_mirror
multilib_chk
clinfo_chk
enable_debian_nonfree
enable_debian_backports
```

There are also compatibility wrappers such as `rpmfusion_chk`, `pip_lib`, and `flatpak_in_lib` which delegate to the corresponding LinuxToys procedures.

## Optimizers library

`optimizers.bash` contains specialized system-optimization integrations. Unlike the generic modules above, these helpers are relevant only to specific procedures.

The current native helper is:

```bash
nvidia_ctkpatch
```

The library also retains compatibility wrappers that delegate to LinuxToys procedures, including:

```bash
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

For new code, prefer `call_script` with the underlying procedure when that expresses the dependency more clearly.

## Transaction tracking and reversion

A major reason to use the core libraries instead of raw shell commands is transaction tracking.

Helpers such as `prep_edit`, `prep_create`, `pkg_install`, service helpers, boot helpers, and other managed operations append records to LinuxToys' transaction map. If the procedure fails, or if the user later removes a tracked feature through LinuxToys, those records can be used by the reversion system.

As a general rule:

- use `prep_edit` before modifying an existing file;
- use `prep_create` before creating a file;
- use `prep_rm` instead of permanently deleting a tracked target;
- use package helpers instead of calling package managers directly;
- use `sysd_*` helpers instead of raw `systemctl` for tracked service changes;
- use `sudo_` instead of raw `sudo`;
- use existing LinuxToys helpers whenever they already model the operation you need.

This is not just convenience: it is how custom LinuxToys procedures participate in the same managed installation and removal experience as the rest of the application.

## Minimal procedure example

```bash
# No library imports are necessary.

if ! is_debian && ! is_ubuntu; then
    die "$incompatmsg"
fi

if ! question "$msg006" "Install Example Service?"; then
    info "$abortmsg"
    exit 0
fi

pkg_install example-package

prep_create /etc/example/example.conf
sudo_ tee /etc/example/example.conf >/dev/null <<'EOF'
enabled=true
EOF

sysd_enable example.service
sysd_start example.service

info "$finishmsg"
```

The loader sees the referenced functions and automatically makes the required core, package, filesystem and systemd functionality available.
