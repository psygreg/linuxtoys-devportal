# LinuxToys Developer Documentation

LinuxToys is a Linux application designed to make installing, configuring and managing software and system features simple for users across a wide range of Linux distributions.

For developers, LinuxToys provides a distribution layer that can take care of much of the Linux-specific work normally required to deliver software to users: distribution and hardware compatibility, package formats, dependencies, installation procedures, updates, system integration and removal.

You do **not** need to learn the entire LinuxToys development interface to distribute an application through it.

LinuxToys supports everything from simple application packages to complex system features, so its developer APIs include tools for many specialized situations and quirks. Most applications only need a small subset of these capabilities.

The first step is choosing the appropriate integration method.

---

<a id="multi-distro"></a>

## What LinuxToys Provides

Linux software distribution can quickly become complicated when an application needs to support several distributions, package managers, desktop environments or hardware configurations.

LinuxToys provides common infrastructure for handling these differences while presenting users with a consistent installation experience.

Depending on the integration, LinuxToys can handle tasks such as:

* detecting the user's distribution and system characteristics;
* restricting features to compatible systems;
* selecting appropriate packages for different distributions;
* installing native packages, Flatpaks, AppImages and other release formats;
* downloading packages directly from upstream releases;
* installing dependencies;
* running pre-installation and post-installation procedures;
* integrating applications into the LinuxToys interface;
* tracking changes made during installation;
* reverting supported installations and system changes;
* managing services and other system components;
* handling distribution-specific package management differences;
* supporting hardware- or desktop-specific features.

Many of these facilities exist because LinuxToys also distributes drivers, system utilities, compatibility layers, performance features and other software that requires considerably more system integration than a typical desktop application.

**A normal application package will usually need only a fraction of them.**

---

## Choosing an Integration Method

LinuxToys provides two primary ways for developers to integrate software.

### Repository Lists

**Start here for most applications.**

Repository Lists provide a declarative way to describe software that LinuxToys can install. Instead of writing an installation script, you describe the application, where it comes from, what package should be installed and any relevant compatibility requirements.

They are well suited to applications that are already distributed through conventional channels, including applications with:

* native packages for one or more Linux distributions;
* Flatpak releases;
* AppImage releases;
* packages published through upstream release systems;
* packages hosted directly by the developer;
* straightforward dependencies;
* simple compatibility requirements;
* small pre-installation or post-installation steps.

Repository Lists allow LinuxToys to perform the installation using its existing infrastructure rather than requiring every developer to implement package management and system detection independently.

They can also express more advanced conditions when necessary, including distribution, architecture, CPU, GPU, desktop environment and other compatibility requirements.

For many applications, a Repository List entry may be the **only LinuxToys-specific integration code you need to maintain**.

### Core Library and Full Scripts

Use a full LinuxToys script when your software requires a procedure that cannot be adequately described as a package installation.

LinuxToys' Core Library provides reusable Bash functions and system information for these more complex integrations.

This is appropriate for features that need to perform operations such as:

* complex installation or migration procedures;
* extensive system configuration;
* conditional operations based on the host system;
* filesystem changes that need to be tracked for removal;
* systemd service management;
* bootloader or boot configuration changes;
* unusual package-management procedures;
* multiple dependent installation stages;
* calling other LinuxToys features;
* custom rollback or cleanup behavior;
* specialized handling for distribution or hardware quirks.

The Core Library exists so these scripts can use the same compatibility detection, package-management abstraction, transaction tracking and system integration infrastructure used by LinuxToys itself.

A full script is therefore considerably more powerful than a Repository List—but most application developers **do not need that additional complexity**.

---

<a id="official-support"></a>

## Oferring Official Support

Developers may also contact the LinuxToys project to request **official support** status for their applications.

Officially supported applications are identified as such within LinuxToys, distinguishing integrations maintained in collaboration with their upstream developers. This status also creates a more direct channel between LinuxToys users and the project responsible for the application.

Advantages of official support include:

* visual identification of the application as officially supported within LinuxToys;
* greater visibility and easier recognition by users;
* more efficient forwarding of application-specific issues to the upstream developer;
* the ability to receive relevant information collected by the LinuxToys bug reporting system, making it easier to investigate issues encountered by users;
* closer collaboration with the LinuxToys project to maintain and improve the integration over time.

Official support **does not require an application to use every LinuxToys integration feature**. An application distributed through a simple Repository List can receive official support just as a more complex integration can.

If you are interested in providing official support for your application through LinuxToys, inform the project maintainers in your *pull request*.

---

## Which Documentation Should I Read?

The easiest way to decide is to start from what you are trying to distribute.

| Your application or feature...                                       | Start with           |
| -------------------------------------------------------------------- | -------------------- |
| Is already available as ordinary Linux packages                      | **Repository Lists** |
| Is distributed as an AppImage or Flatpak                             | **Repository Lists** |
| Publishes installable packages through releases or a download server | **Repository Lists** |
| Needs dependencies installed before the main package                 | **Repository Lists** |
| Needs simple commands before or after installation                   | **Repository Lists** |
| Has different packages for different distributions                   | **Repository Lists** |
| Should only appear on certain hardware, distributions or desktops    | **Repository Lists** |
| Requires a substantial custom installation procedure                 | **Core Library**     |
| Makes extensive changes to the operating system                      | **Core Library**     |
| Needs detailed transaction and rollback handling                     | **Core Library**     |
| Manages services, boot configuration or other system components      | **Core Library**     |
| Cannot reasonably be represented as a package plus optional hooks    | **Core Library**     |

<a id="declarative-deployment"></a>

### → [Repository Lists Documentation](repositorylists.html)

Learn how to describe an application declaratively, define its packages and dependencies, specify compatibility requirements, provide metadata and icons, use supported distribution methods, and add optional installation hooks.

**This is the recommended starting point for application developers.**

### → [Core Library Documentation](corelibraries.html)

Learn how to build complete LinuxToys installation scripts using its compatibility, package-management, filesystem, systemd, boot, system-information and transaction-management facilities.

**Use this when a Repository List is not sufficient for your integration.**

> The filenames above may be adjusted to match the final documentation layout.

---

## You Probably Don't Need Everything

Both documentation sets describe capabilities used throughout LinuxToys itself.

This means you will encounter options that may have little or nothing to do with your application.

For example, LinuxToys needs to support software ranging from ordinary desktop applications to GPU tooling, drivers, bootloader modifications, compatibility components and low-level system configuration. The infrastructure required for those features is exposed so developers can solve similar problems when necessary.

It is **not** a checklist of things every integration should implement.

A developer distributing a conventional application may only need to provide:

1. application metadata;
2. the upstream repository or package source;
3. the appropriate package name or release format; and
4. any compatibility restrictions that genuinely apply.

Everything else can be ignored until your application actually requires it.

The same principle applies to full scripts: use the Core Library functions that solve the problem at hand rather than attempting to incorporate every facility LinuxToys provides.

---

## Prefer the Simplest Integration

When several approaches could accomplish the same result, prefer the one that delegates more work to LinuxToys.

A declarative Repository List is generally preferable to reproducing the same installation procedure in a custom script. It is easier to review, easier to maintain and allows improvements to LinuxToys' package handling and compatibility infrastructure to benefit your application automatically.

Likewise, when a full script is necessary, prefer LinuxToys' Core Library functions over directly implementing distribution-specific commands whenever an appropriate abstraction already exists.

This keeps integrations consistent with the rest of LinuxToys and reduces the amount of distribution-specific behavior developers need to maintain themselves.

---

## Where to Start

For most developers:

**Application → Repository List → LinuxToys handles the installation**

Start with the **[Repository Lists Documentation](repositorylists.html)** and implement only the fields relevant to your application.

For complex system-level integrations:

**Feature → LinuxToys script → Core Library → system**

Start with the **[Core Library Documentation](corelibraries.html)** and use the libraries relevant to the operations your feature needs to perform.

You can always move to a more advanced integration later if the requirements of your software grow.

The goal is not to make developers learn every internal capability of LinuxToys. The goal is to provide enough infrastructure that developers only need to implement the parts that are genuinely specific to their software.
