Database Role
=============

Installs and configures **MariaDB** on a RHEL-family host, then creates an
application database and user — designed to run alongside the `wordpress`
role as the DB tier of a two-node WordPress deployment (see
`wordpress_app.yml` at the repo root).

Requirements
------------

- RHEL/CentOS/Rocky/Alma managed node
- `become: true`
- The `community.mysql` collection (for the `mysql_db` and `mysql_user`
  modules) and `python3-PyMySQL` on the target (installed by the role itself
  via `yum`):
```bash
  ansible-galaxy collection install community.mysql
```

Role Variables
--------------

Defined in `vars/main.yml`, which is **encrypted with Ansible Vault**:

| Variable | Purpose |
|---|---|
| `db_name` | Database created for the application (e.g. WordPress) |
| `db_user` | MySQL user created and granted `ALL` privileges on `db_name.*` |
| `db_password` | Password for `db_user` |
| `db_hostname` | Host pattern the user is allowed to connect from (e.g. the WordPress node's IP, or `%`) |

View/edit the encrypted vars with:

```bash
ansible-vault view roles/database/vars/main.yml
ansible-vault edit roles/database/vars/main.yml
```

Dependencies
------------

None. Expected to run on a **different host** than the `wordpress` role in
the same play (see `wordpress_app.yml`), so `db_hostname` should generally be
set to the WordPress node's address rather than `localhost`.

What it does
------------

1. Installs `mariadb`, `mariadb-server`, `python3-PyMySQL`
2. Starts the `mariadb` service
3. Creates `{{ db_name }}` via the local unix socket
4. Creates `{{ db_user }}` with `ALL` privileges on `{{ db_name }}.*`,
   restricted to connections from `{{ db_hostname }}`
5. Restarts `mariadb`

Example Playbook
-----------------

```yaml
- name: Configure Database Server
  hosts: node2
  become: true
  roles:
    - database
```

(This is exactly how it's used in `wordpress_app.yml` at the repo root.)

License
-------

BSD

Author Information
-------------------

Learning project — see the main repo [README](../../README.md).
