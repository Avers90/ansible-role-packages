# ansible-role-packages

Manage system packages and repositories for Debian/Ubuntu.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `packages_install_common` | see defaults | Common packages for all distros |
| `packages_install_extra` | `[]` | Additional packages (user-defined) |
| `packages_remove` | `[]` | Packages to remove |
| `packages_update_cache` | `true` | Update apt cache |
| `packages_cache_valid_time` | `3600` | Cache validity in seconds |
| `packages_repositories` | `[]` | Additional repositories |
| `packages_yq_install` | `true` | Install yq YAML processor |
| `packages_yq_version` | `v4.44.3` | yq version (used if apt unavailable) |

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
