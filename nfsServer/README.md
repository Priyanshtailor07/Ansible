# NFS Server + Autofs Client

Two small playbooks that together set up an **NFS export** on one node and an
**Autofs auto-mounting client** on another — a classic client/server storage
lab.

## Files

| File | Purpose |
|---|---|
| `nfs-server.yml` | Runs on `node1`. Installs `nfs-utils`, opens the NFS port in `firewalld`, creates `/nfs_share`, exports it via `/etc/exports.d/abc.exports`, and runs `exportfs -rv` |
| `autofs-server.yml` | Runs on `node2`. Installs `nfs-utils` + `autofs`, creates `/data`, registers an autofs map that mounts `node1:/nfs_share` on demand |
| `nfs_vars.yml` | Variables for the server playbook: `pkg`, `service`, `port` |

## Variables (`nfs_vars.yml`)

```yaml
pkg: nfs-utils
service: nfs-server.service
port: 2049/tcp
```

## What happens

**`nfs-server.yml`** (hosts: `node1`)
1. Installs `nfs-utils`
2. Starts/enables `nfs-server.service`
3. Opens `2049/tcp` in `firewalld`
4. Creates `/nfs_share` (mode `0777`)
5. Adds an export line for `/nfs_share` to `172.31.10.69` with `rw,sync`
6. Runs `exportfs -rv` to apply the export

**`autofs-server.yml`** (hosts: `node2`)
1. Installs `nfs-utils` and `autofs`
2. Starts/enables the `autofs` service
3. Creates `/data`
4. Adds a master map entry (`/- /etc/auto.misc`) and a mount entry pointing
   `/data` at `172.31.1.147:/nfs_share`
5. Restarts `autofs`

> **Note:** the NFS server IP hard-coded in the export line and the client IP
> hard-coded in `auto.misc` (`172.31.10.69` / `172.31.1.147`) are specific to
> the lab environment this was built against — update them to match your own
> `node1`/`node2` addresses before running.

## Requirements

- RHEL/CentOS/Rocky/Alma targets (`yum`, `firewalld`)
- `node1` and `node2` defined in the inventory in use (default: `../myhosts`)
- `become: true` (set globally in `../ansible.cfg`)

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
