# Repository Lists

Repository lists allow developers to distribute applications through LinuxToys without having to create a complete LinuxToys shell script.

A repository-list entry describes the application, where LinuxToys should obtain it, which systems it supports, any dependencies it requires, and optional setup that should be performed after installation.

LinuxToys converts a valid entry into a temporary installation script and runs it through the normal LinuxToys execution and transaction system.

## File location

Repository entries can be stored in either:

```text
scripts/repos.json
```

or in any `.json` file located recursively under:

```text
scripts/lists/
```

For example:

```text
scripts/
├── repos.json
└── lists/
    ├── openlogi.json
    ├── amethyst/
    │   ├── app.json
    │   ├── icon.svg
    │   └── setup.sh
    └── utilities/
        └── example.json
```

`repos.json` is loaded first for backwards compatibility. Files under `scripts/lists/` are then loaded recursively in deterministic alphabetical order.

A JSON file may contain either one entry:

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "An example application.",
  "category": "utilities"
}
```

or an array containing several entries:

```json
[
  {
    "name": "example-one",
    "repo": "developer/example-one",
    "description": "First example.",
    "category": "utilities"
  },
  {
    "name": "example-two",
    "repo": "developer/example-two",
    "description": "Second example.",
    "category": "utilities"
  }
]
```

Invalid JSON files and invalid entries are ignored rather than preventing other repository-list files from loading.

## Required fields

Every entry must contain these four non-empty string fields:

| Field         | Purpose                                                   |
| ------------- | --------------------------------------------------------- |
| `name`        | Internal LinuxToys identity for the application.          |
| `repo`        | Upstream repository or project identifier.                |
| `description` | Default user-facing description.                          |
| `category`    | LinuxToys category in which the application is displayed. |

Example:

```json
{
  "name": "example-app",
  "repo": "developer/example-app",
  "description": "A useful example application.",
  "category": "utilities"
}
```

The `name` must be unique across all repository lists. Names are compared case-insensitively. If two entries use the same name, only the first loaded entry is used.

The `category` corresponds to the LinuxToys category directory name. For example:

```json
"category": "gaming"
```

places the entry in the `gaming` category when that category is displayed.

## Basic optional fields

Common optional metadata can be added alongside the required fields:

```json
{
  "name": "example-app",
  "repo": "developer/example-app",
  "description": "A useful example application.",
  "description_tag": "example_app_desc",
  "category": "utilities",
  "icon": "example.svg"
}
```

### `description_tag`

```json
"description_tag": "example_app_desc"
```

If the supplied translation table contains this key, LinuxToys uses its translated value instead of `description`.

The regular `description` remains required and acts as the fallback.

### `icon`

If omitted, LinuxToys uses:

```text
application-x-executable
```

There are two ways to provide an icon.

A normal icon name or filename can continue to use the standard LinuxToys icon resolution:

```json
"icon": "example.svg"
```

or:

```json
"icon": "application-x-executable"
```

Applications whose JSON lives under `scripts/lists/` may instead ship their icon beside the list file:

```text
scripts/lists/example/
├── app.json
└── icon.svg
```

with:

```json
"icon": "./icon.svg"
```

Subdirectories are also supported:

```json
"icon": "assets/icon.png"
```

Local repository-list icons must:

* use a relative path;
* stay within `scripts/lists/`;
* exist on disk;
* be either SVG or PNG.

If those conditions are not met, LinuxToys falls back to `application-x-executable`.

---

## Installation types

The `type` field tells LinuxToys how the application should be installed.

If `type` is omitted, it defaults to:

```json
"type": "git"
```

Currently usable types are:

| Type      | Installation mechanism                            |
| --------- | ------------------------------------------------- |
| `git`     | Latest upstream release through `pkg_fromrelease` |
| `flathub` | Flatpak application through `pkg_flat`            |
| `native`  | Distribution package through `pkg_install`        |
| `url`     | Direct package URL through `pkg_fromurl`          |

`repository` is reserved by the parser but third-party repository installation is not implemented yet. Entries using it are currently rejected and are not shown.

It is also possible to key types to certain `os` values. For example, if you want to use a `native` package for *Arch Linux* and derivatives and fallback to `git`:

```json
"type": {
  "arch": "native",
  "all": "git"
}
```

### `git`

This is the default and simplest option.

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "Example application.",
  "category": "utilities",
  "type": "git"
}
```

LinuxToys generates:

```bash
pkg_fromrelease developer/example
```

The `repo` value is therefore the value expected by LinuxToys' `pkg_fromrelease` helper.

Since `git` is the default, this is equivalent:

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "Example application.",
  "category": "utilities"
}
```

### `flathub`

Use `flathub` when the application should be installed as a Flatpak:

```json
{
  "name": "example",
  "repo": "https://github.com/developer/example",
  "description": "Example application.",
  "category": "utilities",
  "type": "flathub",
  "package-name": "com.example.Application"
}
```

LinuxToys runs:

```bash
pkg_flat com.example.Application
```

### Multiple Flatpaks

`package-name` may also be an array:

```json
"package-name": [
  "com.example.Application",
  "com.example.Extension"
]
```

LinuxToys installs every listed package.

Flatpak installation implicitly requires a systemd-compatible host and cannot be performed inside a container.

### `native`

Use `native` when the application is already provided by the distribution's package manager:

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "Example application.",
  "category": "utilities",
  "type": "native",
  "package-name": "example"
}
```

LinuxToys runs:

```bash
pkg_install example
```

### Multiple native packages

An application may require several packages:

```json
"package-name": [
  "example",
  "example-data",
  "example-plugins"
]
```

Each package is installed through `pkg_install`.

### Different package names on different distributions

`package-name` can also be an object:

```json
"package-name": {
  "debian": "example",
  "fedora": "example-app",
  "arch": "example-git"
}
```

Lists can also be used inside the mapping:

```json
"package-name": {
  "debian": [
    "example",
    "example-data"
  ],
  "fedora": [
    "example-app",
    "example-assets"
  ]
}
```

A generic fallback may be supplied with `all`:

```json
"package-name": {
  "all": "example",
  "fedora": "example-app"
}
```

When the current system matches an explicit distribution entry, that entry takes precedence over `all`.

This means a Fedora installation uses:

```text
example-app
```

while another supported distribution without a more specific mapping falls back to:

```text
example
```

### Native package mapping priority

Some systems expose more than one compatibility key. LinuxToys resolves package mappings using this priority:

```text
ublue
deepin
zorin
pika
manjaro
cachy
ostree
ubuntu
debian
fedora
rhel
suse
solus
arch
all
```

This allows derivatives to override their parent distribution.

For example:

```json
"package-name": {
  "all": "example",
  "arch": "example",
  "cachy": "example-cachyos"
}
```

uses `example-cachyos` on CachyOS rather than the generic Arch package.

### `url`

The `url` type is intended for developers or companies that distribute packages directly, such as through their own CDN or release server.

Example:

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "Example application.",
  "category": "utilities",
  "type": "url",
  "urls": {
    "deb": "https://downloads.example.org/example-amd64.deb",
    "rpm": "https://downloads.example.org/example-x86_64.rpm",
    "pkg.tar.zst": "https://downloads.example.org/example-x86_64.pkg.tar.zst",
    "appimage": "https://downloads.example.org/Example.AppImage"
  }
}
```

Supported URL keys are:

```text
deb
rpm
pacman
pkg.tar.zst
flatpak
appimage
```

URLs must use HTTP or HTTPS.

LinuxToys selects a package appropriate for the current distribution and passes its URL to:

```bash
pkg_fromurl URL
```

### Package selection

LinuxToys prefers a native package whenever one is available.

Debian-family systems check:

```text
deb
```

This includes Debian, Ubuntu, Deepin, Zorin OS and PikaOS.

RPM-family systems check:

```text
rpm
```

This includes Fedora, RHEL, openSUSE, rpm-ostree systems and Universal Blue.

Arch-family systems check:

```text
pkg.tar.zst
pacman
```

in that order.

This includes Arch Linux, CachyOS and Manjaro.

If no usable native package exists, LinuxToys checks portable packages in this order:

```text
appimage
flatpak
```

For example:

```json
"urls": {
  "deb": "https://example.org/app.deb",
  "rpm": "https://example.org/app.rpm",
  "appimage": "https://example.org/App.AppImage"
}
```

will install the DEB on Debian-family systems, the RPM on RPM-family systems, and can fall back to the AppImage elsewhere.

A URL entry is only displayed when LinuxToys can resolve one of its provided package URLs for the current system.

---

## Tarball Applications

The `tar` type is intended for applications distributed as **prebuilt binary tarballs** through GitHub or Codeberg releases. It allows LinuxToys to install software that does not provide a native package, Flatpak, or AppImage, but ships a ready-to-run application as a `.tar.gz` or `.tar.xz` archive.

> **Note:** `tar` is intended for binary application releases, not source archives, and **mandates** a post-install script to finish setting it up.

### GitHub and Codeberg Releases

For an application distributed as a tarball attached to a GitHub or Codeberg release, use:

```json
{
  "name": "myapp",
  "type": "tar",
  "repo": "https://github.com/example/myapp"
}
```

Internally, this causes LinuxToys to use the release installer in tarball mode:

```bash
pkg_fromrelease --tar "https://github.com/example/myapp"
```

The latest release is queried and LinuxToys looks specifically for a compatible `.tar.gz` or `.tar.xz` release asset.

GitHub's automatically generated repository source archives are not considered, as they are not part of the release's uploaded asset list. Release assets identified as source-oriented archives are also filtered out. Developers should therefore provide the **compiled application tarball as an actual release asset**.

Architecture information in asset names is respected. For example:

```text
myapp-2.4.0-x86_64.tar.xz
myapp-2.4.0-aarch64.tar.xz
myapp-2.4.0-source.tar.gz
```

On an x86-64 system, LinuxToys will select the `x86_64` application archive while excluding the incompatible architecture and source archive.

### Direct URLs

Tarballs hosted directly by the developer or project infrastructure can instead use the `url` type:

```json
{
  "name": "myapp",
  "type": "url",
  "urls": {
    "tar": "https://example.com/releases/myapp.tar.xz"
  }
}
```

This invokes the URL installer in tarball mode:

```bash
pkg_fromurl --tar "https://example.com/releases/myapp.tar.xz"
```

The URL may point directly to the archive or use an HTTP redirect. LinuxToys resolves the download filename before determining the archive format.

The resolved file must be a supported tarball format.

### Supported Formats

The tarball handler currently accepts:

```text
.tar.gz
.tar.xz
```

Other archive formats should not be declared using `tar`.

### Archive Layout

Developers may package the application either inside a single top-level directory or with the application files directly at the root of the archive.

A tarball containing its own directory:

```text
MyApp/
├── bin/
│   └── myapp
├── lib/
└── resources/
```

is installed directly as:

```text
~/.local/linuxtoys/apps/MyApp/
```

LinuxToys detects the existing common top-level directory and does **not** create an additional wrapper such as `MyApp/MyApp/`.

A tarball containing loose root-level files is also supported:

```text
myapp
lib/
resources/
README.md
```

In this case, LinuxToys creates an application directory using the archive filename, excluding the `.tar.gz` or `.tar.xz` extension.

For example:

```text
myapp-2.4.0.tar.xz
```

would produce:

```text
~/.local/linuxtoys/apps/myapp-2.4.0/
```

For this reason, developers are encouraged to ship the contents inside a sensibly named top-level directory when they need the installation directory to have a stable name across releases.

### Updates

Running the same tarball installation again is treated as an update.

LinuxToys replaces the existing target directory with the newly extracted application rather than merging the new archive into the old installation. This ensures that files removed by upstream releases do not remain behind after an update.

Developers distributing successive releases should therefore preferably keep the tarball's top-level application directory consistent between versions:

```text
myapp/
```

rather than:

```text
myapp-2.4.0/
myapp-2.5.0/
```

A stable directory name allows subsequent releases to replace the previous installation cleanly.

### OS-Specific Types

`tar` can also be selected through the normal OS-specific `type` mapping. For example, a project may use a native Arch Linux package while distributing a binary tarball for other supported systems:

```json
{
  "name": "myapp",
  "type": {
    "arch": "native",
    "all": "tar"
  },
  "package-name": {
    "arch": "myapp"
  },
  "repo": "https://github.com/example/myapp"
}
```

On Arch Linux and derivatives, LinuxToys will use the native package. Other compatible systems will obtain the binary tarball from the project's releases.

### Choosing Between `tar` and `url`

Use:

```json
"type": "tar"
```

when the binary tarball is published as an asset of the project's GitHub or Codeberg releases and LinuxToys should automatically follow new releases.

Use:

```json
"type": "url"
```

with:

```json
"urls": {
  "tar": "https://example.com/application.tar.gz"
}
```

when the archive is hosted at a URL supplied directly by the developer.

In both cases, the archive must contain an already built, usable application. Compilation of source tarballs is outside the scope of the `tar` type.

---

## Compatibility

Repository-list entries can restrict themselves to particular operating systems, desktop environments, hardware, init systems, or container environments.

Fields that are omitted are generally treated as unrestricted.

### Operating systems

Use `os` to restrict an application to one or more supported operating systems.

One OS:

```json
"os": "fedora"
```

Several:

```json
"os": [
  "fedora",
  "arch",
  "debian"
]
```

Supported values are:

```text
debian
ubuntu
cachy
arch
fedora
rhel
suse
ostree
ublue
zorin
solus
pika
deepin
manjaro
```

The entry is available when at least one requested OS compatibility key matches the host.

For example:

```json
"os": [
  "fedora",
  "rhel"
]
```

allows the entry on either compatible Fedora or RHEL systems.

### Desktop environment

The optional `desktop` field restricts an application to particular desktop environments.

Supported values are:

```text
gnome
plasma
other
```

One desktop:

```json
"desktop": "gnome"
```

Several:

```json
"desktop": [
  "gnome",
  "plasma"
]
```

An entry is accepted when at least one specified desktop matches the current environment.

Use this only when the application or integration genuinely depends on a particular desktop.

### Hardware

Hardware compatibility is declared under `hardware`.

For example:

```json
"hardware": {
  "gpu": "nvidia"
}
```

or:

```json
"hardware": {
  "gpu": [
    "amd",
    "intel"
  ],
  "cpu": "amd"
}
```

LinuxToys converts these values to its normal compatibility keys.

For example:

```text
gpu: "amd"    -> gpu-amd
cpu: "intel"  -> cpu-intel
```

Already-prefixed values may also be used:

```json
"hardware": {
  "gpu": "gpu-xe"
}
```

Values may be strings or arrays.

Within each hardware class, multiple values act as alternatives. For example:

```json
"hardware": {
  "gpu": [
    "amd",
    "nvidia"
  ]
}
```

means AMD **or** NVIDIA rather than requiring both.

The special value:

```text
all
```

does not introduce a hardware requirement.

Hardware names ultimately correspond to compatibility keys exposed by LinuxToys, so use keys supported by the LinuxToys compatibility subsystem.

### systemd

The optional `systemd` field can explicitly restrict an entry according to the init system.

Requires systemd:

```json
"systemd": "yes"
```

Requires a non-systemd system:

```json
"systemd": "no"
```

Omit the field, use `null`, or use an empty string when either is acceptable.

Flatpak installations implicitly require systemd, regardless of whether this field is supplied.

Declaring `services` also implicitly requires systemd.

### Containers

Container compatibility is controlled with:

```json
"container": "allow"
```

or:

```json
"container": "deny"
```

The default is:

```json
"container": "allow"
```

so most entries do not need to specify this field.

Use:

```json
"container": "deny"
```

when an application cannot be installed correctly from inside a container.

### Automatic container restrictions

LinuxToys automatically rejects entries inside containers when the selected installation would install:

```text
Flatpak
AppImage
```

This applies even if:

```json
"container": "allow"
```

was explicitly specified.

The same restriction applies when an entry declares a Flathub dependency.

This guardrail exists because Flatpak and AppImage installation should not be nested inside the supported container workflow.

---

## Dependencies

The optional `dependencies` field installs packages before the main application.

Dependencies are an array of objects.

Currently supported dependency types are:

```text
native
flathub
```

### Native dependency

```json
"dependencies": [
  {
    "type": "native",
    "package-name": "git"
  }
]
```

Native dependencies support exactly the same package-name forms as a native application.

A single package:

```json
{
  "type": "native",
  "package-name": "git"
}
```

Several packages:

```json
{
  "type": "native",
  "package-name": [
    "git",
    "curl"
  ]
}
```

Distribution-specific packages:

```json
{
  "type": "native",
  "package-name": {
    "debian": "libexample-dev",
    "fedora": "example-devel",
    "arch": "example"
  }
}
```

And mappings can themselves contain package arrays:

```json
{
  "type": "native",
  "package-name": {
    "debian": [
      "libexample1",
      "libexample2"
    ],
    "fedora": [
      "example-libs",
      "example-data"
    ]
  }
}
```

If no native dependency mapping can be resolved for the host, the application itself is considered incompatible and is not displayed.

### Flathub dependency

```json
"dependencies": [
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

Multiple Flatpaks are allowed:

```json
{
  "type": "flathub",
  "package-name": [
    "org.example.Runtime",
    "org.example.Extension"
  ]
}
```

A Flathub dependency implicitly requires systemd and makes the entry incompatible with container installation.

### Several dependencies

Different dependencies can be combined:

```json
"dependencies": [
  {
    "type": "native",
    "package-name": {
      "debian": "example-helper",
      "fedora": "example-helper",
      "arch": "example-helper"
    }
  },
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

Dependencies run before installation of the primary package.

---

## Overrides

Additional installation behavior can be declared using:

```json
"overrides": {}
```

Currently supported override keys are:

```text
flatpak
pre
post
```

Other keys cause the entry to be rejected.

### Pre-install hook

`pre` runs before the dependency and application installation commands.

For short operations, it may contain inline shell:

```json
"overrides": {
  "pre": "mkdir -p \"$HOME/.config/example\""
}
```

Because this content is inserted directly into the generated Bash installation script, it should be kept small and predictable.

### Post-install hook

`post` runs after the dependencies, application installation, Flatpak overrides and service setup.

Example:

```json
"overrides": {
  "post": "touch \"$HOME/.config/example/installed\""
}
```

### External pre/post scripts

More complex setup can be shipped as a separate script under `scripts/lists/`.

Example layout:

```text
scripts/lists/example/
├── app.json
├── pre-install.sh
└── post-install.sh
```

The JSON can reference them with:

```json
"overrides": {
  "pre": {
    "script": "example/pre-install.sh"
  },
  "post": {
    "script": "example/post-install.sh"
  }
}
```

These hooks are executed through LinuxToys' `run_list_hook` helper.

Hook paths must be relative. Absolute paths and paths that attempt to traverse above `scripts/lists/` are rejected.

For example, this is invalid:

```json
"pre": {
  "script": "../outside.sh"
}
```

External scripts are most appropriate when setup is too complex to reasonably express as a short inline command.

## Execution order

The generated installation procedure runs in this order:

```text
pre hook

dependencies

main application installation

Flatpak overrides

systemd services

post hook

success message
```

This ordering is important when writing hooks. A `pre` hook cannot assume the application has already been installed, while a `post` hook can.

---

## Flatpak overrides

LinuxToys can apply Flatpak permissions after installation using its `flatpak_override` helper.

Example:

```json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-config/example",
      "target": "com.example.Application"
    }
  ]
}
```

Every Flatpak override requires four fields:

| Field     | Description                               |
| --------- | ----------------------------------------- |
| `scope`   | `user` or `system`                        |
| `type`    | Type of Flatpak override                  |
| `setting` | Permission or setting passed to LinuxToys |
| `target`  | Target Flatpak application                |

Supported override types are:

```text
fs
name
dbus
share
env
runtime
device
socket
filesystem
talk-name
talk-dbus
```

Several overrides may be applied:

```json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-config/example",
      "target": "com.example.Application"
    },
    {
      "scope": "user",
      "type": "device",
      "setting": "dri",
      "target": "com.example.Application"
    }
  ]
}
```

LinuxToys translates each entry into a call equivalent to:

```bash
flatpak_override SCOPE TYPE SETTING TARGET
```

---

## systemd services

The `services` field can tell LinuxToys to enable and immediately start systemd units after installation.

Any entry that declares `services` is automatically restricted to systemd hosts.

### One system service

The shortest form is:

```json
"services": "example"
```

This defaults to a system-level service and becomes:

```text
example.service
```

LinuxToys effectively performs:

```bash
sudo systemctl enable --now example.service
```

### Several system services

```json
"services": [
  "example",
  "example-helper"
]
```

Both default to system scope.

### System and user services

For explicit control:

```json
"services": {
  "system": [
    "example.service"
  ],
  "user": [
    "example-tray.service"
  ]
}
```

System services are enabled with:

```bash
sudo systemctl enable --now UNIT
```

User services are enabled with:

```bash
systemctl --user enable --now UNIT
```

LinuxToys records these operations in its transaction map so they can participate in the normal revert workflow.

### Unit suffixes

If no recognized systemd unit suffix is supplied, LinuxToys automatically appends:

```text
.service
```

Therefore:

```json
"services": "example"
```

and:

```json
"services": "example.service"
```

are equivalent.

Recognized unit suffixes include:

```text
.service
.socket
.timer
.path
.mount
.automount
.target
.slice
.scope
.device
.swap
```

This allows non-service units as well:

```json
"services": {
  "system": [
    "example.socket",
    "example.timer"
  ]
}
```

---

## Complete example

The following example demonstrates most of the currently supported repository-list features:

```json
{
  "name": "example-app",
  "repo": "https://github.com/example/example-app",
  "description": "A cross-platform example application.",
  "description_tag": "example_app_desc",
  "category": "utilities",
  "icon": "./icon.svg",

  "type": "url",

  "urls": {
    "deb": "https://downloads.example.org/example-app-amd64.deb",
    "rpm": "https://downloads.example.org/example-app-x86_64.rpm",
    "pkg.tar.zst": "https://downloads.example.org/example-app-x86_64.pkg.tar.zst",
    "appimage": "https://downloads.example.org/ExampleApp-x86_64.AppImage"
  },

  "os": [
    "debian",
    "ubuntu",
    "deepin",
    "zorin",
    "pika",
    "fedora",
    "rhel",
    "suse",
    "ostree",
    "ublue",
    "arch",
    "cachy",
    "manjaro"
  ],

  "desktop": [
    "gnome",
    "plasma"
  ],

  "hardware": {
    "gpu": [
      "amd",
      "intel",
      "nvidia"
    ]
  },

  "systemd": "yes",
  "container": "deny",

  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": [
          "curl",
          "git"
        ],
        "fedora": [
          "curl",
          "git"
        ],
        "arch": [
          "curl",
          "git"
        ],
        "all": "curl"
      }
    }
  ],

  "overrides": {
    "pre": {
      "script": "example-app/pre-install.sh"
    },

    "flatpak": [],

    "post": {
      "script": "example-app/post-install.sh"
    }
  },

  "services": {
    "system": [
      "example-app.service"
    ],
    "user": [
      "example-app-tray.service"
    ]
  }
}
```

A corresponding directory could look like:

```text
scripts/lists/example-app/
├── app.json
├── icon.svg
├── pre-install.sh
└── post-install.sh
```

---
<a id="minimal-examples"></a>

## Minimal examples

### GitHub release

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "An example application.",
  "category": "utilities"
}
```

### Flathub

```json
{
  "name": "example",
  "repo": "https://github.com/developer/example",
  "description": "An example application.",
  "category": "utilities",
  "type": "flathub",
  "package-name": "com.example.Application"
}
```

### Native package

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "An example application.",
  "category": "utilities",
  "type": "native",
  "package-name": {
    "debian": "example",
    "fedora": "example",
    "arch": "example"
  }
}
```

### Direct package URLs

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "An example application.",
  "category": "utilities",
  "type": "url",
  "urls": {
    "deb": "https://example.org/download/example.deb",
    "rpm": "https://example.org/download/example.rpm",
    "appimage": "https://example.org/download/Example.AppImage"
  }
}
```

---

## Validation behavior

Repository-list entries are validated before appearing in LinuxToys.

An entry will be silently skipped when its required fields are missing, its installation type is invalid or unusable, its compatibility requirements do not match the current machine, a native package or URL cannot be resolved for the host, a dependency cannot be satisfied, an override is malformed, its service definition is invalid, its container setting is invalid, or its `name` duplicates an entry that was loaded earlier.

This means developers should test their repository-list entry on every system class they intend to support.

A repository-list entry is exposed to the rest of LinuxToys in much the same way as a regular LinuxToys script. LinuxToys generates a virtual identity in the form:

```text
repo://NAME
```

and materializes a temporary shell script only when the entry needs to be executed.

Repository-list installations are marked as revertible and participate in LinuxToys' normal installation and transaction workflow.

---

## Choosing an installation method

Use `git` when your project publishes installable artifacts through releases compatible with LinuxToys' release parser.

Use `flathub` when Flatpak is the intended distribution method.

Use `native` when the application is already available through the distributions' normal package repositories.

Use `url` when you publish packages directly and want LinuxToys to select the best format for each distribution.

Dependencies, compatibility fields, services and overrides can then be layered on top of those installation types as necessary.

Keep entries as simple as possible. Repository lists are intended to cover applications that can be installed declaratively. If an application's installation process requires substantial custom logic, a regular LinuxToys script may still be the more appropriate integration method.
