Society Management (MERN) Role
================================

Deploys a full **MERN** (MongoDB, Express, React, Node.js) application —
Society Management System — on a single RHEL 10 host, fronted by **Nginx**
as a reverse proxy and secured with a **Let's Encrypt** TLS certificate. This
is the most complete role in the repo, split into six ordered, tagged parts.

Entry point: `site.yml` at the repo root runs this role against the
`[appserver]` group defined in `inventory/inventory.ini`.

Requirements
------------

- **RHEL 10** target (uses `dnf`, EPEL 10, `firewalld`, SELinux booleans/contexts)
- `become: true`
- `ansible.posix` collection (`sefcontext`, `seboolean`)
- Outbound internet access on the target (NodeSource script, MongoDB repo,
  npm registry, GitHub for the app repo clone, EPEL, Let's Encrypt)
- A DNS record for `domain_name` / `domain_www` pointing at the target, since
  Certbot's `--nginx` mode validates the domain over HTTP before issuing the
  certificate
- `defaults/main.yml` is **Ansible Vault encrypted** — you need the vault
  password to run this role

Role Variables
--------------

All variables live in `defaults/main.yml` (Vault-encrypted). Based on what
the tasks and templates reference, expect at least:

| Variable | Used in | Purpose |
|---|---|---|
| `node_repo_setup_url` | Part A | NodeSource setup script URL (Node.js 20.x) |
| `app_dir` | Parts B–D | Path the app repo is cloned into (e.g. `/opt/society-management`) |
| `app_repo` | Part B | Git URL of the MERN application |
| `app_repo_branch` | Part B | Branch to deploy (defaults to `master` if unset) |
| `mongodb_uri` | Part B (`backend.env.j2`) | MongoDB connection string |
| `jwt_expires_in` | Part B (`backend.env.j2`) | JWT token expiry |
| `smtp_host`, `smtp_port`, `smtp_user`, `smtp_password` | Part B/F | Outgoing mail config; `smtp_user` is also passed to Certbot as the registration email |
| `client_url` | Parts B/C (`.env` files) | Public URL of the frontend, used for CORS/Socket.io origin |
| `domain_name`, `domain_www` | Part D/F (`society.conf.j2`, Certbot) | Domain(s) the Nginx server block and SSL cert cover |

`jwt_secret` is **not** a variable you set — it's generated at runtime in
Part B and stored as a `set_fact`.

View/edit the encrypted vars:

```bash
ansible-vault view roles/society_management/defaults/main.yml
ansible-vault edit roles/society_management/defaults/main.yml
```

Dependencies
------------

None declared in `meta/main.yml`. Everything (Node.js, MongoDB, the app repo,
Nginx, Certbot) is installed by this single role.

What it does (by part)
-----------------------

Each part is `import_tasks`'d from `tasks/main.yml` and individually tagged,
so you can run just one with `--tags`:

| Part | Tag | What it does |
|---|---|---|
| **A — Base prep** | `part_a` | `dnf update`, base tools (git, vim, curl, wget, firewalld), installs Node.js 20 via NodeSource, adds the MongoDB 8.0 repo and installs/starts `mongod` |
| **B — Backend** | `part_b` | Clones `app_repo`, generates a JWT secret, templates `server/.env`, patches two known hardcoded-`localhost` bugs in `app.js` and `cloudinary.js`, `npm install`, starts the backend with PM2 (`pm2 start app.js`, `pm2 save`) |
| **C — Frontend** | `part_c` | Templates `client/.env.production`, patches hardcoded Socket.io URLs and a case-sensitive `AuthSlice`/`authSlice` import bug, `npm install`, `npm run build`, verifies `dist/index.html` exists |
| **D — Nginx** | `part_d` | Installs and starts Nginx, deploys the `society.conf` server block (serves `client/dist`, proxies `/api/` and `/socket.io/` to `127.0.0.1:3000`), fixes ownership/permissions and SELinux context (`httpd_sys_content_t`) on the `dist` folder |
| **E — Firewall/SELinux** | `part_e` | Opens `http`/`https` in `firewalld`, sets the `httpd_can_network_connect` SELinux boolean so Nginx can proxy to the backend (fixes a common 502) |
| **F — SSL** | `part_f` | Installs EPEL 10 + Certbot + `python3-certbot-nginx`, issues a cert for `domain_name`/`domain_www` via `certbot --nginx` (skips if already present), confirms the `certbot-renew.timer` is active |

Handlers
--------

- `reload nginx` — reloads Nginx after the server block changes
- `restart httpd` — defined for completeness/reuse, not currently notified by
  any task in this role (httpd isn't used here; Nginx is the web server)

Example Playbook
-----------------

```yaml
- name: Deploy MERN Society Management System on RHEL 10
  hosts: appserver
  become: yes
  gather_facts: yes
  roles:
    - society_management
```

(This is exactly `site.yml` at the repo root.)

Usage
-----

```bash
# Full run
ansible-playbook site.yml -i inventory/inventory.ini --ask-vault-pass

# Just re-issue/renew SSL
ansible-playbook site.yml -i inventory/inventory.ini --ask-vault-pass --tags part_f

# Everything except SSL (e.g. while DNS isn't ready yet)
ansible-playbook site.yml -i inventory/inventory.ini --ask-vault-pass --skip-tags part_f
```

License
-------

BSD

Author Information
-------------------

Learning project — see the main repo [README](../../README.md).
