# Repository Listings

Repository listings are LinuxToys' declarative format for software that
cannot be handled entirely through the normal AppStream catalog.

If an application is already available through Flathub or a
distribution's native repositories, it should normally be discovered and
installed through AppStream instead. Repository listings are intended
for software distributed through sources such as Git releases,
AppImages, direct download URLs, tarballs, standalone binaries, or
source repositories that need a build step.

AppStream overlays are documented separately. They extend an existing
AppStream application and should not be treated as normal repository
listings.

## Quick start

A repository listing is a JSON object. At minimum, a normal entry needs
a name, repository/source URL, category, and short description.

For an application published through GitHub releases, a minimal entry
can look like this:

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A short description of My App."
}
```

`git` is the default installation type, so `"type": "git"` may be
omitted.

LinuxToys uses the repository's release information to locate a
compatible asset for the current architecture and system. When the
automatic choice is not specific enough, `package-name` can select the
desired release asset.

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A short description of My App.",
  "package-name": "MyApp-*.AppImage"
}
```

Start with the smallest listing that correctly describes the
application. Compatibility rules, dependencies, hooks, services, and
rich app-page metadata can be added only when they are needed.

## Choosing an installation source

The most important decision in a repository listing is where LinuxToys
should obtain the application.

### Git releases

Use the default `git` type for applications distributed as release
assets from a supported Git repository.

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A portable desktop application."
}
```

The `repo` field identifies the project rather than a particular release
file. LinuxToys resolves the latest suitable release and chooses a
compatible asset.

Supported project URLs are HTTPS repositories hosted on GitHub,
Codeberg, or GitLab.

When several release assets could match, use `package-name` as an asset
selector:

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A portable desktop application.",
  "package-name": "MyApp-*.AppImage"
}
```

For `git` entries, `package-name` is optional and may be an asset name
or glob. It may also vary by operating system:

``` json
{
  "package-name": {
    "arch": "MyApp-*-arch.AppImage",
    "all": "MyApp-*.AppImage"
  }
}
```

The `all` value is the fallback when no more specific operating-system
key matches.

### AppImages

An AppImage published as a Git release normally needs no special
installation type. Use a `git` listing and, when necessary, select the
AppImage asset with `package-name`.

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A portable AppImage application.",
  "package-name": "MyApp-*.AppImage"
}
```

If the AppImage is distributed through a stable direct URL instead of a
Git release, use `url`:

``` json
{
  "name": "My App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "A portable AppImage application.",
  "type": "url",
  "urls": {
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

The `repo` field still identifies the application's project or upstream
source. The actual downloadable file belongs in `urls`.

### Direct URLs

Use `type: "url"` when LinuxToys should download a package from an
explicit URL rather than discover it from a Git release.

``` json
{
  "name": "My App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "An application distributed from a direct download.",
  "type": "url",
  "urls": {
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

The `urls` object describes the available package formats. LinuxToys can
understand these keys:

``` text
deb
rpm
pacman
pkg.tar.zst
flatpak
appimage
tar
bin
```

When more than one compatible URL is provided, LinuxToys prefers an
appropriate native package for the current distribution and then falls
back to portable formats in this order:

``` text
appimage → flatpak → tar → bin
```

This makes it possible to describe several upstream downloads in one
entry:

``` json
{
  "name": "My App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "An application with multiple upstream packages.",
  "type": "url",
  "urls": {
    "deb": "https://example.com/download/my-app.deb",
    "rpm": "https://example.com/download/my-app.rpm",
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

For applications already distributed through normal native repositories
or Flathub, prefer AppStream instead of recreating that distribution
path as a repository listing.

#### Dynamic download URLs

Some projects generate a download URL dynamically. A URL value may
reference an environment variable exported by a pre-install hook:

``` json
{
  "type": "url",
  "urls": {
    "appimage": {
      "env": "DOWNLOAD_URL"
    }
  },
  "overrides": {
    "pre": "export DOWNLOAD_URL=\"https://example.com/generated/MyApp.AppImage\""
  }
}
```

The environment-variable form is accepted only when a valid pre-install
hook is present. Use it only when a normal stable URL cannot describe
the download.

### Tarballs

Use `type: "tar"` for a tarball published as a Git release:

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "An application distributed as a release archive.",
  "type": "tar",
  "package-name": "my-app-*.tar.gz",
  "overrides": {
    "post": {
      "script": "my-app/post-install.sh"
    }
  }
}
```

For a tarball hosted at a direct URL, use `type: "url"` instead:

``` json
{
  "name": "My App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "An application distributed as a downloadable archive.",
  "type": "url",
  "urls": {
    "tar": "https://example.com/download/my-app.tar.gz"
  },
  "overrides": {
    "post": {
      "script": "my-app/post-install.sh"
    }
  }
}
```

A tarball installation only extracts the application files. LinuxToys
therefore requires a valid post-install hook for tarball entries. Use
that hook for the integration the extracted application needs, such as
creating a launcher, desktop entry, or symlink.

### Standalone binaries

Use `type: "bin"` for a standalone executable published as a Git
release.

``` json
{
  "name": "My Tool",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "A standalone command-line utility.",
  "type": "bin",
  "package-name": "my-tool"
}
```

Unlike the selector used by `git` and `tar`, a `bin` entry requires an
exact asset filename. Globs and paths are not accepted.

For a directly hosted binary, use `url`:

``` json
{
  "name": "My Tool",
  "repo": "https://example.com/my-tool",
  "category": "utilities",
  "description": "A standalone command-line utility.",
  "type": "url",
  "urls": {
    "bin": "https://example.com/download/my-tool"
  }
}
```

### Building with Make

Use `type: "make"` when the application must be built from source with
`make`.

By default, LinuxToys clones the Git repository, builds it, and uses
`sudo make install` as the installation command.

``` json
{
  "name": "My Tool",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "A tool built from source.",
  "type": "make"
}
```

A custom install target can be declared with `make-command`:

``` json
{
  "type": "make",
  "make-command": "make install-user"
}
```

The command must contain an install target that LinuxToys can map to its
corresponding uninstall target during reversion.

A Make build can also use a release tarball instead of a Git clone:

``` json
{
  "name": "My Tool",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "A tool built from a release tarball.",
  "type": "make",
  "make-source": "tar",
  "package-name": "my-tool-*.tar.gz"
}
```

`make-source` accepts `git` or `tar`.

## Required fields

Normal repository entries require these fields:

### `name`

The display name shown by LinuxToys.

``` json
"name": "My App"
```

Names must be unique across repository listings. LinuxToys also derives
a stable internal repository-app ID from the display name.

### `repo`

The application's upstream project or source URL.

``` json
"repo": "https://github.com/example/my-app"
```

For Git-backed installation types, this is also the repository LinuxToys
uses to discover releases or obtain source code.

For `url` entries, the downloadable package URLs belong in `urls`;
`repo` remains the application's upstream identity.

### `category`

The LinuxToys category in which the application should appear.

``` json
"category": "utilities"
```

Use an existing LinuxToys category identifier.

### `description`

A short description shown in application lists and search results.

``` json
"description": "A fast and lightweight desktop utility."
```

A localized description catalog can be used instead of keeping all
translated descriptions directly in the listing. See
[Localization](#localization).

## Compatibility

Repository listings are filtered before they are shown to the user. Add
compatibility restrictions only when the application or installation
method genuinely requires them.

### Operating systems

Use `os` to restrict an entry to particular LinuxToys compatibility
families:

``` json
"os": ["debian", "ubuntu", "fedora"]
```

Supported keys are:

``` text
debian
ubuntu
cachy
arch
steamos
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

Positive values form an allow-list.

Exclusions begin with `!`:

``` json
"os": ["!steamos"]
```

An exclusion-only list means "all supported systems except these." If
positive and negative values are mixed, exclusions always take
precedence.

Do not include and exclude the same key in one declaration.

### Per-OS installation types

The `type` field can be a mapping when an application genuinely needs
different installation methods on different systems:

``` json
"type": {
  "arch": "make",
  "all": "git"
}
```

`all` is the fallback. Prefer a single installation type whenever
possible.

### Per-OS package or asset names

Fields that resolve package names or release selectors can also use
operating-system mappings where supported:

``` json
"package-name": {
  "arch": "MyApp-*-arch.AppImage",
  "all": "MyApp-*.AppImage"
}
```

### Desktop environments

Use `desktop` when an application or integration is meaningful only for
particular desktop families:

``` json
"desktop": ["gnome", "plasma"]
```

Supported values are:

``` text
gnome
plasma
hyprland
sway
other
```

### systemd

Use `systemd` only when the installation requires or explicitly excludes
systemd:

``` json
"systemd": "yes"
```

or:

``` json
"systemd": "no"
```

Omitting the field is neutral.

Declaring services also implicitly requires a systemd-capable
environment.

### WSL

Use `wsl` to make an entry WSL-only or non-WSL-only:

``` json
"wsl": "yes"
```

or:

``` json
"wsl": "no"
```

### Containers

Repository entries are allowed in supported containers by default.

To explicitly prevent an entry from appearing in containers:

``` json
"container": "deny"
```

Accepted values are `allow` and `deny`.

Flatpak and AppImage installation flows are not supported inside
containers regardless of this setting.

### Hardware

Hardware requirements can be expressed with `gpu` and `cpu`
compatibility values:

``` json
"hardware": {
  "gpu": ["amd"],
  "cpu": ["x86_64"]
}
```

Only use hardware restrictions when the application or installation
method actually depends on them.

## Dependencies

Dependencies are installed before the main application.

A dependency is either `native` or `flathub`.

### Native dependencies

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": "ffmpeg"
  }
]
```

Native package names may vary by operating system:

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": {
      "debian": "libexample-dev",
      "fedora": "example-devel",
      "arch": "example"
    }
  }
]
```

A package name may also be a list when several packages are required:

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": ["git", "curl", "make"]
  }
]
```

### Flathub dependencies

``` json
"dependencies": [
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

Flathub dependencies require a compatible systemd environment.

Dependencies are for prerequisites of the repository-listed application.
They do not turn ordinary native or Flathub applications into repository
listings.

## Installation hooks and overrides

The optional `overrides` object customizes the generated installation
flow.

Supported keys for normal repository listings are:

``` text
pre
post
flatpak
skip-user
```

### Pre-install hooks

A pre-install hook runs before the generated installation commands.

For a small operation, inline shell can be used:

``` json
"overrides": {
  "pre": "export EXAMPLE_MODE=1"
}
```

For more substantial logic, reference a script stored with the
repository list:

``` json
"overrides": {
  "pre": {
    "script": "my-app/pre-install.sh"
  }
}
```

Script paths are resolved relative to the JSON file and must remain
inside the repository-list tree.

### Post-install hooks

Post-install hooks use the same syntax:

``` json
"overrides": {
  "post": {
    "script": "my-app/post-install.sh"
  }
}
```

They run after the generated installation commands.

Tarball installations require a post-install hook because extracting an
archive alone does not define how the application should be integrated
with the user's system.

### Flatpak permission overrides

A repository listing may declare Flatpak permission overrides when its
installation flow includes Flatpak content.

``` json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-download",
      "target": "org.example.App"
    }
  ]
}
```

Supported scopes are `user` and `system`.

Supported override types include:

``` text
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

### Forcing system Flatpak scope

When a listing's Flatpak operations must not use user scope:

``` json
"overrides": {
  "skip-user": true
}
```

Use this only when system scope is genuinely required.

## Services

Repository listings can ask LinuxToys to enable and start systemd
services after installation.

A string or list defaults to system scope:

``` json
"services": "my-app.service"
```

``` json
"services": [
  "my-app.service",
  "my-app-helper.service"
]
```

For explicit system and user services:

``` json
"services": {
  "system": ["my-app.service"],
  "user": ["my-app-session.service"]
}
```

If a unit suffix is omitted, `.service` is added automatically.

System services require privilege escalation. User services are enabled
through the user's systemd instance.

## App page metadata

Repository-listed applications can have the same kind of rich
presentation users expect from the rest of LinuxToys.

These fields affect presentation and do not replace the installation
source.

### Developer

``` json
"developer": "Example Software"
```

When possible, LinuxToys can derive a developer/project namespace from
the repository URL if this field is omitted.

### License

``` json
"license": "GPL-3.0"
```

Keep the displayed license identifier concise.

### Long descriptions

A long description may be written directly:

``` json
"long-description": "My App provides a complete workflow for..."
```

For substantial content, point to a Markdown file stored with the
repository listing:

``` json
"long-description": "my-app/description.md"
```

Markdown paths are resolved relative to the JSON file and may use
subdirectories, but they cannot escape the listing's directory tree.

### Screenshots

`screenshots` can point to one image:

``` json
"screenshots": "my-app/screenshots/main.webp"
```

or several:

``` json
"screenshots": [
  "my-app/screenshots/main.webp",
  "my-app/screenshots/settings.webp"
]
```

It may also point to a directory. LinuxToys will load supported image
files from that directory in sorted order.

Supported screenshot formats are PNG, JPEG, WebP, and SVG.

### Icons

A simple icon value can refer to a GTK icon name or the existing
LinuxToys icon resolver:

``` json
"icon": "applications-utilities"
```

A repository listing can also use an SVG or PNG stored below
`scripts/lists/`:

``` json
"icon": "my-app/icon.svg"
```

Relative icon paths are resolved from the JSON file containing the entry
and cannot escape the repository-list tree.

### Donations

``` json
"donate": "https://example.com/donate"
```

A donation URL can also be represented as an object containing `url`.

### Purchases and subscriptions

Commercial metadata can add purchase and subscription actions to a
repository application's app page.

A one-time purchase:

``` json
"purchase": {
  "url": "https://example.com/buy",
  "price": 49.99
}
```

A subscription:

``` json
"purchase": {
  "url": "https://example.com/subscribe",
  "sub_price": 9.99
}
```

Both may be offered together:

``` json
"purchase": {
  "url": "https://example.com/pricing",
  "price": 49.99,
  "sub_price": 9.99
}
```

LinuxToys also supports localized prices, purchase tiers, subscription
tiers, and subscription periods. Keep simple commercial models simple;
use the tiered forms only when the application's actual pricing requires
them.

Commerce metadata describes the developer's own purchase or subscription
destination. LinuxToys does not become the payment processor.

## Localization

Repository listings can keep their user-facing descriptions separate
from installation metadata.

A listing may reference a sibling JSON description catalog:

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "descriptions": "descriptions.json",
  "description_tag": "my_app_desc",
  "long-description_tag": "my_app_long"
}
```

The description catalog must be a JSON file in the same directory as the
repository-list JSON.

A catalog can provide translations such as:

``` json
{
  "description_tag": "my_app_desc",
  "description_long_tag": "my_app_long",
  "en": {
    "my_app_desc": "A short description.",
    "my_app_long": "my-app/description.md"
  },
  "pt-BR": {
    "my_app_desc": "Uma descrição curta.",
    "my_app_long": "my-app/description-br.md"
  }
}
```

For localized catalog entries, LinuxToys first looks for the current
locale, then its base language, then English.

A localized long-description value may itself point to a Markdown file,
allowing each language to use a separate document.

## A complete example

The following example combines a Git-hosted AppImage with compatibility
metadata, a native dependency, rich app-page content, and a donation
link:

``` json
{
  "name": "My App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "A portable desktop application.",
  "package-name": "MyApp-*.AppImage",
  "icon": "my-app/icon.svg",
  "license": "GPL-3.0",
  "os": ["!steamos"],
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "ubuntu": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper",
        "all": "example-helper"
      }
    }
  ],
  "long-description": "my-app/description.md",
  "screenshots": "my-app/screenshots",
  "donate": "https://example.com/donate"
}
```

Do not use a complete example as a template that must be filled out.
Most applications should need far fewer fields.

## Repository layout

Repository-list resources can live beside the JSON that declares them.

For example:

``` text
scripts/
└── lists/
    └── my-app/
        ├── app.json
        ├── descriptions.json
        ├── icon.svg
        ├── description.md
        ├── description-br.md
        ├── pre-install.sh
        ├── post-install.sh
        └── screenshots/
            ├── main.webp
            └── settings.webp
```

Local icons, screenshots, Markdown descriptions, and hook scripts are
resolved with path-safety checks so entries cannot escape the
repository-list tree.

## Design guidelines

Prefer declarative metadata over shell logic. If LinuxToys already has a
field for what the application needs, use that field rather than
reproducing the behavior in a hook.

Prefer AppStream for applications already represented by Flathub or
supported native repositories.

Prefer Git release discovery when upstream publishes suitable release
assets. Use direct `url` entries when the download location must be
declared explicitly.

Use compatibility restrictions only when necessary. An unnecessarily
narrow `os`, desktop, hardware, WSL, container, or systemd declaration
prevents otherwise compatible users from seeing the application.

Use pre/post hooks for the parts of installation that cannot be
expressed declaratively. Keep those hooks small and auditable.

Treat the app page as part of the integration. A useful description,
screenshots, license information, and developer links help users
understand an application before installing it.

## Field reference

| Field | Purpose |
| --- | --- |
| `name` | Application display name. |
| `repo` | Upstream project/source URL. |
| `category` | LinuxToys category identifier. |
| `description` | Short application description. |
| `type` | Installation type; defaults to `git`. |
| `package-name` | Package name, exact binary asset, or release asset selector depending on `type`. |
| `urls` | Direct package/download URLs for `url` entries. |
| `make-source` | `git` or `tar` source for a `make` entry. |
| `make-command` | Custom Make installation command. |
| `os` | Operating-system compatibility allow/exclude rules. |
| `desktop` | Desktop-environment compatibility. |
| `systemd` | Require or exclude systemd. |
| `wsl` | WSL-only or non-WSL-only restriction. |
| `container` | Allow or deny container execution. |
| `hardware` | CPU/GPU compatibility requirements. |
| `dependencies` | Native or Flathub prerequisites. |
| `overrides` | Pre/post hooks, Flatpak overrides, and Flatpak scope behavior. |
| `services` | systemd system/user units to enable and start. |
| `icon` | GTK/LinuxToys icon or repository-local SVG/PNG. |
| `license` | Short license identifier. |
| `developer` | Developer or company displayed on the app page. |
| `long-description` | Rich description text or repository-local Markdown file. |
| `screenshots` | Repository-local screenshots or screenshot directory. |
| `donate` | Donation destination. |
| `purchase` | Purchase/subscription metadata. |
| `descriptions` | Sibling localization catalog. |
| `description_tag` | Localization key for the short description. |
| `long-description_tag` | Localization key for the long description. |

AppStream overlay fields are intentionally outside the scope of this
document.
