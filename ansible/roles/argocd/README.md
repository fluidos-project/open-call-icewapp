# ArgoCD Role

This role automates the following tasks:
1. Installation of ArgoCD in the Kubernetes cluster.
2. Installation of the ArgoCD CLI.
3. Exposing the ArgoCD server as a NodePort to access the web GUI from outside the cluster.

Additional documentation is provided so that, after installation, you can perform the following tasks:
1. [Change the default password for the _admin_ user.](#change-the-default-password-for-the-admin-user)
2. [Connect ArgoCD to a remote cluster.](#connect-argocd-to-a-remote-cluster)

## Installation Steps

1. **Install ArgoCD in the cluster**
   - The role creates the `argocd` namespace and installs ArgoCD using the official manifest.
   - The ArgoCD server is exposed as a NodePort for external access.

2. **Install the ArgoCD CLI**
   - The CLI is downloaded and installed to `/usr/local/bin/argocd` using the latest stable version.

3. **Change the default admin password**
   - The default password (the name of the `argocd-server` pod) is replaced with a user-defined password for security.

## Change the default password for the _admin_ user
1. Get the default password. Run `argocd admin initial-password -n argocd` to get something like:
```
argocd admin initial-password -n argocd
BSpOacmCYeNqVsd1

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```
2. Get the CONTEXT or ARGOCD_SERVER.
```sh
argocd context
```

3. Log in to ArgoCD, replacing ARGOCD_SERVER with one of the names obtained in the previous step (the one marked with * is the default).
```sh
argocd login <ARGOCD_SERVER>
```
4. Change the password.
```sh
argocd account update-password
```
## Connect ArgoCD to a remote cluster

To manage applications in another Kubernetes cluster from ArgoCD, follow these steps:

1. **On the remote cluster:**
   - Obtain the K3s kubeconfig file (usually at `/etc/rancher/k3s/k3s.yaml`).
   - Copy it to a temporary location, e.g., `/tmp/remote-kubeconfig.yaml`.
     ```sh
     cp /etc/rancher/k3s/k3s.yaml /tmp/remote-kubeconfig.yaml
     ```

2. **Transfer the kubeconfig file:**
   - Use `scp` to send the kubeconfig file to a node in the cluster where ArgoCD is installed:
     ```sh
     scp /tmp/remote-kubeconfig.yaml <user>@<argocd-node-ip>:/tmp/remote-kubeconfig.yaml
     ```
3. **Edit the server in the _remote-kubeconfig.yaml_ file with the master node IP of the remote cluster:**

    - Get the master node IP in the remote cluster with `ip a`.
    - Edit the `clusters.cluster.server` field in `/tmp/remote-kubeconfig.yaml`
    ```sh
    sudo nano /tmp/remote-kubeconfig.yaml
    ```

4. **Connect ArgoCD to the remote cluster:**

   - On the ArgoCD node, run:
     ```sh
     argocd cluster add CONTEXT_NAME --kubeconfig /tmp/remote-kubeconfig.yaml
     ```
   - Replace `CONTEXT_NAME` with the context name found in the kubeconfig file (see with `kubectl config get-contexts --kubeconfig /tmp/remote-kubeconfig.yaml`).

## References
- [ArgoCD CLI Installation](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- [ArgoCD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
