# ICEWAPP: Photovoltaic _Nowcasting_
This document provides a brief introduction to the implementation of the ICEWAPP use case testbed. For more information about the use case and how FLUIDOS has helped to the development of the project, you can check out this [video](https://vimeo.com/1114021451).

During the development and testing phases of the project, the environment had to be reinstalled from scratch several times. For this reason, automation through Ansible scripts allowed us to save time and facilitate replicability. This guide is inspired by the [documentation](https://github.com/fluidos-project/node/blob/v0.1.0-rc.1/docs/installation/installation.md#manual-installation) provided in the [Node](https://github.com/fluidos-project/node/) repository and in the [intelligent-power-grid-use-case](https://github.com/fluidos-project/intelligent-power-grid-use-case).

The testbed environment involves the following components and tools:
- Two K3s Kubernetes clusters 
  - __CONSUMER__: with 1 ``armv8`` master node (Revolution Pi Connect 4).
  - __PROVIDER__: with 1 ``amd64`` master node.
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/): a declarative GitOps continuous delivery tool for Kubernetes.
- [LIQO: v0.10.3](https://docs.liqo.io/en/v0.10.3/index.html)
- FLUIDOS Node: v0.1.1
- Helm
- Kubernetes-Dashboard
- kube-prometheus-stack

Despite the specified nodes, ANSIBLE variables can be customized to install the testbed in a different setup. 

ANSIBLE scripts are located on [/ansible](ansible) folder, but first a brief overview of ANSIBLE and how to setup it will be given.

# 📘 Introduction to Ansible
## ✅ What is Ansible?
Ansible is a configuration automation and systems management tool, based on an agentless approach that uses SSH to communicate with remote machines. It was developed by Red Hat and is designed to be simple, powerful, and flexible.

## 🛠️ What is Ansible used for?
- Installing and configuring software on servers (Apache, Docker, K3s, RabbitMQ...).
- Infrastructure orchestration (deploying microservices, restarting services, etc.).
- Managing heterogeneous environments (local VMs, edge devices, clouds, containers).
- Automating repetitive tasks (copying files, applying configuration changes, restarting systems...).

All this is achieved through playbooks written in YAML.

## 🔁 How does it work?
- Define tasks in YAML files called playbooks.
- Use SSH to execute these tasks on one or more remote servers.
- No agents required (nothing is installed on managed servers).
- An inventory file defines the target servers.

## 🧭 Why NOT run Ansible directly on Windows?
❌ Windows limitations:
- Compatibility issues → Ansible is based on Linux/POSIX and is not designed to run natively on Windows.
- Lacks full SSH support → Until recent versions, Windows did not include robust SSH support, which Ansible requires.
- Dependency problems → Ansible depends on UNIX tools (bash, python, scp...), which are not available natively on Windows.
- Lack of official support → Official documentation does not recommend or support running Ansible directly on Windows.

## ✅ Why use WSL?
WSL (Windows Subsystem for Linux) is the ideal way to use Ansible on Windows:

- Real Linux environment → Install Ansible just like on Ubuntu.
- Full SSH support → Use SSH keys, ssh-agent, etc., without restrictions.
- Integration with Windows → Edit files with VS Code, access shared folders.
- Lightweight and fast → No need for a full VM or VirtualBox.

# Installing and Configuring Ansible

## Installation

Ansible is installed on your computer. To install it, open a WSL terminal and run:

1. Ensure Python is installed, otherwise install it with:
```bash
sudo apt update
sudo apt install -y python3-pip python3-venv
```

2. Install Ansible with pip and the Kubernetes module:
```bash
pip3 install --user ansible
pip3 install kubernetes
```

Make sure `~/.local/bin` is in your `PATH`:
```bash
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Verify the installation:
```bash
ansible --version
```

Expected output:
```bash
ansible [core 2.x.x]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Feb  4 2025, 14:57:36) [GCC 11.4.0]
```

## Configuring the target machine (VM* or edge device)

*Important: Virtual machines require their own IP address for SSH connections, so use bridge mode networking.*

Ansible connects via SSH. From the host:

### From your host
1. Generate an SSH key:
```bash
ssh-keygen -t rsa -b 4096
```

2. Copy the public key to the VM:
```bash
ssh-copy-id -i /path/to/id_rsa.pub user@VM_IP
```

3. If using root in WSL, copy the key to root’s folder:
```bash
mkdir -p /root/.ssh
cp /mnt/c/Git_repository/eium_ansible/.ssh/id_rsa /root/.ssh/
chmod 600 /root/.ssh/id_rsa
```

4. Add the configuration to `~/.ssh/config`:
```bash
Host <IP_ADDRESS>
    User <USER>
    IdentityFile <PATH_TO_GENERATED_KEY> (e.g. /root/.ssh/id_rsa)
```

### On the VM or edge device
1. Ensure the SSH server is installed and running:
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

2. Check SSH permissions:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

3. Verify SSH server configuration:  
In `/etc/ssh/sshd_config` make sure these lines are present:
```bash
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication yes
```

Restart SSH if changes were made:
```bash
sudo systemctl restart ssh
```

4. Ensure port 22 is open:
```bash
sudo ufw allow ssh
sudo ufw enable
```

5. Install Python and required modules for Ansible:
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
pip3 install six
```

## Final check
If everything is configured correctly, you can connect to the VM/edge from your host with:
```bash
ssh user@ip
```
without needing to enter the password.
