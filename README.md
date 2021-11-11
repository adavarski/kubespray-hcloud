# Hetzner k8s cluster: kubespray-based (HA: multi-master)

Note: This module assumes that you have a public key in `~/.ssh/id_rsa.pub` and a Hetzner API token ready to be used.

## Usage

```
### Provisioning hcloud infrastructure:

$ cat terraform.tfvars 
multi_master = true
master_node_count  = 3
worker_node_count  = 2
hcloud_token = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXxx" 

$ terraform init 
$ terraform plan
$ terraform apply

### Use kubespray for k8s cluster provisioning (see bellow examle)

```

## Example:

```

$ terraform apply

...

Apply complete! Resources: 14 added, 0 changed, 0 destroyed.

Outputs:

master_ip_address = [
  "10.0.1.3",
  "10.0.1.4",
  "10.0.1.5",
]
worker_ip_address = [
  "10.0.1.1",
  "10.0.1.2",
]
worker_ip_addresses = [
  "10.0.1.1",
  "10.0.1.2",
]


### hcloud cli
$ export HCLOUD_TOKEN="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXx"

$ hcloud server list
ID         NAME         STATUS    IPV4            IPV6                     DATACENTER
15913825   worker-001   running   23.88.61.1      2a01:4f8:c0c:2d3b::/64   nbg1-dc3
15913826   worker-002   running   49.12.226.241   2a01:4f8:c0c:aa9a::/64   nbg1-dc3
15913827   master-001   running   23.88.101.36    2a01:4f8:c2c:329f::/64   nbg1-dc3
15913828   master-002   running   23.88.107.54    2a01:4f8:c0c:963d::/64   nbg1-dc3
15913829   master-003   running   49.12.225.212   2a01:4f8:c2c:232b::/64   nbg1-dc3

$ hcloud network list
ID        NAME                               IP RANGE      SERVERS
1264676   hetzner-kubeadm-ddBVm�rZZtqmHetB   10.0.0.0/16   5 servers


### run on master-001: 23.88.101.36

$ scp /home/davar/.ssh/id_rsa root@23.88.101.36:~/.ssh/

$ ssh root@23.88.101.36


root@master-001:~# apt update
root@master-001:~# git clone https://github.com/kubernetes-sigs/kubespray
root@master-001:~# cd kubespray
root@master-001:~/kubespray# apt install python3-pip
root@master-001:~/kubespray# pip3 install -r requirements.txt
root@master-001:~/kubespray# cp -rfp inventory/sample inventory/mycluster
root@master-001:~/kubespray# declare -a IPS=(10.0.1.3 10.0.1.4 10.0.1.5 10.0.1.1 10.0.1.2)
root@master-001:~/kubespray# CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
DEBUG: Adding group all
DEBUG: Adding group kube_control_plane
DEBUG: Adding group kube_node
DEBUG: Adding group etcd
DEBUG: Adding group k8s_cluster
DEBUG: Adding group calico_rr
DEBUG: adding host node1 to group all
DEBUG: adding host node2 to group all
DEBUG: adding host node3 to group all
DEBUG: adding host node4 to group all
DEBUG: adding host node5 to group all
DEBUG: adding host node1 to group etcd
DEBUG: adding host node2 to group etcd
DEBUG: adding host node3 to group etcd
DEBUG: adding host node1 to group kube_control_plane
DEBUG: adding host node2 to group kube_control_plane
DEBUG: adding host node1 to group kube_node
DEBUG: adding host node2 to group kube_node
DEBUG: adding host node3 to group kube_node
DEBUG: adding host node4 to group kube_node
DEBUG: adding host node5 to group kube_node
root@master-001:~/kubespray# vi inventory/mycluster/hosts.yaml
root@master-001:~/kubespray# cat inventory/mycluster/hosts.yaml
all:
  hosts:
    master-001:
      ansible_host: 10.0.1.3
      ip: 10.0.1.3
      access_ip: 10.0.1.3
    master-002:
      ansible_host: 10.0.1.4
      ip: 10.0.1.4
      access_ip: 10.0.1.4
    master-003:
      ansible_host: 10.0.1.5
      ip: 10.0.1.5
      access_ip: 10.0.1.5
    worker-001:
      ansible_host: 10.0.1.1
      ip: 10.0.1.1
      access_ip: 10.0.1.1
    worker-002:
      ansible_host: 10.0.1.2
      ip: 10.0.1.2
      access_ip: 10.0.1.2
  children:
    kube_control_plane:
      hosts:
        master-001:
        master-002:
        master-003:
    kube_node:
      hosts:
        master-001:
        master-002:
        master-003:
        worker-001:
        worker-002:
    etcd:
      hosts:
        master-001:
        master-002:
        master-003:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
      
root@master-001:~/kubespray# for i in {1..5}; do ssh 10.0.1.$i date;done
Thu 11 Nov 2021 06:28:32 AM UTC
Thu 11 Nov 2021 06:28:33 AM UTC
Thu 11 Nov 2021 06:28:34 AM UTC
Thu 11 Nov 2021 06:28:35 AM UTC
Thu 11 Nov 2021 06:28:35 AM UTC

root@master-001:~/kubespray# ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
...
PLAY RECAP **********************************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
master-001                 : ok=587  changed=121  unreachable=0    failed=0    skipped=1170 rescued=0    ignored=2   
master-002                 : ok=521  changed=108  unreachable=0    failed=0    skipped=1027 rescued=0    ignored=1   
master-003                 : ok=523  changed=109  unreachable=0    failed=0    skipped=1025 rescued=0    ignored=1   
worker-001                 : ok=370  changed=74   unreachable=0    failed=0    skipped=653  rescued=0    ignored=1   
worker-002                 : ok=370  changed=74   unreachable=0    failed=0    skipped=653  rescued=0    ignored=1   

Thursday 11 November 2021  09:45:36 +0000 (0:00:00.168)       0:13:30.964 ***** 
=============================================================================== 
kubernetes/preinstall : Install packages requirements ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 41.37s
container-engine/docker : ensure docker packages are installed ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 31.34s
kubernetes/control-plane : Joining control plane node to the cluster. --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 26.12s
kubernetes/kubeadm : Join to cluster ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 25.43s
kubernetes/control-plane : kubeadm | Initialize first master ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 18.72s
kubernetes/preinstall : Update package management cache (APT) ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 13.26s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 10.38s
etcd : Gen_certs | Write etcd member and admin certs to other etcd nodes ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.57s
etcd : Gen_certs | Write etcd member and admin certs to other etcd nodes ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.05s
network_plugin/calico : Calico | Create calico manifests ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 8.17s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 8.16s
container-engine/docker : ensure docker-ce repository is enabled --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 8.14s
download : download_container | Download image if required --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 7.02s
etcd : Gen_certs | Write node certs to other etcd nodes ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 6.88s
etcd : Gen_certs | Write node certs to other etcd nodes ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 6.36s
download : download | Download files / images ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 6.29s
download : download_container | Download image if required --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.83s
download : download_container | Download image if required --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.67s
network_plugin/calico : Start Calico resources --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.64s
etcd : Configure | Check if etcd cluster is healthy ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.54s

### Laptop/Worksation (k8s deploy hcloud add-ons/etc.)

$ scp root@23.88.101.36:/etc/kubernetes/admin.conf .
$ sed -i "s/127.0.0.1/23.88.101.36/" admin.conf 
$ export KUBECONFIG=./admin.conf 

$ kubectl get node -o wide
NAME         STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master-001   Ready    control-plane,master   5m38s   v1.22.3   10.0.1.3      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
master-002   Ready    control-plane,master   5m15s   v1.22.3   10.0.1.4      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
master-003   Ready    control-plane,master   5m5s    v1.22.3   10.0.1.5      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
worker-001   Ready    <none>                 4m5s    v1.22.3   10.0.1.1      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
worker-002   Ready    <none>                 4m5s    v1.22.3   10.0.1.2      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9


$ kubectl get all --all-namespaces  -o wide
NAMESPACE     NAME                                           READY   STATUS    RESTARTS        AGE     IP             NODE         NOMINATED NODE   READINESS GATES
kube-system   pod/calico-kube-controllers-684bcfdc59-4v6tm   1/1     Running   1 (3m10s ago)   3m11s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/calico-node-4l8jv                          1/1     Running   0               3m49s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/calico-node-5nms2                          1/1     Running   0               3m49s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/calico-node-bnv85                          1/1     Running   0               3m49s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/calico-node-gkl7t                          1/1     Running   0               3m49s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/calico-node-q2w2d                          1/1     Running   0               3m49s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/coredns-8474476ff8-57n8n                   1/1     Running   0               2m40s   10.233.94.1    master-003   <none>           <none>
kube-system   pod/coredns-8474476ff8-vwwgs                   1/1     Running   0               2m46s   10.233.97.1    master-002   <none>           <none>
kube-system   pod/dns-autoscaler-5ffdc7f89d-mnjmv            1/1     Running   0               2m43s   10.233.123.1   master-001   <none>           <none>
kube-system   pod/kube-apiserver-master-001                  1/1     Running   0               5m58s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-apiserver-master-002                  1/1     Running   0               5m37s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-apiserver-master-003                  1/1     Running   0               5m28s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-controller-manager-master-001         1/1     Running   1               5m56s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-controller-manager-master-002         1/1     Running   1               5m38s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-controller-manager-master-003         1/1     Running   1               5m29s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-proxy-6jjm8                           1/1     Running   0               4m21s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-proxy-8nndb                           1/1     Running   0               4m21s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/kube-proxy-g559r                           1/1     Running   0               4m21s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/kube-proxy-hvzxh                           1/1     Running   0               4m21s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-proxy-nsvzd                           1/1     Running   0               4m21s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-scheduler-master-001                  1/1     Running   1               5m56s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-scheduler-master-002                  1/1     Running   1               5m39s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-scheduler-master-003                  1/1     Running   1               5m28s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/nginx-proxy-worker-001                     1/1     Running   0               4m27s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/nginx-proxy-worker-002                     1/1     Running   0               4m28s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/nodelocaldns-6trcw                         1/1     Running   0               2m41s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/nodelocaldns-gkv4p                         1/1     Running   0               2m41s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/nodelocaldns-mhj27                         1/1     Running   0               2m41s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/nodelocaldns-tnx6k                         1/1     Running   0               2m41s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/nodelocaldns-zmdb6                         1/1     Running   0               2m41s   10.0.1.4       master-002   <none>           <none>

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
default       service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP                  6m      <none>
kube-system   service/coredns      ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   2m46s   k8s-app=kube-dns

NAMESPACE     NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE     CONTAINERS    IMAGES                                     SELECTOR
kube-system   daemonset.apps/calico-node    5         5         5       5            5           kubernetes.io/os=linux   3m50s   calico-node   quay.io/calico/node:v3.20.2                k8s-app=calico-node
kube-system   daemonset.apps/kube-proxy     5         5         5       5            5           kubernetes.io/os=linux   5m58s   kube-proxy    k8s.gcr.io/kube-proxy:v1.22.3              k8s-app=kube-proxy
kube-system   daemonset.apps/nodelocaldns   5         5         5       5            5           kubernetes.io/os=linux   2m41s   node-cache    k8s.gcr.io/dns/k8s-dns-node-cache:1.21.1   k8s-app=nodelocaldns

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                IMAGES                                                       SELECTOR
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           3m12s   calico-kube-controllers   quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers
kube-system   deployment.apps/coredns                   2/2     2            2           2m48s   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns
kube-system   deployment.apps/dns-autoscaler            1/1     1            1           2m46s   autoscaler                k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE     CONTAINERS                IMAGES                                                       SELECTOR
kube-system   replicaset.apps/calico-kube-controllers-684bcfdc59   1         1         1       3m12s   calico-kube-controllers   quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers,pod-template-hash=684bcfdc59
kube-system   replicaset.apps/coredns-8474476ff8                   2         2         2       2m48s   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns,pod-template-hash=8474476ff8
kube-system   replicaset.apps/dns-autoscaler-5ffdc7f89d            1         1         1       2m46s   autoscaler                k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler,pod-template-hash=5ffdc7f89d


### Apply hcloud add-ons: CCM & CSI

$ export HCLOUD_TOKEN="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

$ hcloud network list
ID        NAME                               IP RANGE      SERVERS
1264461   hetzner-kubeadm-dMOqiTT5rFOzP@h5   10.0.0.0/16   5 servers

$ cat hcloud-add-ons/CCM-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: hcloud
  namespace: kube-system
stringData:
  token: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  network: "1264676"

$ cat hcloud-add-ons/CSI-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: hcloud-csi
  namespace: kube-system
stringData:
  token: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXx"
  

$ kubectl apply -f hcloud-add-ons/CCM-secret.yaml
secret/hcloud created
$ kubectl apply -f https://raw.githubusercontent.com/hetznercloud/hcloud-cloud-controller-manager/master/deploy/ccm-networks.yaml
serviceaccount/cloud-controller-manager created
clusterrolebinding.rbac.authorization.k8s.io/system:cloud-controller-manager created
Warning: spec.template.metadata.annotations[scheduler.alpha.kubernetes.io/critical-pod]: non-functional in v1.16+; use the "priorityClassName" field instead
deployment.apps/hcloud-cloud-controller-manager created
$ kubectl apply -f hcloud-add-ons/CSI-secret.yaml 
secret/hcloud-csi created
$ kubectl apply -f https://raw.githubusercontent.com/hetznercloud/csi-driver/master/deploy/kubernetes/hcloud-csi.yml
csidriver.storage.k8s.io/csi.hetzner.cloud created
storageclass.storage.k8s.io/hcloud-volumes created
serviceaccount/hcloud-csi created
clusterrole.rbac.authorization.k8s.io/hcloud-csi created
clusterrolebinding.rbac.authorization.k8s.io/hcloud-csi created
statefulset.apps/hcloud-csi-controller created
daemonset.apps/hcloud-csi-node created
service/hcloud-csi-controller-metrics created
service/hcloud-csi-node-metrics created

$ kubectl get all --all-namespaces -o wide
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS        AGE     IP             NODE         NOMINATED NODE   READINESS GATES
kube-system   pod/calico-kube-controllers-684bcfdc59-4v6tm           1/1     Running   1 (6m32s ago)   6m33s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/calico-node-4l8jv                                  1/1     Running   0               7m11s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/calico-node-5nms2                                  1/1     Running   0               7m11s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/calico-node-bnv85                                  1/1     Running   0               7m11s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/calico-node-gkl7t                                  1/1     Running   0               7m11s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/calico-node-q2w2d                                  1/1     Running   0               7m11s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/coredns-8474476ff8-57n8n                           1/1     Running   0               6m2s    10.233.94.1    master-003   <none>           <none>
kube-system   pod/coredns-8474476ff8-vwwgs                           1/1     Running   0               6m8s    10.233.97.1    master-002   <none>           <none>
kube-system   pod/dns-autoscaler-5ffdc7f89d-mnjmv                    1/1     Running   0               6m5s    10.233.123.1   master-001   <none>           <none>
kube-system   pod/hcloud-cloud-controller-manager-75c979d97c-fjvcw   1/1     Running   0               86s     10.0.1.1       worker-001   <none>           <none>
kube-system   pod/hcloud-csi-controller-0                            5/5     Running   0               19s     10.233.110.1   worker-001   <none>           <none>
kube-system   pod/hcloud-csi-node-5z7j4                              3/3     Running   0               19s     10.233.123.2   master-001   <none>           <none>
kube-system   pod/hcloud-csi-node-dprgb                              3/3     Running   0               19s     10.233.110.2   worker-001   <none>           <none>
kube-system   pod/hcloud-csi-node-st85c                              3/3     Running   0               19s     10.233.112.1   worker-002   <none>           <none>
kube-system   pod/hcloud-csi-node-v58rv                              3/3     Running   0               19s     10.233.97.2    master-002   <none>           <none>
kube-system   pod/hcloud-csi-node-w9sb9                              3/3     Running   0               19s     10.233.94.2    master-003   <none>           <none>
kube-system   pod/kube-apiserver-master-001                          1/1     Running   0               9m20s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-apiserver-master-002                          1/1     Running   0               8m59s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-apiserver-master-003                          1/1     Running   0               8m50s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-controller-manager-master-001                 1/1     Running   1               9m18s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-controller-manager-master-002                 1/1     Running   1               9m      10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-controller-manager-master-003                 1/1     Running   1               8m51s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-proxy-6jjm8                                   1/1     Running   0               7m43s   10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-proxy-8nndb                                   1/1     Running   0               7m43s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/kube-proxy-g559r                                   1/1     Running   0               7m43s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/kube-proxy-hvzxh                                   1/1     Running   0               7m43s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-proxy-nsvzd                                   1/1     Running   0               7m43s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/kube-scheduler-master-001                          1/1     Running   1               9m18s   10.0.1.3       master-001   <none>           <none>
kube-system   pod/kube-scheduler-master-002                          1/1     Running   1               9m1s    10.0.1.4       master-002   <none>           <none>
kube-system   pod/kube-scheduler-master-003                          1/1     Running   1               8m50s   10.0.1.5       master-003   <none>           <none>
kube-system   pod/nginx-proxy-worker-001                             1/1     Running   0               7m49s   10.0.1.1       worker-001   <none>           <none>
kube-system   pod/nginx-proxy-worker-002                             1/1     Running   0               7m50s   10.0.1.2       worker-002   <none>           <none>
kube-system   pod/nodelocaldns-6trcw                                 1/1     Running   0               6m3s    10.0.1.1       worker-001   <none>           <none>
kube-system   pod/nodelocaldns-gkv4p                                 1/1     Running   0               6m3s    10.0.1.3       master-001   <none>           <none>
kube-system   pod/nodelocaldns-mhj27                                 1/1     Running   0               6m3s    10.0.1.2       worker-002   <none>           <none>
kube-system   pod/nodelocaldns-tnx6k                                 1/1     Running   0               6m3s    10.0.1.5       master-003   <none>           <none>
kube-system   pod/nodelocaldns-zmdb6                                 1/1     Running   0               6m3s    10.0.1.4       master-002   <none>           <none>

NAMESPACE     NAME                                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
default       service/kubernetes                      ClusterIP   10.233.0.1     <none>        443/TCP                  9m22s   <none>
kube-system   service/coredns                         ClusterIP   10.233.0.3     <none>        53/UDP,53/TCP,9153/TCP   6m8s    k8s-app=kube-dns
kube-system   service/hcloud-csi-controller-metrics   ClusterIP   10.233.3.8     <none>        9189/TCP                 18s     app=hcloud-csi-controller
kube-system   service/hcloud-csi-node-metrics         ClusterIP   10.233.61.82   <none>        9189/TCP                 18s     app=hcloud-csi

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE     CONTAINERS                                                   IMAGES                                                                                                                                     SELECTOR
kube-system   daemonset.apps/calico-node       5         5         5       5            5           kubernetes.io/os=linux   7m12s   calico-node                                                  quay.io/calico/node:v3.20.2                                                                                                                k8s-app=calico-node
kube-system   daemonset.apps/hcloud-csi-node   5         5         5       5            5           <none>                   19s     csi-node-driver-registrar,hcloud-csi-driver,liveness-probe   k8s.gcr.io/sig-storage/csi-node-driver-registrar:v2.2.0,hetznercloud/hcloud-csi-driver:1.6.0,k8s.gcr.io/sig-storage/livenessprobe:v2.3.0   app=hcloud-csi
kube-system   daemonset.apps/kube-proxy        5         5         5       5            5           kubernetes.io/os=linux   9m20s   kube-proxy                                                   k8s.gcr.io/kube-proxy:v1.22.3                                                                                                              k8s-app=kube-proxy
kube-system   daemonset.apps/nodelocaldns      5         5         5       5            5           kubernetes.io/os=linux   6m3s    node-cache                                                   k8s.gcr.io/dns/k8s-dns-node-cache:1.21.1                                                                                                   k8s-app=nodelocaldns

NAMESPACE     NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                        IMAGES                                                       SELECTOR
kube-system   deployment.apps/calico-kube-controllers           1/1     1            1           6m34s   calico-kube-controllers           quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers
kube-system   deployment.apps/coredns                           2/2     2            2           6m10s   coredns                           k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns
kube-system   deployment.apps/dns-autoscaler                    1/1     1            1           6m8s    autoscaler                        k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler
kube-system   deployment.apps/hcloud-cloud-controller-manager   1/1     1            1           86s     hcloud-cloud-controller-manager   hetznercloud/hcloud-cloud-controller-manager:v1.9.1          app=hcloud-cloud-controller-manager

NAMESPACE     NAME                                                         DESIRED   CURRENT   READY   AGE     CONTAINERS                        IMAGES                                                       SELECTOR
kube-system   replicaset.apps/calico-kube-controllers-684bcfdc59           1         1         1       6m34s   calico-kube-controllers           quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers,pod-template-hash=684bcfdc59
kube-system   replicaset.apps/coredns-8474476ff8                           2         2         2       6m10s   coredns                           k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns,pod-template-hash=8474476ff8
kube-system   replicaset.apps/dns-autoscaler-5ffdc7f89d                    1         1         1       6m8s    autoscaler                        k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler,pod-template-hash=5ffdc7f89d
kube-system   replicaset.apps/hcloud-cloud-controller-manager-75c979d97c   1         1         1       86s     hcloud-cloud-controller-manager   hetznercloud/hcloud-cloud-controller-manager:v1.9.1          app=hcloud-cloud-controller-manager,pod-template-hash=75c979d97c

NAMESPACE     NAME                                     READY   AGE   CONTAINERS                                                                  IMAGES
kube-system   statefulset.apps/hcloud-csi-controller   1/1     19s   csi-attacher,csi-resizer,csi-provisioner,hcloud-csi-driver,liveness-probe   k8s.gcr.io/sig-storage/csi-attacher:v3.2.1,k8s.gcr.io/sig-storage/csi-resizer:v1.2.0,k8s.gcr.io/sig-storage/csi-provisioner:v2.2.2,hetznercloud/hcloud-csi-driver:1.6.0,k8s.gcr.io/sig-storage/livenessprobe:v2.3.0

### Test hcloud LB & PVC creation:

$ kubectl apply -f hello/hello-default.yaml 
deployment.apps/hello-kubernetes created
service/hello-kubernetes created
persistentvolumeclaim/csi-pvc created
```

## Ref: 

- https://github.com/kubernetes-sigs/kubespray/blob/master/README.md
- https://github.com/kubernetes-sigs/kubespray/blob/master/docs/getting-started.md
- https://kubernetes.io/docs/setup/production-environment/tools/kubespray/
