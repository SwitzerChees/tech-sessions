### Install multipass

https://multipass.run/

```bash
# Show running instances
multipass list
# Launch new instance with name
multipass launch --name <instance-name>
# Execute shell into instance
multipass shell <instance-name>
# Stop instance
multipass stop <instance-name>
# Start instance
multipass start <instance-name>
# Delete instance
multipass delete <instance-name>
# Cleanup deleted instances
multipass purge
```

### Install k3s Kubernetes (Single-Node)

https://k3s.io/

[K3S Resource Requirements](https://rancher.com/docs/k3s/latest/en/installation/installation-requirements/resource-profiling/)

```bash
curl -sfL https://get.k3s.io | sh -
```

### Install k3s Kubernetes (HA)

https://rancher.com/docs/k3s/latest/en/installation/install-options/

```bash
# ==== FIRST MASTER NODE ====
# Install k3s as single node setup
INSTALL_K3S_EXEC='server --cluster-init'
K3S_TOKEN='supersecret123'
curl -sfL https://get.k3s.io | K3S_TOKEN=$K3S_TOKEN INSTALL_K3S_EXEC=$INSTALL_K3S_EXEC sh -


# ==== ADDITIONAL MASTER NODES ====
# Set needed env variables to join additional master node
K3S_TOKEN='supersecret123'
K3S_URL='https://<master-node-ip>:6443'
INSTALL_K3S_EXEC='server'
# Add additional master node
curl -sfL https://get.k3s.io | K3S_TOKEN=$K3S_TOKEN K3S_URL=$K3S_URL INSTALL_K3S_EXEC=$INSTALL_K3S_EXEC sh -

# ==== ADDITIONAL WORKER NODES ====
# Set needed env variables to join additional master node
K3S_TOKEN='supersecret123'
K3S_URL='https://<master-node-ip>:6443'
INSTALL_K3S_EXEC='agent'
# Add additional master node
curl -sfL https://get.k3s.io | K3S_TOKEN=$K3S_TOKEN K3S_URL=$K3S_URL INSTALL_K3S_EXEC=$INSTALL_K3S_EXEC sh -
```

## Install Kube-VIP (HA Kube-API)
https://kube-vip.io/usage/k3s/
https://kube-vip.io/install_daemonset/#generating-a-manifest
```bash
# Determine your network interface name for example with ip a
export INTERFACE=<your-network-interface>
# Define a virtual IP address
export VIP=<your-VIP>
# Install RBAC resources
curl https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/k3s/server/manifests/kube-vip-rbac.yaml
# Run the kube-vip container interactive to generate the daemon manifests and save it in the k3s manfiest folder to auto deploy it
kubectl run -it kube-vip-init  --image=ghcr.io/kube-vip/kube-vip:main --restart=Never --rm -- manifest daemonset \
    --interface $INTERFACE \
    --address $VIP \
    --inCluster \
    --taint \
    --controlplane \
    --arp \
    --leaderElection > /var/lib/rancher/k3s/server/manifests/kube-vip-daemonset.yaml
```

### Important files

```bash
# kubeconfig location
cat /etc/rancher/k3s/k3s.yaml
# node-token location to join new nodes
cat /var/lib/rancher/k3s/server/node-token
# auto-deploy manifests folder path
/var/lib/rancher/k3s/server/manifests
```

### Cleanup k3s
```bash
# kill actual k3s process
/usr/local/bin/k3s-killall.sh
# delete actual k3s installation (server)
/usr/local/bin/k3s-uninstall.sh
# delete actual k3s installation (agent)
/usr/local/bin/k3s-agent-uninstall.sh
```

### Auto Upgrade K3S
https://rancher.com/docs/k3s/latest/en/upgrades/automated/

Channels: https://update.k3s.io/v1-release/channels

```bash
# Install upgrade controller in system-upgrade namespace
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/latest/download/system-upgrade-controller.yaml
# Apply the service plans
kubectl apply -f upgrade-plans.yml
```
