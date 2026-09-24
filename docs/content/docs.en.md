# LinuxToys Developer Documentation

LinuxToys can integrate applications through several different paths. For most desktop applications, the right starting point is **AppStream**.

> **If your application is available through Flathub or a supported distribution's native repositories, this is the only integration guide you need.**

LinuxToys discovers those applications through AppStream and turns their existing metadata into LinuxToys app pages and installation entries. You do not need to create a separate LinuxToys package definition simply to make the same application available through LinuxToys.

## Choose your integration path

### Your application is available through AppStream

Stay on this page if your application is available through **Flathub** or a supported distribution repository and has usable AppStream metadata. LinuxToys discovers the application, places it in the appropriate category, presents its metadata, and installs it from sources available on the user's system.

You only need LinuxToys-specific metadata when you want to extend that existing AppStream entry with an **AppStream overlay**.

### Your application is distributed independently

Use [Repository Lists](repositorylists.html) when the application is not adequately provided through AppStream and LinuxToys needs its own installation definition.

Repository Lists cover software distributed through sources such as Git releases, AppImages, direct download URLs, tarballs, standalone binaries, or source repositories requiring a build step.

### You are integrating a LinuxToys or system feature

Use [Core Libraries](corelibraries.html) when implementing a system-level LinuxToys feature or custom procedure that belongs in LinuxToys itself rather than describing a distributable application.

---

## AppStream applications

AppStream is the preferred integration path for ordinary applications already distributed through established Linux software sources.

LinuxToys consumes AppStream catalogs from supported native repositories and Flathub, adapts their components to the LinuxToys catalog, and applies its compatibility and source-selection rules. The application remains defined by the software sources where it is actually distributed.

### What LinuxToys gets from AppStream

Good upstream AppStream metadata can provide LinuxToys with the information needed to present an application without a separate LinuxToys listing, including:

- application name and summary;
- icon;
- developer information;
- license;
- homepage and other project links;
- long description;
- screenshots;
- categories and application type;
- release information and other AppStream metadata.

The exact information shown depends on what is available in the AppStream component. Maintaining complete, accurate upstream AppStream metadata is therefore the first step toward a good LinuxToys listing.

### Categories

LinuxToys maps standard AppStream/Desktop Menu categories into its own catalog. Both broad categories and more specific additional categories are considered.

Describe the application accurately using the normal AppStream category system rather than adding LinuxToys-specific categories to upstream metadata.

### Native packages and Flathub

The same AppStream application may be available from more than one installation source. LinuxToys can combine those representations into one application experience and select an appropriate source according to its source-selection rules. Where applicable, the app page can expose a source selector.

Do **not** create a Repository List merely because your application exists both natively and on Flathub. Let AppStream represent the application and LinuxToys handle the available sources.

### App pages

AppStream applications are presented through LinuxToys app pages. Depending on available metadata and application state, a page can include its description, screenshots, developer and license information, project links, ratings, installation source, installation controls, and an **Open** action for an installed application that can be launched.

![app page](/assets/app-page.webp)

LinuxToys also tracks installed state and integrates AppStream installations with its installation queue and removal flow.

---

## AppStream overlays

Most AppStream applications need no LinuxToys-specific definition.

An **AppStream overlay** adds reviewed, application-specific information or installation behavior on top of an existing AppStream component. It **does not create a new application** and does not replace its normal AppStream installation source.

The target is declared with `appstream-name`:

```json
{
  "appstream-name": "org.example.App"
}
```

Use the application's AppStream component ID. A trailing `.desktop` is normalized by LinuxToys and is not required.

AppStream overlays intentionally support only:

```text
appstream-name
purchase
dependencies
overrides
```

They are not general-purpose Repository Lists.

### Dependencies

An overlay can declare prerequisites installed as part of the AppStream application's installation flow.

#### Native dependencies

```json
{
  "appstream-name": "org.example.App",
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "ubuntu": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper"
      }
    }
  ]
}
```

Native package names may vary by distribution. A package declaration can also contain multiple package names.

#### Flathub dependencies

```json
{
  "appstream-name": "org.example.App",
  "dependencies": [
    {
      "type": "flathub",
      "package-name": "org.example.Runtime"
    }
  ]
}
```

Dependencies augment the normal AppStream installation; they do not replace its selected native or Flatpak source.

### Pre- and post-install hooks

Reviewed hooks can run immediately before or after the normal AppStream installation:

```json
{
  "appstream-name": "org.example.App",
  "overrides": {
    "pre": {
      "script": "example/pre-install.sh"
    },
    "post": {
      "script": "example/post-install.sh"
    }
  }
}
```

Hook scripts are repository-local resources resolved within the Repository List tree. Keep hooks small and use them only for behavior that cannot be represented declaratively.

### Flatpak permission overrides

```json
{
  "appstream-name": "org.example.App",
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
}
```

Supported scopes are `user` and `system`. Supported override types are:

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

AppStream overlays deliberately do **not** support Repository List source-selection controls such as `skip-user`. AppStream source and scope selection remain owned by the AppStream installation flow.

### Purchases and subscriptions

An overlay can add purchase or subscription actions to an existing AppStream app page.

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/buy",
    "price": 49.99
  }
}
```

For a subscription:

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/subscribe",
    "sub_price": 9.99
  }
}
```

Both may be declared together. LinuxToys also supports the tiered purchase/subscription metadata used by Repository Lists when required by the application's pricing model.

These actions point to the developer's or company's own destination. LinuxToys does not become the payment processor.

### Combining overlay features

The supported fields can be combined:

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/pricing",
    "price": 49.99
  },
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper"
      }
    }
  ],
  "overrides": {
    "post": {
      "script": "example/post-install.sh"
    }
  }
}
```

Start with no overlay at all. Add one only when the existing AppStream entry needs LinuxToys-specific behavior.

---

## LinuxToys install links

LinuxToys exposes a URI scheme that lets websites, documentation and other applications send an installation request directly to LinuxToys.

For an AppStream application, use its stable AppStream component ID:

```text
linuxtoys://install/org.example.App
```

This is the recommended form for AppStream applications.

```html
<a href="linuxtoys://install/org.example.App">
  Install with LinuxToys
</a>
```

LinuxToys resolves the requested target against applications available and compatible on that system and continues through its normal installation experience. Installation links can also target supported curated LinuxToys entries, but AppStream IDs are preferable for AppStream applications because they provide a stable identity independent of the displayed name.

### Install with LinuxToys button

Use the LinuxToys URI behind an installation button on your project's website, download page or documentation.

#### Ready-to-go English button

<img src="/assets/installwithlinuxtoys_en.webp"
     alt="Install with LinuxToys"
     data-no-theme-image>

This space is reserved for the official English button asset and its ready-to-copy integration snippet.

#### Editable blank button

<img src="/assets/installwithlinuxtoys_base.webp"
     alt="Install with LinuxToys"
     data-no-theme-image>

This space is reserved for the blank button asset intended for localized or otherwise appropriate labels.

```html
<a href="linuxtoys://install/org.example.App">
  <img src="install-with-linuxtoys.svg" alt="Install with LinuxToys">
</a>
```

Replace `org.example.App` with the application's real AppStream component ID.

---

## Applications outside AppStream

If LinuxToys cannot obtain and install the application through an appropriate AppStream source, use a **Repository List**.

Repository Lists describe applications distributed through Git releases, AppImages, direct URLs, tarballs, standalone binaries, source builds and other supported distribution methods. They can also define compatibility rules, dependencies, hooks, services and rich app-page metadata.

Continue with the [Repository Lists documentation](repositorylists.html).

Do not create a Repository List for an ordinary application already adequately represented through AppStream unless LinuxToys genuinely needs a separate installation path.

---

## System integrations and LinuxToys features

Not every LinuxToys integration represents an application.

For a system tweak, driver workflow, maintenance operation, platform integration or another feature whose installation logic belongs directly to LinuxToys, use the **Core Libraries** and LinuxToys' script integration model.

Continue with the [Core Libraries documentation](corelibraries.html).

The distinction is intentional:

- **AppStream** describes applications already distributed through supported software sources.
- **Repository Lists** describe applications LinuxToys must obtain or install through another supported distribution method.
- **Core Libraries** are for LinuxToys-native procedures and system integrations.
