# itential.valkey — Ansible Collection

## Overview

`itential.valkey` is a standalone Ansible collection (namespace `itential`, name `valkey`)
that installs and configures Valkey as an alternative to `itential.deployer`'s `redis` role.

**This collection is transitional.** It exists separately from `itential.deployer` only until
`roles/redis` is retired there. At that point, `roles/valkey` and its playbooks are intended to
be merged directly into `itential.deployer`, and this repository archived. See
`roles/valkey/CLAUDE.md`'s "Collection Context" section for exactly what reverts at merge time.

## Collection Metadata

| Field | Value |
|-------|-------|
| Namespace | `itential` |
| Name | `valkey` |
| Version | `1.0.0` |
| Depends on | `itential.deployer` `>=4.0.0` |

## Playbook Inventory

| Playbook | FQCN | Description |
|----------|------|-------------|
| `valkey.yml` | `itential.valkey.valkey` | Install Valkey on `valkey_master`/`valkey_replica`; Sentinel on `valkey_sentinel` hosts |
| `verify_valkey.yml` | `itential.valkey.verify_valkey` | Pre-install verification for Valkey hosts |
| `certify_valkey.yml` | `itential.valkey.certify_valkey` | Generate Valkey/Sentinel installation certification reports |
| `download_packages_valkey.yml` | `itential.valkey.download_packages_valkey` | Download Valkey packages for offline install |

## Roles Summary

| Role | Purpose |
|------|---------|
| `valkey` | Installs and configures Valkey (via the native OS package repositories only — EL9 AppStream or Amazon Linux 2023's core repo) and Valkey Sentinel: auth, TLS, replication. No source install, no Remi, no EL8 support, no relocatable install paths, no role-level SELinux step (handled entirely by the base OS policy). See `roles/valkey/CLAUDE.md` for full detail. |

## Cross-Collection Dependency

`roles/valkey`'s tasks call two shared utility roles that live in `itential.deployer`, by FQCN:

- `itential.deployer.common` (used in `verify-valkey.yml`, `verify-sentinel.yml`)
- `itential.deployer.offline` (used in `download-packages.yml`, `install-from-repo.yml`)

Both collections must be installed together for this to work — see `README.md`'s
Installation section. This is declared in `galaxy.yml`'s `dependencies`.

## Docs Index

| File | Contents |
|------|----------|
| `docs/valkey_guide.md` | Valkey role variables, Sentinel, TLS, EL9/Amazon Linux 2023 native package install |

## CI

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ansible-lint.yml` | push/PR to `main`, `dev` | Lint validation |
| `role-readme-check.yml` | push/PR to `dev` | Fails the build if any `roles/*/` directory is missing a README (Ansible Galaxy requirement) |
| `updateChangelog.yml` | GitHub release or manual | Updates the changelog |
| `publish_ansible_collection.yml` | GitHub release or manual | Publish to Galaxy |
