# Ansible Role: Mono Project Repository

[![CI](https://github.com/deekayen/ansible-role-repo-mono/workflows/CI/badge.svg)](https://github.com/deekayen/ansible-role-repo-mono/actions?query=workflow%3ACI)  [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive)  ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Linux platform](https://img.shields.io/badge/platform-linux-lightgrey)

> **Deprecated.** The Mono Project only published repositories for CentOS 7
> and 8, and Mono moved to WineHQ in 2024 with no EL 9 or later packages.
> The role is kept for existing EL 7/8 hosts. CI lints and syntax-checks it
> but no longer converges it on a running system.

Installs the [Mono repository](https://www.mono-project.com/download/stable/#download-lin-centos) for RHEL/CentOS.

## Requirements

This role only is needed/runs on RHEL and its derivatives.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    mono_base_url: "https://download.mono-project.com/repo/centos{{ ansible_facts.distribution_major_version }}-stable/"
    mono_repo_gpg_key_url: "https://download.mono-project.com/repo/xamarin.gpg"
    mono_repofile_path: "/etc/yum.repos.d/mono-centos{{ ansible_facts.distribution_major_version }}-stable.repo"
    mono_rpm_key_url: "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"

The Mono repo URL and GPG key URL. Generally, these should not be changed, but if this role is out of date, or if you need a very specific version, these can both be overridden.

## Dependencies

None.

## Example Playbook

    - hosts: servers
      roles:
        - deekayen.repo-mono

## License

BSD
