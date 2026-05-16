# (IN DEVELOPMENT) How to rebuild the Ubuntu kernel from source

This guide explains how to properly rebuild and install the Ubuntu kernel from source, including workarounds for common nuances and edge cases.

<https://gitlab.com/brlin/ubuntu-kernel-rebuild-howto>  
[![The GitLab CI pipeline status badge of the project's `main` branch](https://gitlab.com/brlin/ubuntu-kernel-rebuild-howto/badges/main/pipeline.svg?ignore_skipped=true "Click here to check out the comprehensive status of the GitLab CI pipelines")](https://gitlab.com/brlin/ubuntu-kernel-rebuild-howto/-/pipelines) [![GitHub Actions workflow status badge](https://github.com/brlin-tw/ubuntu-kernel-rebuild-howto/actions/workflows/check-potential-problems.yml/badge.svg "GitHub Actions workflow status")](https://github.com/brlin-tw/ubuntu-kernel-rebuild-howto/actions/workflows/check-potential-problems.yml) [![pre-commit enabled badge](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white "This project uses pre-commit to check potential problems")](https://pre-commit.com/) [![REUSE Specification compliance badge](https://api.reuse.software/badge/gitlab.com/brlin/ubuntu-kernel-rebuild-howto "This project complies to the REUSE specification to decrease software licensing costs")](https://api.reuse.software/info/gitlab.com/brlin/ubuntu-kernel-rebuild-howto)

## Preface

I have to rebuild my Ubuntu kernel for troubleshooting a screen hang problem on my Framework Laptop 13 (AMD 7040 series).  This guide documents the process.

## Note

This process is tested on Ubuntu 26.04, your mileage may vary on other versions of Ubuntu.

## Enable the source portion of the Ubuntu archive

To acquire the source package of the running Ubuntu kernel, we must first enable the source portion of the Ubuntu archive.

1. Edit the /etc/apt/sources.list.d/ubuntu.sources file _as root_, append `deb-src` to the value of the `Types` fields(separated by space).
1. Run the following command in a terminal _as root_ to update the Ubuntu archive index:

    ```bash
    apt update
    ```

## Prepare a working directory for the process

Create a new folder as the working directory of this process.  The path of this folder _must not contain spaces_ (and ideally, any characters that is non-alpha-numeric or dash) to prevent triggering compatibility issues with the build system.

Run the following command in the terminal to change the working directory to the newly created folder:

```bash
cd /path/to/working/directory
```

Replace the `/path/to/working/directory` placeholder text with the actual path of the newly created folder.

## Acquire source package of the Ubuntu kernel

1. Run the following command in the terminal to query the version of the running kernel:

    ```bash
    uname -r
    ```

    The output should be something like `7.0.0-15-generic`.

1. Run the following commands in the terminal to acquire the source package of the current running Ubuntu kernel:

    ```bash
    apt source linux-image-_kernel version_
    ```

   Replace the `_kernel version_` placeholder text with kernel version string acquired from the previous step.

   This will download the source package, and extract it in the current directory.

**NOTE:** If you're already build and running a custom kernel, the `uname -r` command may not return a version that has a corresponding source package in the Ubuntu archive.  In this case, you can manually specify the version of the kernel you want to acquire the source package of, by replacing `$(uname -r)` text with the version string.

## Install metapackage build dependencies

To allow clean removal of the build dependencies after the process, we will install the build dependencies of the kernel source package via a metapackage.

Run the following command in the terminal _as root_ to install the software for generate such metapackages:

```bash
apt install devscripts equivs
```

## Apply the modifications

Apply whatever modifications you want to the source code.

**NOTE:** If you want to apply patches that is delivered in the mbox format, you can directly use the following command to do so:

```bash
patch --strip=1 < /path/to/mbox/file
```

## Change the working directory to the extracted source code folder

This simplifies the instructions in the following steps.

Run the following command in the terminal to change the working directory to the extracted source code folder:

```bash
cd linux-_upstream version_
```

Replace the `_upstream version_` placeholder text with the upstream version string of the kernel, for example, the `7.0.0-15.15` Ubuntu kernel version has an upstream version of `7.0.0`.

## Prepare the kernel source

Run the following command in the terminal to prepare the kernel source for building:

```bash
chmod a+x debian/scripts/* && \
    chmod a+x debian/scripts/misc/* && \
    fakeroot debian/rules clean
```

## Add an entry to the Debian changelog

Before building the kernel, we must add an entry to the Debian changelog.  This is required to properly version the resulting kernel packages.

1. Run the following command in the terminal to add an entry to the Debian changelog:

    ```bash
    debchange --increment --changelog debian.master/changelog
    ```

   **NOTE:** Kernel package doesn't use the debian/changelog file.

   a text editor will open to edit the new changelog entry.
1. In the version field of the new changelog entry:
    + Change the number right after the `-` character(the ABI number) to `999` to differentiate the custom kernel from the kernel released by Canonical.
    + Rename the `ubuntuN` part to something that properly indicates the nature of the modifications, for example, `custom1`.
1. Replace the `UNRELEASED` text in the changelog entry to the codename of the current Ubuntu release.
1. Write a proper changelog message in the message field of the changelog entry, for example, `Rebuild the kernel with custom modifications.`.
1. Save the file and exit the text editor.

## References

The following materials are referenced during the writing of this guide:

* [How to build an Ubuntu Linux kernel - Ubuntu Kernel documentation](https://documentation.ubuntu.com/kernel/how-to/develop-customise/build-kernel/)  
  Explains the general process of building an Ubuntu kernel.
* [OPTIONS — dch(1) — devscripts — Debian bookworm — Debian Manpages](https://manpages.debian.org/bookworm/devscripts/dch.1.en.html#OPTIONS)  
  Explains how to use the `debchange` command to add an entry to the Debian changelog.

## Licensing

Unless otherwise noted([comment headers](https://reuse.software/spec-3.3/#comment-headers)/[REUSE.toml](https://reuse.software/spec-3.3/#reusetoml)), this product is licensed under [the 4.0 version of the Creative Commons Attribution-ShareAlike license](https://creativecommons.org/licenses/by-sa/4.0/), or any of its more recent versions of your preference.

This work complies to [the REUSE Specification](https://reuse.software/spec/), refer to the [REUSE - Make licensing easy for everyone](https://reuse.software/) website for info regarding the licensing of this product.
