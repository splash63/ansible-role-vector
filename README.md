# ansible-role-vector

Installs [Vector](https://vector.dev) from the official release tarball and runs it as a systemd
service, with its configuration deployed from a Jinja2 template.

The role is idempotent: it checks whether the binary is already unpacked before downloading/unpacking
again, and the service is only restarted when the rendered config or unit file actually changes.

## Requirements

- A systemd-based Linux distribution (Debian/Ubuntu, RHEL/CentOS/Rocky, ...)
- Outbound HTTPS access to `packages.timber.io` from the target host
- `become: true` / root privileges

## Role Variables

### `defaults/main.yml` (safe to override from your playbook/inventory)

| Variable              | Default                          | Description                                                    |
|------------------------|-----------------------------------|------------------------------------------------------------------|
| `vector_version`       | `0.46.0`                          | Version of Vector to install                                     |
| `vector_arch`          | `x86_64-unknown-linux-gnu`        | Platform triplet of the release tarball                          |
| `vector_install_dir`   | `/opt/vector`                     | Where the tarball is unpacked                                    |
| `vector_bin_path`      | `/usr/local/bin/vector`           | Symlink to the vector binary, placed on `PATH`                   |
| `vector_config_dir`    | `/etc/vector`                     | Directory holding `vector.yaml`                                  |
| `vector_data_dir`      | `/var/lib/vector`                 | Vector's `data_dir` (on-disk buffers/state)                      |
| `vector_source`        | `journald`                        | `sources.*.type` in the rendered config                          |
| `vector_sink_type`     | `console`                         | `sinks.*.type` in the rendered config                            |

### `vars/main.yml` (computed / internal — override only if you know what you're doing)

| Variable                 | Description                                             |
|----------------------------|-----------------------------------------------------------|
| `vector_config_file`       | Full path to the rendered config (`{{ vector_config_dir }}/vector.yaml`) |
| `vector_download_url`      | Release tarball URL, built from `vector_version`/`vector_arch` |
| `vector_archive_path`      | Where the tarball is downloaded to on the target host        |
| `vector_service_name`      | systemd unit/service name (`vector`)                          |
| `vector_systemd_unit_path` | Path of the deployed systemd unit file                        |

## Templates

- `templates/vector.yaml.j2` — renders `sources.<vector_source>_input` -> `sinks.<vector_sink_type>_output`
- `templates/vector.service.j2` — systemd unit running `vector --config {{ vector_config_file }}`

## Handlers

- `Reload systemd daemon` — runs `systemctl daemon-reload`, notified when the unit file changes
- `Restart vector` — restarts the service, notified when the config or unit file changes (unit-file
  changes notify both handlers, in that order, so systemd sees the new unit before the restart)

## Example Playbook

```yaml
- name: Install and configure vector
  hosts: vector
  roles:
    - role: vector
      vector_version: "0.46.0"
      vector_source: journald
      vector_sink_type: console
```

## License

MIT

## Author Information

Evgenii - hands-on web/server administration, Beget hosting, MODX.
