# ANSIBLE User Manual

This manual is designed to guide a new user through the configuration and use of ANSIBLE within this project. Please follow each step carefully to ensure correct playbook execution.

## 1. Installation and Initial Setup

1. **Install ANSIBLE** follow the steps given in this [repository overview](../README.md)
2. Make sure you have SSH access to the remote nodes and your user has sudo privileges.

## 2. Project Structure

- `playbook.yml`: Main playbook for node configuration.
- `inventories/`: Folder with inventory and host variables.
  - `inventory.ini`: Defines groups and nodes.
  - `host_vars/`: Node-specific variables (one `.yml` file per node).
- `roles/`: ANSIBLE roles for different components (k3s, helm, fluidos, etc).

## 3. Important Variable Configuration

Before running any playbook, **you must configure the following variables** in the `inventories/host_vars/` files for each node:

- `ansible_host`: Node IP address.
- `ansible_user`: SSH user for connection.
- `ansible_ssh_private_key_file`: Path to the SSH private key.
- `ansible_become_password`: Sudo password (not stored in the repository, add it manually).
- Node-specific variables, for example:
  - `k3s_node_name`, `liqo_cluster_name`, `fluidos_version`, `rear_port`, `enable_local_discovery`, `third_octect`, `net_interface`.

**Note:** All parameters have default values except passwords, which you must fill in manually.

## 4. Running Playbooks

1. Rename the folder `ansible/inventories/host_vars_dummy` to `ansible/inventories/host_vars` if needed.
2. Edit the `.yml` files in `host_vars` for each node, setting the variables mentioned above.
3. Run the main playbook:

```sh
ansible-playbook -i inventories/inventory.ini playbook.yml
```

You can use tags to run only certain roles, for example:

```sh
ansible-playbook -i inventories/inventory.ini playbook.yml --tags "k3s,helm"
```

## 5. Uninstalling Packages

To uninstall components, check the roles for specific uninstall tasks or remove the corresponding roles from the playbook.

---

**Remember:**
- Do not share your passwords or private keys.
- Check each role's documentation for additional variables.
- If in doubt, review the comments in each role's files.
