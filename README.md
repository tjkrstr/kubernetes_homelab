# Kubernetes Home Lab

The following repo mainly consist of configs and notes created and used during the setup of a Kubernetes cluster using old laptops (only two laptops). The goal is to create a simple but fully fledged Kubernetes setup that we can quickly spin up and keep alive for some time to run simple workloads. Ubuntu server 22.04.3 LTS was installed on each laptop along with the lightweight version of Kubernetes, k3s.

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
K3s has been used during the execution of this project. The [setup](https://k3s.io/) of a k3s cluster is fairly straight forward and requires minimal resources.   

Setup the k3s server (master node) using the following command:
```bash
curl -sfL https://get.k3s.io | sh - 
sudo k3s kubectl get node 
```

Setup worker node we requred the ip of the master node along with its token which is located in /etc/lib/rancher/k3s/server/node-token. Add the worker node using the following command and necessary information.
```bash
# On a different node run the below command. 
# NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token on your server (master node)
$ curl -sfL https://get.k3s.io/ | \
K3S_URL=https://<master_node_ip>:6443 \
K3S_TOKEN=<token> sh -
```

The environment file of the k3s-agent.service located in /etc/systemd/system/k3s-agent.service.env can be checked to see how the added ip and token is stored:

```bash
# cat /etc/systemd/system/k3s-agent.service.env
K3S_TOKEN=K173832c8dd5a175bf2123d840ad491850199cbd19309ab3b37827047cdd6319b04::server:faddf0a734d338cd66d4ab19fb4bed73
K3S_URL=https://10.12.1.40:6443
```

Get all the nodes in the k3s cluster to check if the worker node has been added.
```bash
kubectl get nodes -o wide
```

The worker node has been added to the cluster BUT has not been specified or authorised [cluster access](https://docs.k3s.io/cluster-access). If access to the cluster is needed from the worker node the `kubeconfig` located on the master (k3s server) in /etc/rancher/k3s/k3s.yaml must be copoed onto the machine as ~/.kube/config.

```bash
scp /etc/rancher/k3s/k3s.yaml <worker_node>@<worker_node_ip>:~/.kube/config
```

Replace the value of the `server:` field with the IP or name of your k3s server. kubectl can now manage your K3s cluster.


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
