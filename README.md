# Ansible Collection - itential.valkey

## Overview

`itential.valkey` installs and configures [Valkey](https://valkey.io) — a BSD-licensed,
community-governed fork of Redis — as an alternative cache/broker backend for the Itential
automation platform stack.

Valkey is protocol- and config-compatible with Redis: same RESP protocol, same config
grammar, same Sentinel mechanics. The Itential Platform connects to either backend the same
way. See [`docs/valkey_guide.md`](docs/valkey_guide.md) for the full role reference.

**This collection is transitional.** It exists separately from
[`itential.deployer`](https://github.com/itential/deployer) (which owns the `redis` role and
the rest of the platform stack: MongoDB, Itential Platform, Itential Gateway) only until
`roles/redis` is retired there. At that point this collection's `roles/valkey` and its
playbooks are intended to be merged directly into `itential.deployer`, and this repository
will be archived.

## Why Valkey, and why EL9/Amazon Linux 2023 only

Redis relicensed away from a fully open-source model in 2024; Valkey is the community fork
that stayed BSD-licensed. This role deliberately:

- Installs **only** via the native OS package repositories (the AppStream module on EL9,
  Amazon Linux 2023's own core repo) — never Remi, never a source compile.
- Does **not** support EL8 at all (no AppStream stream exists there, and this role does not
  compile from source to work around that).
- Does **not** support customizing install paths — the Valkey RPM is not relocatable.

Customers who need EL8 support or non-standard install paths should use `itential.deployer`'s
`redis` role instead. See [`docs/valkey_guide.md`](docs/valkey_guide.md#supported-platforms)
for the full rationale.

## Dependencies

| Collection | Version Constraint | Why |
|------------|--------------------|-----|
| `itential.deployer` | `>=4.0.0` | Provides the shared `common` and `offline` utility roles this role calls by FQCN (`itential.deployer.common`, `itential.deployer.offline`). You need `itential.deployer` installed regardless, since it owns MongoDB, Platform, and Gateway. |

## Installation

```bash
ansible-galaxy collection install itential.valkey
```

`ansible-galaxy` automatically resolves and installs `itential.deployer` too, since it's
declared as a dependency in this collection's `galaxy.yml` — you don't need to install it
separately.

## Getting Started

Add a `valkey_master` group (and `valkey_replica`/`valkey_sentinel` for HA) to your inventory:

```yaml
all:
  vars:
    platform_release: 6
  children:
    valkey_master:
      hosts:
        host01.example.com:
      vars:
        valkey_tls_enabled: false
```

`host01.example.com` must be RHEL/Rocky/AlmaLinux 9 or Amazon Linux 2023 — `valkey_master`
fails fast on anything else (including EL8; use `itential.deployer`'s `redis` role there
instead).

Then:

```bash
# Confirm the environment is ready
ansible-playbook -i <inventory> itential.valkey.verify

# Install Valkey
ansible-playbook -i <inventory> itential.valkey.valkey -v

# Confirm the installation and generate a certification report
ansible-playbook -i <inventory> itential.valkey.certify
```

**&#9432; Known gap:** `itential.deployer`'s `os.yml` playbook (baseline OS/security/firewalld
packages) does not yet target `valkey_master`/`valkey_replica`/`valkey_sentinel`. Until that's
fixed upstream, a genuinely fresh host may need those baseline packages installed some other
way — `roles/valkey`'s firewalld port-opening degrades gracefully (skips rather than fails) if
firewalld isn't present.

This collection only covers Valkey. For the rest of the stack (MongoDB, Itential Platform,
Itential Gateway) and how to combine them into one inventory alongside Valkey, see
[`itential.deployer`'s own README](https://github.com/itential/deployer#running-the-deployer).

For Sentinel HA topologies, TLS configuration, offline installs, and the full variable
reference, see [`docs/valkey_guide.md`](docs/valkey_guide.md) and the examples under
[`example_inventories/valkey/`](example_inventories/valkey/).

## Playbooks

| Playbook | FQCN | Description |
|----------|------|-------------|
| `valkey.yml` | `itential.valkey.valkey` | Install Valkey on `valkey_master`/`valkey_replica`; Sentinel on `valkey_sentinel` hosts |
| `verify.yml` | `itential.valkey.verify` | Pre-install verification for Valkey hosts |
| `certify.yml` | `itential.valkey.certify` | Generate Valkey/Sentinel installation certification reports |
| `download_packages.yml` | `itential.valkey.download_packages` | Download Valkey packages for offline install |

## Component Guide

[Valkey Guide](docs/valkey_guide.md)

## License

See [LICENSE](LICENSE).
