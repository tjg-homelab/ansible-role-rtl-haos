# Ansible Role: rtl-haos

[![CI](https://github.com/tjg-homelab/ansible-role-rtl-haos/actions/workflows/ci.yml/badge.svg)](https://github.com/tjg-homelab/ansible-role-rtl-haos/actions/workflows/ci.yml)

Installs and configures [rtl-haos](https://github.com/jaronmcd/rtl-haos) — a
bridge that reads 433 MHz sensor transmissions via an RTL-SDR dongle and
publishes them to Home Assistant over MQTT (auto-discovery).

The role clones rtl-haos at a pinned commit, builds a Python virtualenv,
installs it, renders the `.env` configuration, and manages the `rtl-bridge`
systemd service.

## Requirements

- Debian 12/13 (Raspberry Pi OS included)
- An RTL-SDR dongle
- A reachable MQTT broker (e.g. the Mosquitto add-on in Home Assistant)

## Role Variables

**Required** (set for your environment):

| Variable | Description |
|---|---|
| `rtl_haos_mqtt_host` | MQTT broker hostname/IP |
| `rtl_haos_mqtt_user` | MQTT username |
| `rtl_haos_mqtt_pass` | MQTT password (store in Ansible Vault) |

**Common:**

| Variable | Default | Description |
|---|---|---|
| `rtl_haos_mqtt_port` | `1883` (rtl-haos default) | Broker port (uncomment to override) |
| `rtl_haos_install_path` | `/opt/rtl-haos` | Install directory |
| `rtl_haos_version` | pinned commit SHA | Upstream commit to deploy (no upstream tags) |
| `rtl_haos_rtl_config` | single weather-radio entry | JSON array of radio configs (name, id, freq, rate) |
| `rtl_haos_device_whitelist` | common sensor families | JSON array; only matching devices are published |
| `rtl_haos_device_blacklist` | _(unset)_ | JSON array of device patterns to suppress |
| `rtl_haos_source_mode` | `git` | `local` skips clone/venv/pip (used by molecule) |

Additional optional tuning variables (expiry, throttling, logging) are listed
and documented in `defaults/main.yml`.

## Example Playbook

```yaml
- hosts: rtl_receivers
  roles:
    - role: rtl-haos
      vars:
        rtl_haos_mqtt_host: 192.0.2.10
        rtl_haos_mqtt_user: rtl_433
        rtl_haos_mqtt_pass: "{{ vault_rtl_haos_mqtt_pass }}"
        rtl_haos_rtl_config: '[{"name": "Weather Radio", "id": "102", "freq": "433.92M"}]'
```

Installing via `requirements.yml`:

```yaml
roles:
  - name: rtl-haos
    src: https://github.com/tjg-homelab/ansible-role-rtl-haos.git
    version: v1.0.0
```

## Testing

Molecule (Docker driver) runs in `local` source mode (no git clone / RTL-SDR
hardware) against Debian 12 and Debian 13, verifying the install directory,
rendered `.env`, systemd unit, and service enablement.

```bash
pip install ansible-core molecule molecule-plugins[docker] docker
ansible-galaxy collection install community.docker ansible.posix
molecule test
```

## License

MIT

## Author

Rodney Nissen ([The Jira Guy](https://thejiraguy.com))
