# lxd

[![Build Status](https://drone.osshelp.ru/api/badges/ansible/lxd/status.svg)](https://drone.osshelp.ru/ansible/lxd)

Installs LXD configures it.

## Usage (example with non-auto addresses)

```yaml
    - role: lxd
      ipv4: { address: '10.103.23.1/24', nat: true }
      ipv6: { address: 'fd42:b34:7700:e81c::1/64', nat: true }
```

## Available parameters

### Main

Main params to use in playbooks.

| Param | Default | Description |
| -------- | -------- | -------- |
| `lxd_setup` | `full` | Setup mode. See [OSSHelp KB article](https://oss.help/kb4895) |
| `br_name` | `lxdbr0` | Name of bridge to be created. |
| `ipv4.address` | `auto` | IPv4 subnet to create. |
| `ipv4.nat` | `true` | Whether to enable NAT for IPv4. |
| `ipv6.address` | `auto` | IPv6 subnet to create. |
| `ipv6.nat` | `true` | Whether to enable NAT for IPv6. |
| `nesting_by_default` | `false` | Whether to set [nesting mode](https://ubuntu.com/blog/nested-containers-in-lxd) for containers by default. |
| `privileged_by_default` | `false` | Whether to create containers [privileged](https://linuxcontainers.org/ru/lxc/security/) by default. |
| `ubuntu_minimal_remote` | `true` | Whether to add remote with minimal Ubuntu images. |
| `osshelp_remote` | `true` | Whether to add the remote with [our images](https://gitea.osshelp.ru/org/lxc/dashboard). |
| `lxd_create_wakeup_service` | `false` | Whether to create a service for forced daemon wake up via oneshot systemd-unit (daemon sleeps until the first call) |
| `storage_pool_dirs` | - | Dirs to be created for custom storage pools. |
| `storage_pools` | `{ name: default, driver: dir }` | Array, describing needed storage pools and their params. |
| `place_cfg_template` | `false` |  Whether to create initial-setup script. Recommended to use only if you need LXD in LXC-containers. |

### Misc

Usually there is no need in changing theese.

| Param | Default | Description |
| -------- | -------- | -------- |
| `lxd_packages` | `[ 'lxd=3.*', 'lxd-client=3.*', 'apparmor' ]` | Array of packages to install. |
| `initial_config` | `/usr/local/etc/lxd-init-config.yml` | Absolute path to config for further lxd initialization. |
| `lxd_init_done_file` | `/var/lib/lxd/init.done` | Absolute path for file, indicating that lxd initialization was already done. |
| `templates_dir` | `/usr/local/osshelp` | Absolute path to directory for templates storage (where default-setup usually placed). |
| `lxd_set_subuid`, `lxd_set_subgid` | `[]` | Dictionaries for setting custom subuid/subgid ranges. Check examples in FAQ section below before using it in your builds! |

## FAQ

### IPv4/IPv6 subnet keeps changing every deploy

Make sure that:

1. You are using the up to date version of the role
1. Role is able to create the `lxd_init_done_file` (see above)
1. `ipv4.address` and `ipv6.address` variables are not set to `auto` in your playbook

### Errors on initialization

- If you get `Failed to create network` errors - try using a newer version of [bind9](https://gitea.osshelp.ru/ansible/bind9) role (at least v4.x.x).
- If lxd.service does not start after reboot without any errors, use the parameter `lxd_create_wakeup_service`

### I have troubles with building an lxc-image with this role

You may be lacking of [initial-setup](templates/initial-setup.j2) functionality. Make sure that you've set the `place_cfg_template` param to `true` (see above). If it will not help, describe your problem with an issue, we'll try to investigate and fix it.

### I need to set custom subuid/subgid ranges

**Warning** Changing ranges on systems with already deployed containers will result in invalid permissions on mounted persistent data. You can remap permissions with [fuidshift](http://manpages.ubuntu.com/manpages/cosmic/man1/fuidshift.1.html) tool.

Example of setting custom subuid/subgid ranges with `lxd_set_subuid` and `lxd_set_subgid` dictionaries:

``` yaml
    - role: lxd
      lxd_set_subuid:
        lxd: '165536:65536'
        root: '165536:65536'
        noname: '100000:65536'
      lxd_set_subgid:
        lxd: '165536:65536'
        root: '165536:65536'
        noname: '100000:65536'
```

Will result in:

```plaintext
lxd:165536:65536
root:165536:65536
noname:100000:65536
```

## Useful links

- [Official documentation](https://linuxcontainers.org/lxd/docs/master/)
- [Our article](https://oss.help/kb903)

## TODO

- better way to manage LXD remotes (not via shell?)
- using the role for setting up LXD in lxc-containers could be buggy

## License

GPL3

## Author

OSSHelp Team, see <https://oss.help>
