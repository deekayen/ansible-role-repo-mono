# deekayen.repo_mono

[![CI](https://github.com/deekayen/ansible-role-repo-mono/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-repo-mono/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.repo__mono-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/repo_mono/) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Linux platform](https://img.shields.io/badge/platform-linux-lightgrey)

> **Deprecated.** The Mono Project only published repositories for CentOS 7
> and 8, and Mono moved to WineHQ in 2024 with no EL 9 or later packages.
> The role is kept for existing EL 7/8 hosts. CI lints and syntax-checks it
> but no longer converges it on a running system.

An Ansible role that adds the [Mono Project](https://www.mono-project.com/download/stable/) stable yum repository to an EL 7 or EL 8 host, so later tasks can install Mono packages with the system package manager. It installs no Mono packages itself.

The role imports the Mono signing key into the RPM database from `mono_rpm_key_url`, then writes `/etc/yum.repos.d/mono-centos<major>-stable.repo` with `ansible.builtin.yum_repository`, pointing at `mono_base_url` with `gpgcheck` on and `mono_repo_gpg_key_url` as the repository key. `<major>` is `ansible_facts.distribution_major_version`.

The Galaxy name is `deekayen.repo_mono`, with an underscore, not `deekayen.repo-mono`.

## Requirements

- ansible-core 2.15 or newer on the controller. For EL 7 targets, use ansible-core 2.16 or older; as of October 2026, the [Ansible support matrix](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html) lists Python 2.7 and 3.6 as target versions for 2.16 but not 2.17, and EL 7 ships those two Pythons.
- An EL 7 or EL 8 target. `tasks/assert.yml` fails the play on any other OS family or major version.
- Fact gathering left on. The URLs, file name, and assertion use `ansible_facts`.
- Outbound HTTPS from the target to `mono_rpm_key_url`, and from its package manager to `mono_base_url` and `mono_repo_gpg_key_url`.
- Privilege escalation. The key and repository tasks set `become: true` themselves, but the `/etc/yum.repos.d` task does not, so run the play with `become: true`.

## Supported platforms

| Platform | Versions |
| --- | --- |
| EL | 7, 8 |

CI runs `ansible-lint` and `ansible-playbook --syntax-check` only.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.repo_mono
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.repo_mono
    src: https://github.com/deekayen/ansible-role-repo-mono.git
    scm: git
    version: main
```

```bash
ansible-galaxy role install -r requirements.yml
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `mono_base_url` | `https://download.mono-project.com/repo/centos{{ ansible_facts.distribution_major_version }}-stable/` | Repository `baseurl`. Must be an `http://` or `https://` URL. Override it to use an internal mirror. |
| `mono_repo_gpg_key_url` | `https://download.mono-project.com/repo/xamarin.gpg` | Repository `gpgkey`, which the package manager uses to verify packages. Must be an `http://` or `https://` URL. |
| `mono_repofile_path` | `/etc/yum.repos.d/mono-centos{{ ansible_facts.distribution_major_version }}-stable.repo` | File the role checks for before adding the repository. See [Known issues](#known-issues). |
| `mono_rpm_key_url` | `https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF` | Key imported with `rpm_key`. Must be an `http://` or `https://` URL. |

## Behavior

- The repository task runs only if `mono_repofile_path` does not exist. Once the file is there, later changes to `mono_base_url` or `mono_repo_gpg_key_url` are not applied. Delete the `.repo` file and run the role again to rewrite it.

## Dependencies

None.

## Example playbook

```yaml
---
- name: Add the Mono repository to legacy .NET hosts.
  hosts: mono_app_servers
  become: true

  vars:
    mono_base_url: "https://mirror.example.internal/mono/centos{{ ansible_facts.distribution_major_version }}-stable/"

  roles:
    - deekayen.repo_mono
```

`mirror.example.internal` is a placeholder for an internal mirror of the Mono repository.

## Known issues

- `mono_repofile_path` is only read by the existence check in `tasks/main.yml` line 12. `yum_repository` writes the file named after its `name:` value, `mono-centos<major>-stable.repo`, regardless of this variable. Setting `mono_repofile_path` to another path makes the check look at a file the role never writes, so the repository task runs on every play.
- "Setup /etc/yum.repos.d path." (`tasks/main.yml` line 27) sets `owner: root` and `group: root` without `become`, unlike the other tasks that need root. A play without `become: true` fails there.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It runs `ansible-lint --profile production` and syntax-checks `tests/test.yml`. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.repo_mono
ANSIBLE_ROLES_PATH=.ansible/roles:~/.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Existence check, RPM key import, and repository file. |
| `tasks/assert.yml` | EL 7/8 check and URL validation, tagged `always`. |
| `defaults/main.yml` | Every user-facing variable. |
| `meta/argument_specs.yml` | Argument types and descriptions. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.repo_mono`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Authors

[David Norman](https://github.com/deekayen) adapted this role from [geerlingguy/ansible-role-repo-epel](https://github.com/geerlingguy/ansible-role-repo-epel) by [Jeff Geerling](https://github.com/geerlingguy). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
