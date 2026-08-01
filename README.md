# Ansible

Personal Ansible learning repository — playbooks, ad‑hoc exercises, and reusable
roles for provisioning RHEL/Ubuntu servers, built while learning configuration
management. It contains standalone practice playbooks plus three "real" mini
projects:

| Project | What it does | Docs |
|---|---|---|
| **Web Server** | Installs & configures Apache (httpd) on RHEL dev/prod nodes | [`webServer/README.md`](webServer/README.md) |
| **NFS Server** | Sets up an NFS export + an Autofs client that mounts it | [`nfsServer/README.md`](nfsServer/README.md) |
| **WordPress + Database** | Two‑tier app: MariaDB (`roles/database`) + WordPress on Apache (`roles/wordpress`) | [`roles/database/README.md`](roles/database/README.md), [`roles/wordpress/README.md`](roles/wordpress/README.md) |
| **Society Management (MERN)** | Full MERN stack deploy behind Nginx with SSL (`roles/society_management`) | [`roles/society_management/README.md`](roles/society_management/README.md) |

---

## Repository layout


```text
Ansible/
├── ansible.cfg                 # Local configuration (inventory, roles_path, become)
├── myhosts                     # Inventory for ad-hoc commands
├── site.yml                    # Main playbook (runs society_management role)
├── wordpress_app.yml           # Deploys MariaDB + WordPress
├── stop_nodes.yml              # Shuts down all managed nodes
├── first_playbook.yml          # Practice playbook (creates group and user)
│
├── inventory/
│   └── inventory.ini           # Inventory used by site.yml
│
├── facts/                      # Custom facts (facts.d examples)
│
├── practice/                   # Practice playbooks (loops, conditions, handlers, blocks, imports)
│
├── Assignment1/                # Basic Ansible module assignments
├── Assignment2/                # Jinja2, handlers, block/rescue assignments
│
├── webServer/                  # Apache web server playbooks
├── nfsServer/                  # NFS server and Autofs client playbooks
│
└── roles/
    ├── apache/                 # Cross-platform Apache role (RHEL & Debian) - WIP
    ├── database/               # MariaDB role
    ├── wordpress/              # WordPress deployment role
    └── society_management/     # MERN Stack + Nginx + Let's Encrypt role
```
