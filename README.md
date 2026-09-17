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

## Getting Started

This walks through deploying Valkey alongside the rest of the Itential stack (MongoDB,
Platform, Gateway), using `itential.deployer`'s own bootstrap flow. If you've already got a
working directory set up for `itential.deployer` (see its README's "Running the Deployer"
section for the full explanation of each step below), skip to
[3. Add Valkey to your inventory](#3-add-valkey-to-your-inventory).

### 1. Install both collections

```bash
ansible-galaxy collection install itential.deployer itential.valkey
```

### 2. Set up your working directory

Same layout `itential.deployer` uses — this collection doesn't need its own separate working
directory.

```bash
mkdir -p <WORKING-DIR>/inventories/dev
cd <WORKING-DIR>
```

`itential.valkey` has no artifacts to stage (Valkey installs from the OS's own package
repositories, not an uploaded binary), so unlike Platform/Gateway there's no `files` directory
or symlink step needed for it specifically. If you're also installing Platform/Gateway in the
same run, follow `itential.deployer`'s "Determine Installation Artifacts Method" steps for
those.

### 3. Add Valkey to your inventory

Add a `valkey_master` group (and `valkey_replica`/`valkey_sentinel` for HA — see
[`docs/valkey_guide.md`](docs/valkey_guide.md)) to the **same** inventory file you're using
for the rest of the stack. This is a full example combining Valkey with MongoDB and Platform,
adapted from `itential.deployer`'s own example inventory:

```yaml
# <WORKING-DIR>/inventories/dev/hosts
all:
  vars:
    platform_release: 6
    env: dev

  children:
    valkey_master:
      hosts:
        host01.example.com:
      vars:
        valkey_tls_enabled: false

    mongodb:
      hosts:
        host01.example.com:

    platform:
      hosts:
        host01.example.com:
      vars:
        platform_encryption_key: <key>
        platform_packages:
          - https://registry.aws.itential.com/repository/PLATFORM/Platform%206.0.0/itential-platform-<version>.noarch.rpm
        repository_username: <username>
        repository_password: !vault |
          $ANSIBLE_VAULT;1.1;AES123
          ...

        platform_mongo_url: mongodb://host01.example.com:27017/itential

        # Point Platform at Valkey the same way it would point at Redis -- the client
        # only speaks RESP and doesn't care which server product is on the other end.
        platform_redis_host: host01.example.com
```

Note that `host01.example.com` must be an EL9 (RHEL/Rocky/AlmaLinux) or Amazon Linux 2023 host
— `valkey_master` fails fast on anything else. If you need Redis on an EL8 host instead, use
`redis_master` and `itential.deployer`'s own `redis` role there.

### 4. Verify, then install, then certify

```bash
# Confirm the environment is ready (repo connectivity, host specs, etc.)
ansible-playbook -i inventories/dev itential.deployer.verify
ansible-playbook -i inventories/dev itential.valkey.verify_valkey

# Install Valkey
ansible-playbook -i inventories/dev itential.valkey.valkey -v

# Install the rest of the stack (MongoDB, Platform, Gateway) -- unchanged from
# itential.deployer's own flow, run separately since Valkey isn't wired into
# itential.deployer.site
ansible-playbook -i inventories/dev itential.deployer.platform_site -v

# Confirm the installation and generate a certification report
ansible-playbook -i inventories/dev itential.valkey.certify_valkey
ansible-playbook -i inventories/dev itential.deployer.certify
```

**&#9432; Known gap:** `itential.deployer`'s `os.yml` playbook (which installs baseline OS,
security, and operational packages -- including whatever provides `firewalld`) does not yet
include `valkey_master`/`valkey_replica`/`valkey_sentinel` in its target hosts. Until that's
fixed upstream, a genuinely fresh host may need baseline OS packages installed by some other
means first, or `roles/valkey`'s automatic firewalld port-opening simply won't have anything
to act on (it degrades gracefully -- it skips opening the port rather than failing -- but the
port then needs to be opened some other way).

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
