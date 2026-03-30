# dbaascli PDB Clone

Ansible automation for cloning PDBs on Oracle ExaCS using `dbaascli`.

> PoC/MVP. Only `local_tde` mode tested end-to-end. Passwords go in plain text in the inventory.

## The problem

`dbaascli` is interactive — prompts for passwords, no `--password` flag, no stdin. A plain `shell:` task hangs. The solution is `expect`: writes a temp script, runs it, deletes it.

## TDE modes

| Mode | Description |
|------|-------------|
| `local_tde` | TDE managed locally by each CDB. Simplest. Only this one is tested. |
| `okv` | Oracle Key Vault. Adds grant/revoke phases around the clone. |

## Prerequisites

- Common user `C##...` in the source CDB with `CREATE SESSION`
- Database link in the target CDB pointing to the source
- `expect` installed on the ExaCS nodes

## Inventory

Copy `inventory/clones.yml.example` to `inventory/clones.yml` and fill in your values. The file is gitignored.

Key variables:

```yaml
tde_mode: local_tde
dblink_username: "C##DBLINK"
db_password: "your_password"         # same password sent to all 4 prompts

hosts:
  clone_pdb01:
    ansible_host: 10.0.0.1
    source_db: PROD_CDB
    target_db: TEST_CDB
    source_pdb: PROD_DATA
    target_pdb: PROD_DATA_CLONE
    source_db_connection_string: "10.0.0.1:1521/service_name.subnet.vcn.oraclevcn.com"
```

## Running

```bash
# Check prerequisites only (recommended first run)
ansible-playbook playbook.yml -i inventory/clones.yml -e "run_prereqs=true"

# Run the clone
ansible-playbook playbook.yml -i inventory/clones.yml
```

## Logs

Written on the Ansible controller at `log_dir` (default `/tmp/dbaascli-clones`):

```
/tmp/dbaascli-clones/clone_pdb01_20260330T091234.log
```

Includes Job ID and dbaascli log paths on the remote host (`/var/opt/oracle/log/<target_db>/pdb/remoteClone/`).

