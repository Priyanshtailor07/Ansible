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
Ansible/
├── ansible.cfg # Local config: default inventory, roles_path, become
├── myhosts # Ad-hoc inventory (node1..node4, groups: rhel/dev/prod/webserver)
├── site.yml # Entry playbook -> runs the society_management role
├── wordpress_app.yml # Entry playbook -> runs database + wordpress roles
├── stop_nodes.yml # Utility: shuts down all hosts in inventory
├── first_playbook.yml # Practice: create a group + user
│
├── inventory/
│ └── inventory.ini # Inventory used by site.yml ([appserver] group)
│
├── facts/ # Custom facts examples (facts.d style)
├── practice/ # Loops, conditionals, blocks, handlers, imports — practice snippets
├── Assignment1/ # Lab tasks 1-5 (basic modules)
├── Assignment2/ # Lab tasks 1-5 (Jinja2 loops/conditions, handlers, block/rescue)
│
├── webServer/ # Standalone playbook: Apache web server (see its own README)
├── nfsServer/ # Standalone playbooks: NFS server + Autofs client (see its own README)
│
└── roles/ # Reusable Ansible roles (standard ansible-galaxy init layout)
├── apache/ # Cross-platform Apache/httpd role (RHEL + Debian) — WIP
├── database/ # MariaDB role — used with wordpress role
├── wordpress/ # WordPress-on-Apache role — used with database role
└── society_management/ # MERN stack + Nginx + Let's Encrypt role
