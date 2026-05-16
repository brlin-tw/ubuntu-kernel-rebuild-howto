# How to rebuild the Ubuntu kernel from source

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

## Acquire source package of the Ubuntu kernel

Run the following commands in the terminal to acquire the source package of the current running Ubuntu kernel:

```bash
if ! kernel_version="$(uname -r)"; then
    printf \
        'Error: Unable to query the version of the running kernel.\n' \
        1>&2
else
    if ! apt source "linux-image-${kernel_version}"; then
        printf \
            'Error: Unable to acquire the source package of the running kernel.\n' \
            1>&2
    fi
fi
```

This will download the source package, and extract it in the current directory.

**NOTE:** If you're already build and running a custom kernel, the `uname -r` command may not return a version that has a corresponding source package in the Ubuntu archive.  In this case, you can manually specify the version of the kernel you want to acquire the source package of, by replacing `$(uname -r)` text with the version string.

## Licensing

Unless otherwise noted([comment headers](https://reuse.software/spec-3.3/#comment-headers)/[REUSE.toml](https://reuse.software/spec-3.3/#reusetoml)), this product is licensed under [the 4.0 version of the Creative Commons Attribution-ShareAlike license](https://creativecommons.org/licenses/by-sa/4.0/), or any of its more recent versions of your preference.

This work complies to [the REUSE Specification](https://reuse.software/spec/), refer to the [REUSE - Make licensing easy for everyone](https://reuse.software/) website for info regarding the licensing of this product.
