# NFS Server + Autofs Client

Two small playbooks that together set up an **NFS export** on one node and an
**Autofs auto-mounting client** on another — a classic client/server storage
lab.

## Files

| File | Purpose |
|---|---|
| `nfs-server.yml` | Runs on `node1`. Installs `nfs-utils`, opens the NFS port in `firewalld`, creates `/nfs_share`, exports it via `/etc/exports.d/abc.exports`, and runs `exportfs -rv` |
| `autofs-server.yml` | Runs on `node2`. Installs `nfs-utils` + `autofs`, creates `/data`, registers an autofs map that mounts `node1:/nfs_share` on demand |
| `nfs_vars.yml` | **Not committed to git** (see below). Holds `pkg`, `service`, `port`, `nfs_client_ip`, `nfs_server_ip` |

## Variables (`nfs_vars.yml`)

This file is listed in the repo's root `.gitignore`, so it won't exist after
a fresh clone — create it yourself in `nfsServer/` before running either
playbook:

```yaml
pkg: nfs-utils
service: nfs-server.service
port: 2049/tcp
nfs_client_ip: <IP allowed to mount the export, used in nfs-server.yml>
nfs_server_ip: <IP of the NFS server, used in autofs-server.yml>
```

> Previously these IPs were hardcoded directly in the task files. They were
> moved into this gitignored vars file so real network addresses aren't
> committed to a public repo.

## What happens

**`nfs-server.yml`** (hosts: `node1`)
1. Installs `nfs-utils`
2. Starts/enables `nfs-server.service`
3. Opens `2049/tcp` in `firewalld`
4. Creates `/nfs_share` (mode `0777`)
5. Adds an export line for `/nfs_share` to `{{ nfs_client_ip }}` with `rw,sync`
6. Runs `exportfs -rv` to apply the export

**`autofs-server.yml`** (hosts: `node2`)
1. Installs `nfs-utils` and `autofs`
2. Starts/enables the `autofs` service
3. Creates `/data`
4. Adds a master map entry (`/- /etc/auto.misc`) and a mount entry pointing
   `/data` at `{{ nfs_server_ip }}:/nfs_share`
5. Restarts `autofs`

> Note: `autofs-server.yml` currently reads `nfs_server_ip` but doesn't
> declare a `vars_files:` for `nfs_vars.yml` the way `nfs-server.yml` does —
> make sure that variable is available to it (e.g. via `group_vars`,
> `-e`, or add a matching `vars_files:` block) or the play will fail on an
> undefined variable.

## Requirements

- RHEL/CentOS/Rocky/Alma targets (`yum`, `firewalld`)
- `node1` and `node2` defined in the inventory in use (default: `../myhosts`)
- `become: true` (set globally in `../ansible.cfg`)
- A local, untracked `nfsServer/nfs_vars.yml` (see above) — the playbooks
  won't run without it

## Usage

From the repo root, run the server first, then the client:

```bash
ansible-playbook nfsServer/nfs-server.yml
ansible-playbook nfsServer/autofs-server.yml
```

Verify on the client:

```bash
ls /data          # triggers the automount
df -h | grep nfs_share
```
