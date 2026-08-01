WordPress Role
===============

Downloads and installs **WordPress** on a RHEL-family Apache host: installs
httpd + php-fpm, fetches and extracts WordPress, templates `wp-config.php`
and an Apache vhost, sets SELinux context, and (re)starts httpd. Designed to
run alongside the `database` role, which provisions the MariaDB backend it
connects to (see `wordpress_app.yml` at the repo root).

Requirements
------------

- RHEL/CentOS/Rocky/Alma managed node with SELinux enabled (`sefcontext` /
  `restorecon` are used)
- `become: true`
- `ansible.posix` collection (for `sefcontext`), plus the `httpd`/`php-fpm`
  packages available in the configured repos

Role Variables
--------------

| Variable | Where | Purpose |
|---|---|---|
| `pkg` | not currently defined — needs adding to `defaults/main.yml` | List of packages to install (`httpd`, `php`, `php-fpm`, `php-mysqlnd`, etc. — the role loops `yum` over `{{ pkg }}`) |
| `wordpress_url` | not currently defined — needs adding | Download URL for the WordPress tarball (e.g. `https://wordpress.org/latest.tar.gz`) |
| DB connection vars used in `wp-config.j2` | `vars/main.yml` (**Vault-encrypted**) | DB name/user/password/host to write into `wp-config.php` — should match the values set for the `database` role |

View/edit the encrypted vars:

```bash
ansible-vault view roles/wordpress/vars/main.yml
ansible-vault edit roles/wordpress/vars/main.yml
```

> `pkg` and `wordpress_url` are referenced in `tasks/main.yml` but not yet
> defined anywhere — add them to `defaults/main.yml` (see example below)
> before running the role.

Templates
---------

| Template | Deployed to | Purpose |
|---|---|---|
| `wp-config.j2` | `/home/admin/wordpress/wp-config.php` | WordPress DB connection config |
| `wordpress.j2` | `/etc/httpd/conf.d/wordpress.conf` | Apache VirtualHost serving `/var/www/html/wordpress` |

Dependencies
------------

Expects a database (MariaDB) to already be reachable — normally provided by
the `database` role running on a separate node in the same play.

What it does
------------

1. Installs `{{ pkg }}` (loop), starts/enables `httpd` and `php-fpm`
2. Creates `/home/admin/wordpress`, downloads WordPress from
   `{{ wordpress_url }}`, extracts it
3. Templates `wp-config.php` into the extracted WordPress directory
4. Copies WordPress into `/var/www/html/wordpress/` owned by `apache:apache`
5. Sets the SELinux file context (`httpd_sys_rw_content_t`) and runs
   `restorecon`
6. Templates the Apache VirtualHost and restarts `httpd`

Example Playbook
-----------------

```yaml
- name: Configure WordPress Server
  hosts: node1
  become: true
  roles:
    - wordpress
```

(This is exactly how it's used in `wordpress_app.yml` at the repo root,
alongside the `database` role running on `node2`.)

License
-------

BSD

Author Information
-------------------

Learning project — see the main repo [README](../../README.md).
