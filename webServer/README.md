# Web Server (Apache/httpd)

A single playbook that installs and configures **Apache (httpd)** on RHEL-based
hosts in the `dev` inventory group, opens port 80 via `firewalld`, drops in a
custom vhost config, and deploys a static `index.html`.

This is a standalone playbook (not a role) — everything it needs lives in this
folder.

## Files

| File | Purpose |
|---|---|
| `webserver.yml` | The playbook: install package → start service → open firewall port → copy vhost conf → validate `httpd -t` → copy `index.html` → restart |
| `web_vars.yml` | Variables consumed by the playbook: `pkg`, `service`, `port` |
| `myweb.conf` | Apache VirtualHost config, copied to `/etc/httpd/conf.d/` |
| `index.html` | Static page copied to `/var/www/html` |
| `env.j2` / `hosts-style.j2` | Jinja2 template examples (host-style / env file templating practice) |

## Variables (`web_vars.yml`)

```yaml
pkg: httpd
service: httpd.service
port: 80/tcp
```

## What the playbook does

1. Installs `{{ pkg }}` via `yum`
2. Starts and enables `{{ service }}`
3. Opens `{{ port }}` permanently in `firewalld`
4. Copies `myweb.conf` to `/etc/httpd/conf.d/`
5. Runs `httpd -t` to validate the config and prints the result
6. Copies `index.html` to `/var/www/html`
7. Restarts `{{ service }}` to apply the new config

## Requirements

- RHEL/CentOS/Rocky/Alma target (uses the `yum` module and `firewalld`)
- Target host must be in the `dev` group of the inventory in use (default:
  `../myhosts`)
- `become: true` (already set globally in `../ansible.cfg`)

## Usage

From the repo root:

```bash
ansible-playbook webServer/webserver.yml
```

To target a different group, edit `hosts: dev` in `webserver.yml`, or add
`-l <hostname>` on the command line.
