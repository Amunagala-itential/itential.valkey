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

This does not automatically install `itential.deployer` — install both:

```bash
ansible-galaxy collection install itential.deployer itential.valkey
```

## Quick Start

Add a `valkey_master` group to your inventory alongside your existing `mongodb`/`platform`/
`gateway` groups (see `itential.deployer`'s own README for those):

```yaml
all:
  vars:
    platform_release: 6
  children:
    valkey_master:
      hosts:
        <host1>:
          ansible_host: <addr1>
```

Then run:

```bash
ansible-playbook itential.valkey.valkey -i <inventory>
```

For Sentinel HA topologies, TLS configuration, offline installs, and the full variable
reference, see [`docs/valkey_guide.md`](docs/valkey_guide.md) and the examples under
[`example_inventories/valkey/`](example_inventories/valkey/).

## Playbooks

| Playbook | FQCN | Description |
|----------|------|-------------|
| `valkey.yml` | `itential.valkey.valkey` | Install Valkey on `valkey_master`/`valkey_replica`; Sentinel on `valkey_sentinel` hosts |
| `verify_valkey.yml` | `itential.valkey.verify_valkey` | Pre-install verification for Valkey hosts |
| `certify_valkey.yml` | `itential.valkey.certify_valkey` | Generate Valkey/Sentinel installation certification reports |
| `download_packages_valkey.yml` | `itential.valkey.download_packages_valkey` | Download Valkey packages for offline install |

## Component Guide

[Valkey Guide](docs/valkey_guide.md)

## License

See [LICENSE](LICENSE).
