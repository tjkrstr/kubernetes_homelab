# Kubernetes Home Lab

The following repo mainly consist of configs and notes created and used during the setup of a Kubernetes cluster using old laptops (only two laptops). The goal is to create a simple but fully-fledged Kubernetes setup that we can quickly spin up and keep alive for some time to run simple workloads. Ubuntu server 22.04.3 LTS was installed on each laptop along with the lightweight version of Kubernetes, k3s.

## Suspend and Hibernation
Because the devices are laptops the screen/power saving options will be changed to make sure the laptop stays on even when the lid is closed. This 

```bash
# Disable Suspend and Hibernation deamons 
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
# Check that their status is in fact inactive 
sudo systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
# If necessary enabling the deamons can be done with the following command:
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target


sudo vim /etc/systemd/logind.conf

# Replace whatever values of following to "ignore"
HandleHibernateKey=ignore
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore

# Restart the login service
systemctl restart systemd-logind
```

## K3s setup
In the following project a very 
Setup master node:
```bash
curl -sfL https://get.k3s.io | sh - 
```

Setup worker node:
```bash
# On a different node run the below command. 
# NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token on your server (master node)
$ curl -sfL https://get.k3s.io/ | \
K3S_URL=https://<master_node_ip>:6443 \
K3S_TOKEN=<token> sh -
```

Get all nodes
```bash
kubectl get nodes -o wide
```

# Helm
Install helm

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Check helm version and if helm relevant pods are running
```
helm version
kubectl get pods -n kube-system
```
