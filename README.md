# ansible-role-packages

Manage system packages and repositories for Debian/Ubuntu.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `packages_install_common` | see defaults | Common packages for all distros |
| `packages_install_extra` | `[]` | Additional packages (user-defined) |
| `packages_remove` | `[]` | Packages to remove |
| `packages_update_cache` | `true` | Update apt cache (with retries: 3, delay: 10s) |
| `packages_cache_valid_time` | `3600` | Cache validity in seconds |
| `packages_repositories` | `[]` | Additional repositories |
| `packages_upgrade_mode` | `none` | Upgrade mode: `none` / `safe` / `full` |
| `packages_full_upgrade_async_timeout` | `1800` | Max wait for safe/full upgrade (sec) |
| `packages_full_upgrade_async_poll` | `30` | Poll interval — ansible prints status (sec) |
| `packages_yq_install` | `true` | Install yq YAML processor |
| `packages_yq_version` | `v4.44.3` | yq version (used if apt unavailable) |

## Upgrade modes

| Mode | apt command | What it does | Time | Reboot? |
|------|-------------|--------------|------|---------|
| `none` | — | Nothing, only installs new packages from `packages_install_*` | 0 | no |
| `safe` | `apt-get upgrade` | Security/bugfix updates **without** kernel/firmware/new deps | 1-3 min | usually no |
| `full` | `apt-get dist-upgrade` | Everything including kernel + `linux-firmware` (~130 MB) + autoremove | 10-30 min | **yes** |

Use `safe` for regular maintenance, `full` for planned reboots once per 3-6 months
(snapshot the VM first).

## Notes

- **`dpkg_options: force-confold,force-confdef`** is applied to all `apt`
  install/remove/upgrade tasks. This prevents dpkg from hanging on interactive
  prompts when a maintainer changed a config file you have locally modified
  (keeps your version, takes maintainer default for everything else).
- **`apt update`** is wrapped with `retries: 3, delay: 10` to survive transient
  503 from mirrors.
- **Safe and full upgrade are async** with default `1800s` total / `30s` poll,
  so ansible prints a heartbeat instead of going silent while `linux-firmware`
  (130+ MB) and kernel packages download.

## Distro-specific packages

Automatically loaded from `vars/{{ ansible_facts['distribution'] }}.yml`:

- **Debian**: `sudo`
- **Ubuntu**: none

## yq Installation

The `yq` YAML processor is installed automatically:

- **Ubuntu 23.04+**: installed from apt repository
- **Ubuntu 22.04, Debian 11/12**: downloaded from [GitHub releases](https://github.com/mikefarah/yq)

To disable yq installation:

```yaml
packages_yq_install: false
```

## Examples

### Add extra packages
```yaml
packages_install_extra:
  - docker-ce
  - docker-compose
```

### Remove packages
```yaml
packages_remove:
  - apache2
  - sendmail
```

### Add repository
```yaml
packages_repositories:
  - repo: "deb https://download.docker.com/linux/debian {{ ansible_distribution_release }} stable"
    key_url: "https://download.docker.com/linux/debian/gpg"
    filename: docker
```

## License

MIT
