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

<a id="git-package"></a>

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

<a id="flathub-package"></a>

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

<a id="native-package"></a>

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

<a id="url-fetching"></a>

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

<a id="single-binary"></a>

## Single-Binary Applications

LinuxToys can install applications distributed as a **single executable binary**, without requiring a native package, AppImage, Flatpak, or tarball.

This is useful for applications whose upstream releases provide standalone executables such as:

```text
myapp
myapp-linux-x86_64
myapp-v1.4.2-linux-amd64
```

When LinuxToys installs a single-binary application, it automatically:

* creates an application directory under:

  ```text
  ~/.local/linuxtoys/apps/<application name>/
  ```

* copies the downloaded binary into that directory;

* marks the binary as executable;

* creates an application-menu shortcut using the repository-list name, description, and icon;

* registers the installation with LinuxToys' transaction system so it can be reverted normally.

For repository lists, single binaries can be installed either from a GitHub, Codeberg or GitLab release or directly from a URL.

### Installing a Binary from a GitHub, Codeberg or GitLab Release

Use:

```json
"type": "bin"
```

The `repo` field must point to the application's repository, while `package-name` must contain the **exact name of the release asset** containing the executable.

For example:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://github.com/example/example",
  "type": "bin",
  "package-name": "example-linux-x86_64",
  "icon": "example.svg"
}
```

LinuxToys will obtain the latest stable release, locate the requested asset, download it, and install it as a standalone application.

### The Binary Filename Must Be Explicit

Unlike package formats such as `.deb`, `.rpm`, or `.AppImage`, standalone binaries frequently have **no identifying file extension**.

Because of this, LinuxToys cannot safely determine which release asset is the application's binary automatically.

Developers using `"type": "bin"` must therefore provide the exact release filename through `package-name`.

For example, if a release contains:

```text
example-linux-x86_64
example-linux-aarch64
example.sha256
source.tar.gz
```

the repository entry should explicitly select:

```json
"package-name": "example-linux-x86_64"
```

Wildcards should not be used for binary release assets.

### Release Versions in Binary Filenames

Some projects include the release version directly in the binary filename.

For example, an upstream release tagged:

```text
v2.4.1
```

might contain:

```text
example-v2.4.1-linux-x86_64
```

For this case, LinuxToys provides:

```text
$APP_GIT_VERSION
```

inside the binary asset name.

You can therefore write:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://github.com/example/example",
  "type": "bin",
  "package-name": "example-$APP_GIT_VERSION-linux-x86_64"
}
```

LinuxToys will determine the latest stable release version first and substitute `$APP_GIT_VERSION` before locating the asset.

This avoids having to update the repository-list entry whenever upstream publishes a new version.

> `APP_GIT_VERSION` corresponds to the GitHub release tag. If upstream uses tags such as `v2.4.1`, the `v` is therefore part of the value.

### Installing a Binary Directly from a URL

A standalone binary can also be installed through the regular `"url"` repository-list type.

Use the `bin` key under `urls`:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://example.org",
  "type": "url",
  "urls": {
    "bin": "https://example.org/releases/example-linux-x86_64"
  },
  "icon": "example.svg"
}
```

LinuxToys will download the file through its normal URL download mechanism and then install it using the same single-binary installation procedure.

This is especially useful for projects that publish standalone executables outside GitHub Releases.

### Architecture-Specific Entries

If upstream publishes separate binaries for different CPU architectures, the repository entry should select the correct asset for the systems the entry supports.

For example:

```text
example-linux-x86_64
example-linux-aarch64
```

A repository entry intended only for x86-64 systems should reference:

```json
"package-name": "example-linux-x86_64"
```

and use the appropriate repository-list hardware or compatibility restrictions when necessary.

LinuxToys may use architecture information present in release filenames while locating assets, but the developer should still identify the intended binary explicitly.

### Application Menu Integration

Single-binary installations automatically receive an application-menu shortcut.

The shortcut uses the same metadata already provided by the repository entry:

* `name` becomes the application display name;
* the translated `description`, when available, becomes the application description;
* `icon` becomes the application icon;
* the installed executable becomes the shortcut's launch command.

As a result, repository-list developers normally do **not** need to provide a post-install script merely to create a `.desktop` file for a standalone application.

The binary is installed under:

```text
~/.local/linuxtoys/apps/<application name>/
```

and the generated shortcut points to the installed copy rather than the temporary downloaded file.

### Choosing Between `bin` and `url`

Use `"type": "bin"` when:

* the application is hosted on GitHub Releases;
* upstream distributes one executable file;
* you can identify the release asset by its exact filename.

Use `"type": "url"` with:

```json
"urls": {
  "bin": "..."
}
```

when:

* the standalone executable is available from a stable direct URL;
* the project does not use GitHub Releases for distribution;
* or you explicitly want LinuxToys to download from another source.

---

<a id="tarball-package"></a>

## Tarball Applications

The `tar` type is intended for applications distributed as **prebuilt binary tarballs** through GitHub or Codeberg releases. It allows LinuxToys to install software that does not provide a native package, Flatpak, or AppImage, but ships a ready-to-run application as a `.tar.gz` or `.tar.xz` archive.

> **Note:** `tar` is intended for binary application releases, not source archives, and **mandates** a post-install script to finish setting it up. Installations of this kind export a **`LINUXTOYS_TARBALL_DIR`** variable pointing towards the final directory name after the tarball is extracted for your convenience in the post-install script. If you are distributing an application in this format, you might also be interested in [creating a `.desktop` app menu shortcut automatically](corelibraries.html#app-shortcuts).

### GitHub, Codeberg and GitLab Releases

For an application distributed as a tarball attached to a GitHub, Codeberg or GitLab release, use:

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

## Using Dynamic Download URLs

Application pages do not change how an application is installed. They can therefore be combined with other repository-list functionality, including dynamically discovered download URLs.

For example:

```json
{
  "name": "Example App",
  "repo": "https://example.org",
  "category": "office",
  "icon": "./example.svg",
  "type": "url",

  "urls": {
    "appimage": {
      "env": "URL"
    }
  },

  "overrides": {
    "pre": {
      "script": "./example-pre.sh"
    }
  },

  "descriptions": "descriptions.json",
  "screenshots": "screenshots/"
}
```

The pre-installation script can discover the current download URL and export it:

```bash
#!/usr/bin/env bash

# Determine the appropriate release URL...
export URL="https://example.org/releases/latest/example.AppImage"
```

LinuxToys then expands that environment variable when `pkg_fromurl` is invoked.

Dynamic URL declarations require a pre-installation hook. Environment-variable names are explicitly declared using the `{"env": "VARIABLE"}` form rather than placing shell variables directly in URL strings.

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

#### Enforce system-level flatpaks

Certain flatpak applications may have issues working on user level. For such cases, there's an override option available:

```json
"overrides": {
    "skip-user": true
}
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

<a id="app-pages"></a>

## Application Pages

Repository-list entries can optionally provide an **application page**. This gives users more information about an application before installing it, including a longer description, screenshots, and optional purchase or donation links.

Application pages are intended for applications that benefit from a richer presentation than the standard installation confirmation dialog.

If none of the application-page fields are provided, LinuxToys skips the application page entirely and follows the normal installation flow.

#### Basic Example

An entry with an application page may look like this:

```json
{
  "name": "Example App",
  "repo": "https://example.org",
  "category": "office",
  "icon": "./example.svg",
  "type": "url",

  "urls": {
    "appimage": "https://example.org/releases/example.AppImage"
  },

  "description": "A short description of the application.",

  "long-description": "A longer description containing additional information about the application and its features.",

  "screenshots": "screenshots/",

  "donate": "https://example.org/donate"
}
```

When the user selects this entry, LinuxToys opens its application page instead of immediately displaying the installation confirmation.

The page retains the usual LinuxToys application header containing the application's name, short description, repository information, and icon. The main area displays the longer description and screenshot viewer.

Selecting **Install** continues through the normal LinuxToys installation flow.

### Long Descriptions

The short `description` remains the text displayed throughout the normal LinuxToys interface. The application page can additionally provide a longer description:

```json
"long-description": "A detailed description of the application, its purpose, and its main features."
```

The underscore form is also accepted:

```json
"long_description": "A detailed description."
```

Long descriptions can also use the normal LinuxToys translation system:

```json
"description": "Short fallback description.",
"description_tag": "example_desc",
"long-description": "Long fallback description.",
"long-description_tag": "example_long_desc"
```

For repository entries with substantial descriptions, however, a repository-local description catalog is recommended.

#### Markdown Long Descriptions

Developers can use a Markdown file instead of supplying the long description directly. To opt into Markdown formatting, set the long description to a relative path ending in `.md`:

```json
"long-description": "description.md"
```

LinuxToys detects the `.md` extension, loads the referenced file, and renders its contents as Markdown on the application page.

Markdown descriptions support standard formatting such as:

```markdown
# Example Application

A **powerful** application with support for:

- Feature one
- Feature two
- Feature three

## Additional Information

Visit the [project website](https://example.org) for more information.
```

To keep application pages visually consistent, Markdown headings of all levels are displayed using a restrained heading size equivalent to `####`. This allows developers to use normal Markdown document structure without producing excessively large text in the LinuxToys interface.

Markdown is entirely optional. A normal text value continues to be displayed as plain text:

```json
"long-description": "This remains an ordinary plain-text long description."
```

Markdown descriptions can also be used with translated long-description tags. This allows each language to provide its own `.md` document, as described below.

### Repository-Local Description Translations

Repository lists can keep their application descriptions separate from the main LinuxToys translation files by placing a JSON description catalog alongside the repository-list file.

Reference it with:

```json
"descriptions": "descriptions.json"
```

`description-file` is also accepted as an alias.

The description file must be located in the **same directory as the repository-list JSON**.

For example:

```text
scripts/lists/example/
├── repository.json
├── descriptions.json
├── description.en.md
├── description.pt-BR.md
├── example.svg
└── screenshots/
    ├── main.webp
    ├── editor.webp
    └── settings.webp
```

A `descriptions.json` file can use plain-text long descriptions:

```json
{
  "description_tag": "example_desc",
  "description_long_tag": "example_long",
  "en": {
    "example_desc": "A short description of the application.",
    "example_long": "A longer description explaining the application and its main features."
  },
  "pt": {
    "example_desc": "Uma descrição curta do aplicativo.",
    "example_long": "Uma descrição mais longa explicando o aplicativo e seus principais recursos."
  }
}
```

`description_tag` identifies the short description, while `description_long_tag` identifies the long application-page description.

The same long-description tag can instead point to a Markdown file for each language:

```json
{
  "description_tag": "example_desc",
  "description_long_tag": "example_long",
  "en": {
    "example_desc": "A short description of the application.",
    "example_long": "description.en.md"
  },
  "pt-BR": {
    "example_desc": "Uma descrição curta do aplicativo.",
    "example_long": "description.pt-BR.md"
  }
}
```

In this case, LinuxToys first resolves `example_long` according to the selected language and then loads the `.md` file referenced by that translation. This allows each translation to provide a complete independently formatted Markdown description.

Markdown paths are resolved relative to the repository-list file. They can also point to files inside subdirectories belonging to the repository entry, allowing a layout such as:

```text
scripts/lists/example/
├── repository.json
├── descriptions.json
└── descriptions/
    ├── description.en.md
    └── description.pt-BR.md
```

with:

```json
{
  "description_long_tag": "example_long",
  "en": {
    "example_long": "descriptions/description.en.md"
  },
  "pt-BR": {
    "example_long": "descriptions/description.pt-BR.md"
  }
}
```

LinuxToys first looks for the currently selected language and falls back to English when an appropriate translation is unavailable. This applies equally to plain-text and Markdown long descriptions.

Existing inline descriptions and translation tags remain supported, which is useful when migrating an existing LinuxToys script into a repository-list entry.

### Screenshots

Use `screenshots` to provide images for the application page.

There are two ways to do this.

#### Screenshot Directory

The simplest method is to point to a directory:

```json
"screenshots": "screenshots/"
```

LinuxToys automatically loads the supported image files directly inside that directory.

Supported formats are:

* `.png`
* `.jpg`
* `.jpeg`
* `.webp`
* `.svg`

The files are sorted by filename, so filenames can also be used to control their order:

```text
screenshots/
├── 01-main.webp
├── 02-editor.webp
└── 03-settings.webp
```

#### Individual Screenshots

Specific files can instead be listed:

```json
"screenshots": [
  "screenshots/main.webp",
  "screenshots/editor.webp",
  "screenshots/settings.webp"
]
```

Screenshot paths are relative to the repository-list JSON.

For security, screenshot paths must remain within the `scripts/lists` hierarchy. Paths that resolve outside it are rejected.

#### Screenshot Viewer

When multiple screenshots are available, LinuxToys presents them as a circular viewer.

Users can move both forwards and backwards through the images. Reaching either end wraps around to the other end:

```text
1 → 2 → 3 → 1
```

and:

```text
1 ← 2 ← 3 ← 1
```

The keyboard Left and Right arrow keys can also be used while the application page is open.

### Developer Identification

A repository entry can specify the name of the developer, team, or company responsible for the application using `developer`:

```json
"developer": "Example Company"
```

When available, the developer name is displayed on the application page, allowing users to easily identify who develops or maintains the software.

For applications hosted on GitHub, this field is optional. If `developer` is not specified, LinuxToys automatically uses the name of the GitHub user or organization that owns the repository.

For example:

```json
{
  "name": "Example App",
  "repo": "https://github.com/example-company/example-app"
}
```

will result in:

```text
example-company
```

being used as the developer name.

The `developer` field can be used to override this automatic value with a more appropriate display name:

```json
{
  "name": "Example App",
  "repo": "https://github.com/example-company/example-app",
  "developer": "Example Company"
}
```

This is particularly useful when the public name of a person, project, or company differs from its GitHub account or organization name.

Automatic detection applies only to repositories hosted on GitHub. For other repository sources, set `developer` explicitly if you want a developer name to be displayed.

#### Official Support Indicator

When an application is part of the official LinuxToys index, a verification indicator is displayed beside the developer name on its application page.

This indicator identifies applications with official support in LinuxToys. Its status is determined by the official index itself and **cannot be enabled through a repository-list entry**.

Therefore, setting `developer` only controls the displayed developer name and does not grant or imply official support status. [You can check how to obtain official support status here](documentation.html#official-support).

### Donation Links

A donation link can be added with:

```json
"donate": "https://example.org/donate"
```

The object form is also accepted:

```json
"donate": {
  "url": "https://example.org/donate"
}
```

LinuxToys displays a **Donate** button on the application page which opens the specified URL.

Only valid HTTP or HTTPS URLs are accepted.

### Paid Applications

Applications that are purchased rather than freely downloaded can provide a purchase link and price:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99
}
```

The `price` field defines the application's base price and is always specified as a numeric value in **US dollars (USD)**.

LinuxToys displays the price directly in the purchase button, for example:

```text
Purchase · $19.99
```

The purchase button is visually highlighted on the application page.

#### Localized Prices

In addition to the base price in US dollars, developers can provide prices for other currencies through the `prices` field:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99,
  "prices": {
    "BRL": 59.90,
    "EUR": 17.99,
    "GBP": 15.99
  }
}
```

The keys in `prices` correspond to the international currency codes reported by the system's `locale int_curr_symbol`, such as `BRL`, `EUR`, and `GBP`.

When LinuxToys finds a currency matching the system's locale, it uses the corresponding localized price instead of the base price. The currency symbol displayed on the button is automatically obtained from `locale currency_symbol`.

For example, on a system whose locale reports `BRL`, the configuration above may be displayed as:

```text
Purchase · R$59.90
```

On a system configured for `EUR`:

```text
Purchase · €17.99
```

There is no need to add a `USD` entry to `prices`. The `price` field already represents the price in US dollars and serves as the **mandatory fallback**.

If the system's currency is not present in `prices`, if its monetary locale information cannot be determined, or if `locale` is unavailable, LinuxToys automatically uses the base USD price and the `$` symbol.

This allows an entry to provide localized pricing only for markets where it is desired:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99,
  "prices": {
    "BRL": 59.90
  }
}
```

In this example, users whose locale reports `BRL` receive the localized `R$59.90` price, while all other users receive the base `$19.99` price.

A purchase URL can also be supplied without a price:

```json
"purchase": {
  "url": "https://example.org/buy"
}
```

In this case, LinuxToys simply displays **Purchase**.

When both purchase and donation links are present, both buttons are displayed, with the purchase action receiving the primary emphasis.

### Complete Example

A more complete repository entry can therefore look like:

```json
[
  {
    "name": "Example App",
    "developer": "Example Company",
    "repo": "https://example.org",
    "category": "office",
    "icon": "./example.svg",
    "type": "url",
    "urls": {
      "appimage": "https://example.org/releases/example.AppImage"
    },
    "descriptions": "descriptions.json",
    "screenshots": "screenshots/",
    "purchase": {
      "url": "https://example.org/purchase",
      "price": 14.99,
      "prices": {
        "BRL": 44.90,
        "EUR": 12.99
      }
    },
    "donate": "https://example.org/donate"
  }
]
```
### Paid Applications and Subscriptions

Applications that require payment can provide a purchase link together with pricing information. LinuxToys distinguishes between a **one-time purchase price** and a **subscription price**, and applications may provide either or both.

#### One-Time Purchase

For applications sold through a one-time purchase, use `price`:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99
}
```

The price is specified as a numeric value in **US dollars**.

LinuxToys displays the price directly in the purchase button, for example:

```text
Purchase · $19.99
```

Localized prices can be provided using `prices`:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99,
  "prices": {
    "BRL": 59.90,
    "EUR": 17.99
  }
}
```

Each key in `prices` is an ISO currency code. LinuxToys uses the system's monetary locale to select the appropriate localized price when one is available. If no matching localized price is provided, the base `price` in US dollars is used.

Localized prices are specified directly by the developer; LinuxToys does not perform currency conversion.

#### Subscriptions

For applications offered through a subscription, use `sub_price` instead:

```json
"purchase": {
  "url": "https://example.org/subscribe",
  "sub_price": 9.99
}
```

This is displayed as a subscription action:

```text
Subscribe · $9.99
```

Subscription prices support localization in exactly the same way as one-time purchase prices, using `sub_prices`:

```json
"purchase": {
  "url": "https://example.org/subscribe",
  "sub_price": 9.99,
  "sub_prices": {
    "BRL": 29.90,
    "EUR": 8.99
  }
}
```

`sub_price` is the base subscription price in **US dollars**, while `sub_prices` provides developer-defined localized prices for other currencies.

The subscription period itself is determined by the application's purchase page. LinuxToys only displays the supplied subscription price and does not assume whether it represents a monthly, yearly, or other billing interval.

#### Offering Both Options

An application may provide both a one-time purchase and a subscription option:

```json
"purchase": {
  "url": "https://example.org/pricing",
  "price": 49.99,
  "prices": {
    "BRL": 149.90,
    "EUR": 44.99
  },
  "sub_price": 9.99,
  "sub_prices": {
    "BRL": 29.90,
    "EUR": 8.99
  }
}
```

In this case, LinuxToys displays both actions:

```text
Purchase · $49.99
Subscribe · $9.99
```

Each price is localized independently using its respective `prices` or `sub_prices` mapping.

A purchase URL can also be supplied without pricing information:

```json
"purchase": {
  "url": "https://example.org/buy"
}
```

In this case, LinuxToys simply displays **Purchase**.

When purchase, subscription, and donation options are available, LinuxToys can display them alongside the application's normal installation action, allowing developers to direct users toward the appropriate way of supporting or obtaining the application.

### Complete Example

A more complete repository entry can therefore look like:

```json
[
  {
    "name": "Example App",
    "repo": "https://example.org",
    "category": "office",
    "icon": "./example.svg",
    "type": "url",
    "urls": {
      "appimage": "https://example.org/releases/example.AppImage"
    },
    "descriptions": "descriptions.json",
    "screenshots": "screenshots/",
    "purchase": {
      "url": "https://example.org/pricing",
      "price": 14.99,
      "prices": {
        "BRL": 44.90,
        "EUR": 12.99
      },
      "sub_price": 4.99,
      "sub_prices": {
        "BRL": 14.90,
        "EUR": 4.49
      }
    },
    "donate": "https://example.org/donate"
  }
]
```

This example offers the application through both a **one-time purchase** and a **subscription**, with localized prices for Brazilian real and euro users, while also providing a separate donation option.

With the following directory structure:

```text
scripts/lists/example/
├── repository.json
├── descriptions.json
├── example.svg
└── screenshots/
    ├── 01-main.webp
    ├── 02-project.webp
    └── 03-settings.webp
```

And:

```json
{
  "description_tag": "example_desc",
  "description_long_tag": "example_long",

  "en": {
    "example_desc": "A short description of Example App.",
    "example_long": "A detailed explanation of Example App, its purpose, and the features available to the user."
  },

  "pt": {
    "example_desc": "Uma descrição curta do Example App.",
    "example_long": "Uma explicação detalhada do Example App, sua finalidade e os recursos disponíveis para o usuário."
  }
}
```

### When Is an Application Page Displayed?

An application page is automatically enabled when the entry provides at least one application-page feature:

* a long description;
* one or more valid screenshots;
* a purchase URL; or
* a donation URL.

You do not need to explicitly enable it with an additional option.

If none of these are present, selecting the application follows the standard LinuxToys installation flow.

### Checklist Behavior

Application pages only affect the activation of an individual application.

If an application is selected individually from a checklist, its application page opens normally when one is available.

When the user performs a **multi-application checklist installation**, application pages are intentionally skipped. LinuxToys proceeds with the normal batch installation flow instead, preventing several application pages from interrupting a checklist operation.

Opening an individual application's page does not clear the user's existing checklist selections.

### Recommendations

Keep the short description concise, since it is used in the normal LinuxToys interface and application-page header. Use the long description for additional context, major features, compatibility information, or other details that help the user decide whether to install the application.

For applications with translated long descriptions, prefer a repository-local `descriptions.json`. This keeps application-specific text out of the main LinuxToys translation files and allows the repository list, screenshots, icon, and descriptions to be maintained together.

Screenshots should be reasonably sized and compressed. WebP is particularly useful when several screenshots are shipped with a repository list.

An application page is optional. Simple packages that only require a name and short description should generally continue using the standard installation flow.

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
