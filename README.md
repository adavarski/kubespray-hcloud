# Hetzner k8s cluster: kybespray-based

This module assumes that you have a public key in `~/.ssh/id_rsa.pub` and a Hetzner API token ready to be used.

Example:

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

worker-002
CPX11 / 40 GB / eu-central
95.217.166.151
Helsinki
3 minutes ago
worker-001
CPX11 / 40 GB / eu-central
65.108.52.118
Helsinki
3 minutes ago
master-003
CPX11 / 40 GB / eu-central
95.217.166.243
Helsinki
3 minutes ago
master-001
CPX11 / 40 GB / eu-central
95.217.166.247
Helsinki
3 minutes ago
master-002
CPX11 / 40 GB / eu-central
95.217.166.203
Helsinki
3 minutes ago

Note: master1: 95.217.166.247

$ scp root@23.88.61.1:/etc/kubernetes/admin.conf .

$ sed -i "s/127.0.0.1/23.88.61.1/" admin.conf 
$ export KUBECONFIG=./admin.conf 

$ ssh root@95.217.166.247

root@master-001:~# apt update
root@master-001:~# apt install python-pip3


root@master-001:~# git clone https://github.com/kubernetes-sigs/kubespray
root@master-001:~# cd kubespray
root@master-001:~/kubespray# pip3 install -r requirements.txt

root@master-001:~/kubespray# cp -rfp inventory/sample inventory/mycluster

declare -a IPS=(10.0.1.3 10.0.1.4 10.0.1.5 10.0.1.1 10.0.1.2)

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
    node1:
      ansible_host: 10.0.1.3
      ip: 10.0.1.3
      access_ip: 10.0.1.3
    node2:
      ansible_host: 10.0.1.4
      ip: 10.0.1.4
      access_ip: 10.0.1.4
    node3:
      ansible_host: 10.0.1.5
      ip: 10.0.1.5
      access_ip: 10.0.1.5
    node4:
      ansible_host: 10.0.1.1
      ip: 10.0.1.1
      access_ip: 10.0.1.1
    node5:
      ansible_host: 10.0.1.2
      ip: 10.0.1.2
      access_ip: 10.0.1.2
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
        node3: 
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
        node5:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}

$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
Tuesday 09 November 2021  10:47:21 +0000 (0:00:00.158)       0:12:41.833 ****** 
=============================================================================== 
container-engine/docker : ensure docker packages are installed ------------------------------------------------------------------------------------------------------------------------------------------------------ 31.01s
kubernetes/control-plane : Joining control plane node to the cluster. ----------------------------------------------------------------------------------------------------------------------------------------------- 26.82s
kubernetes/kubeadm : Join to cluster -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 25.55s
kubernetes/control-plane : kubeadm | Initialize first master -------------------------------------------------------------------------------------------------------------------------------------------------------- 19.80s
kubernetes/preinstall : Update package management cache (APT) ------------------------------------------------------------------------------------------------------------------------------------------------------- 12.22s
etcd : reload etcd -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 10.90s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------------- 9.21s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ---------------------------------------------------------------------------------------------------------------------------------------------------------- 8.66s
kubernetes/preinstall : Install packages requirements ---------------------------------------------------------------------------------------------------------------------------------------------------------------- 8.52s
network_plugin/calico : Calico | Create calico manifests ------------------------------------------------------------------------------------------------------------------------------------------------------------- 7.96s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ----------------------------------------------------------------------------------------------------------------------------------------------- 7.73s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------------- 7.41s
etcd : Gen_certs | Write etcd member and admin certs to other etcd nodes --------------------------------------------------------------------------------------------------------------------------------------------- 7.19s
download : download | Download files / images ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 7.08s
etcd : Gen_certs | Write etcd member and admin certs to other etcd nodes --------------------------------------------------------------------------------------------------------------------------------------------- 7.02s
container-engine/docker : ensure docker-ce repository is enabled ----------------------------------------------------------------------------------------------------------------------------------------------------- 6.90s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------------- 6.61s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------------- 6.33s
etcd : wait for etcd up ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 6.10s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------------- 6.09s
root@master-001:~/kubespray# 

### Laptop/Worksation 

$ scp root@95.217.166.247:/etc/kubernetes/admin.conf .
admin.conf                                                                                                                                                                                 100% 5649   113.4KB/s   00:00    
$ sed -i "s/127.0.0.1/95.217.166.247/" admin.conf 
$ export KUBECONFIG=./admin.conf 

$ kubectl get node -o wide
NAME    STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node1   Ready    control-plane,master   12m   v1.22.3   10.0.1.3      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
node2   Ready    control-plane,master   12m   v1.22.3   10.0.1.4      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
node3   Ready    control-plane,master   11m   v1.22.3   10.0.1.5      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
node4   Ready    <none>                 10m   v1.22.3   10.0.1.1      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9
node5   Ready    <none>                 10m   v1.22.3   10.0.1.2      <none>        Ubuntu 20.04.3 LTS   5.4.0-89-generic   docker://20.10.9

$ kubectl get all --all-namespaces  -o wide
NAMESPACE     NAME                                           READY   STATUS    RESTARTS      AGE     IP            NODE    NOMINATED NODE   READINESS GATES
kube-system   pod/calico-kube-controllers-684bcfdc59-9tfjp   1/1     Running   1 (10m ago)   10m     10.0.1.1      node4   <none>           <none>
kube-system   pod/calico-node-6vgcl                          1/1     Running   0             10m     10.0.1.3      node1   <none>           <none>
kube-system   pod/calico-node-q4752                          1/1     Running   0             10m     10.0.1.1      node4   <none>           <none>
kube-system   pod/calico-node-qzsfl                          1/1     Running   0             10m     10.0.1.2      node5   <none>           <none>
kube-system   pod/calico-node-t7g7j                          1/1     Running   0             10m     10.0.1.4      node2   <none>           <none>
kube-system   pod/calico-node-v5trf                          1/1     Running   0             10m     10.0.1.5      node3   <none>           <none>
kube-system   pod/coredns-8474476ff8-fqxhd                   1/1     Running   0             9m52s   10.233.92.1   node3   <none>           <none>
kube-system   pod/coredns-8474476ff8-nzs8b                   1/1     Running   0             9m47s   10.233.90.1   node1   <none>           <none>
kube-system   pod/dns-autoscaler-5ffdc7f89d-9fwvn            1/1     Running   0             9m48s   10.233.96.1   node2   <none>           <none>
kube-system   pod/kube-apiserver-node1                       1/1     Running   0             12m     10.0.1.3      node1   <none>           <none>
kube-system   pod/kube-apiserver-node2                       1/1     Running   0             12m     10.0.1.4      node2   <none>           <none>
kube-system   pod/kube-apiserver-node3                       1/1     Running   0             12m     10.0.1.5      node3   <none>           <none>
kube-system   pod/kube-controller-manager-node1              1/1     Running   1             12m     10.0.1.3      node1   <none>           <none>
kube-system   pod/kube-controller-manager-node2              1/1     Running   1             12m     10.0.1.4      node2   <none>           <none>
kube-system   pod/kube-controller-manager-node3              1/1     Running   1             12m     10.0.1.5      node3   <none>           <none>
kube-system   pod/kube-proxy-88r6n                           1/1     Running   0             11m     10.0.1.5      node3   <none>           <none>
kube-system   pod/kube-proxy-hnk5p                           1/1     Running   0             11m     10.0.1.3      node1   <none>           <none>
kube-system   pod/kube-proxy-ql2qc                           1/1     Running   0             11m     10.0.1.1      node4   <none>           <none>
kube-system   pod/kube-proxy-tx4hj                           1/1     Running   0             11m     10.0.1.4      node2   <none>           <none>
kube-system   pod/kube-proxy-wv8rk                           1/1     Running   0             11m     10.0.1.2      node5   <none>           <none>
kube-system   pod/kube-scheduler-node1                       1/1     Running   1             12m     10.0.1.3      node1   <none>           <none>
kube-system   pod/kube-scheduler-node2                       1/1     Running   1             12m     10.0.1.4      node2   <none>           <none>
kube-system   pod/kube-scheduler-node3                       1/1     Running   1             12m     10.0.1.5      node3   <none>           <none>
kube-system   pod/nginx-proxy-node4                          1/1     Running   0             11m     10.0.1.1      node4   <none>           <none>
kube-system   pod/nginx-proxy-node5                          1/1     Running   0             11m     10.0.1.2      node5   <none>           <none>
kube-system   pod/nodelocaldns-7q6x5                         1/1     Running   0             9m47s   10.0.1.3      node1   <none>           <none>
kube-system   pod/nodelocaldns-fcsdk                         1/1     Running   0             9m47s   10.0.1.5      node3   <none>           <none>
kube-system   pod/nodelocaldns-hchlc                         1/1     Running   0             9m47s   10.0.1.1      node4   <none>           <none>
kube-system   pod/nodelocaldns-j4ztx                         1/1     Running   0             9m47s   10.0.1.2      node5   <none>           <none>
kube-system   pod/nodelocaldns-sz84l                         1/1     Running   0             9m47s   10.0.1.4      node2   <none>           <none>

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
default       service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP                  12m     <none>
kube-system   service/coredns      ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   9m52s   k8s-app=kube-dns

NAMESPACE     NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE     CONTAINERS    IMAGES                                     SELECTOR
kube-system   daemonset.apps/calico-node    5         5         5       5            5           kubernetes.io/os=linux   10m     calico-node   quay.io/calico/node:v3.20.2                k8s-app=calico-node
kube-system   daemonset.apps/kube-proxy     5         5         5       5            5           kubernetes.io/os=linux   12m     kube-proxy    k8s.gcr.io/kube-proxy:v1.22.3              k8s-app=kube-proxy
kube-system   daemonset.apps/nodelocaldns   5         5         5       5            5           kubernetes.io/os=linux   9m47s   node-cache    k8s.gcr.io/dns/k8s-dns-node-cache:1.17.1   k8s-app=nodelocaldns

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                IMAGES                                                       SELECTOR
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           10m     calico-kube-controllers   quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers
kube-system   deployment.apps/coredns                   2/2     2            2           9m53s   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns
kube-system   deployment.apps/dns-autoscaler            1/1     1            1           9m51s   autoscaler                k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE     CONTAINERS                IMAGES                                                       SELECTOR
kube-system   replicaset.apps/calico-kube-controllers-684bcfdc59   1         1         1       10m     calico-kube-controllers   quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers,pod-template-hash=684bcfdc59
kube-system   replicaset.apps/coredns-8474476ff8                   2         2         2       9m54s   coredns                   k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns,pod-template-hash=8474476ff8
kube-system   replicaset.apps/dns-autoscaler-5ffdc7f89d            1         1         1       9m52s   autoscaler                k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler,pod-template-hash=5ffdc7f89d

$ export HCLOUD_TOKEN="YVc08n6Z3ev3keXupzREuOfkFVp2aTZ4HNdOZ80Mm5Xmot8GsQ6UelYz2Z8KNeR2"

$ hcloud network list
ID        NAME                               IP RANGE      SERVERS
1262393   hetzner-kubeadm-fktytPQ3PQlw44L9   10.0.0.0/16   5 servers

$ cat hcloud-add-ons/CCM-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: hcloud
  namespace: kube-system
stringData:
  token: "YVc08n6Z3ev3keXupzREuOfkFVp2aTZ4HNdOZ80Mm5Xmot8GsQ6UelYz2Z8KNeR2"
  network: "1262393"

### Apply hcloud add-ons: CCM & VSI

$ kubectl apply -f hcloud-add-ons/CCM-secret.yaml
secret/hcloud created

$ kubectl apply -f https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm-networks.yaml
serviceaccount/cloud-controller-manager created
clusterrolebinding.rbac.authorization.k8s.io/system:cloud-controller-manager created
Warning: spec.template.metadata.annotations[scheduler.alpha.kubernetes.io/critical-pod]: non-functional in v1.16+; use the "priorityClassName" field instead
deployment.apps/hcloud-cloud-controller-manager created

$ kubectl apply -f hcloud-add-ons/CSI-secret.yaml 
secret/hcloud-csi created
davar@carbon:~/Documents/0-0-0-0-GoStudent/0-GITHUB-tf-ansible-k8s/TEST-repo/Production/kubespray/terraform-hetzner-kubeadm$ kubectl apply -f https://raw.githubusercontent.com/hetznercloud/csi-driver/v1.5.3/deploy/kubernetes/hcloud-csi.yml
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
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS      AGE   IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   pod/calico-kube-controllers-684bcfdc59-9tfjp           1/1     Running   1 (58m ago)   58m   10.0.1.1       node4    <none>           <none>
kube-system   pod/calico-node-6vgcl                                  1/1     Running   0             59m   10.0.1.3       node1    <none>           <none>
kube-system   pod/calico-node-q4752                                  1/1     Running   0             59m   10.0.1.1       node4    <none>           <none>
kube-system   pod/calico-node-qzsfl                                  1/1     Running   0             59m   10.0.1.2       node5    <none>           <none>
kube-system   pod/calico-node-t7g7j                                  1/1     Running   0             59m   10.0.1.4       node2    <none>           <none>
kube-system   pod/calico-node-v5trf                                  1/1     Running   0             59m   10.0.1.5       node3    <none>           <none>
kube-system   pod/coredns-8474476ff8-fqxhd                           1/1     Running   0             58m   10.233.92.1    node3    <none>           <none>
kube-system   pod/coredns-8474476ff8-nzs8b                           1/1     Running   0             58m   10.233.90.1    node1    <none>           <none>
kube-system   pod/dns-autoscaler-5ffdc7f89d-9fwvn                    1/1     Running   0             58m   10.233.96.1    node2    <none>           <none>
kube-system   pod/hcloud-cloud-controller-manager-666d7bbfcc-qb4f4   1/1     Running   0             13m   10.0.1.1       node4    <none>           <none>
kube-system   pod/hcloud-csi-controller-0                            0/5     Pending   0             49s   <none>         <none>   <none>           <none>
kube-system   pod/hcloud-csi-node-4grl7                              3/3     Running   0             48s   10.233.105.3   node4    <none>           <none>
kube-system   pod/hcloud-csi-node-8czx7                              3/3     Running   0             49s   10.233.92.4    node3    <none>           <none>
kube-system   pod/hcloud-csi-node-9xm7l                              3/3     Running   0             48s   10.233.70.4    node5    <none>           <none>
kube-system   pod/hcloud-csi-node-h9b6g                              3/3     Running   0             49s   10.233.96.4    node2    <none>           <none>
kube-system   pod/hcloud-csi-node-hf7cv                              3/3     Running   0             48s   10.233.90.4    node1    <none>           <none>
kube-system   pod/kube-apiserver-node1                               1/1     Running   0             61m   10.0.1.3       node1    <none>           <none>
kube-system   pod/kube-apiserver-node2                               1/1     Running   0             60m   10.0.1.4       node2    <none>           <none>
kube-system   pod/kube-apiserver-node3                               1/1     Running   0             60m   10.0.1.5       node3    <none>           <none>
kube-system   pod/kube-controller-manager-node1                      1/1     Running   1             61m   10.0.1.3       node1    <none>           <none>
kube-system   pod/kube-controller-manager-node2                      1/1     Running   1             60m   10.0.1.4       node2    <none>           <none>
kube-system   pod/kube-controller-manager-node3                      1/1     Running   1             60m   10.0.1.5       node3    <none>           <none>
kube-system   pod/kube-proxy-88r6n                                   1/1     Running   0             59m   10.0.1.5       node3    <none>           <none>
kube-system   pod/kube-proxy-hnk5p                                   1/1     Running   0             59m   10.0.1.3       node1    <none>           <none>
kube-system   pod/kube-proxy-ql2qc                                   1/1     Running   0             59m   10.0.1.1       node4    <none>           <none>
kube-system   pod/kube-proxy-tx4hj                                   1/1     Running   0             59m   10.0.1.4       node2    <none>           <none>
kube-system   pod/kube-proxy-wv8rk                                   1/1     Running   0             59m   10.0.1.2       node5    <none>           <none>
kube-system   pod/kube-scheduler-node1                               1/1     Running   1             61m   10.0.1.3       node1    <none>           <none>
kube-system   pod/kube-scheduler-node2                               1/1     Running   1             60m   10.0.1.4       node2    <none>           <none>
kube-system   pod/kube-scheduler-node3                               1/1     Running   1             60m   10.0.1.5       node3    <none>           <none>
kube-system   pod/nginx-proxy-node4                                  1/1     Running   0             59m   10.0.1.1       node4    <none>           <none>
kube-system   pod/nginx-proxy-node5                                  1/1     Running   0             59m   10.0.1.2       node5    <none>           <none>
kube-system   pod/nodelocaldns-7q6x5                                 1/1     Running   0             58m   10.0.1.3       node1    <none>           <none>
kube-system   pod/nodelocaldns-fcsdk                                 1/1     Running   0             58m   10.0.1.5       node3    <none>           <none>
kube-system   pod/nodelocaldns-hchlc                                 1/1     Running   0             58m   10.0.1.1       node4    <none>           <none>
kube-system   pod/nodelocaldns-j4ztx                                 1/1     Running   0             58m   10.0.1.2       node5    <none>           <none>
kube-system   pod/nodelocaldns-sz84l                                 1/1     Running   0             58m   10.0.1.4       node2    <none>           <none>

NAMESPACE     NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
default       service/kubernetes                      ClusterIP   10.233.0.1      <none>        443/TCP                  61m   <none>
kube-system   service/coredns                         ClusterIP   10.233.0.3      <none>        53/UDP,53/TCP,9153/TCP   58m   k8s-app=kube-dns
kube-system   service/hcloud-csi-controller-metrics   ClusterIP   10.233.63.230   <none>        9189/TCP                 48s   app=hcloud-csi-controller
kube-system   service/hcloud-csi-node-metrics         ClusterIP   10.233.17.65    <none>        9189/TCP                 48s   app=hcloud-csi

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS                                                   IMAGES                                                                                                                     SELECTOR
kube-system   daemonset.apps/calico-node       5         5         5       5            5           kubernetes.io/os=linux   59m   calico-node                                                  quay.io/calico/node:v3.20.2                                                                                                k8s-app=calico-node
kube-system   daemonset.apps/hcloud-csi-node   5         5         5       5            5           <none>                   49s   csi-node-driver-registrar,hcloud-csi-driver,liveness-probe   quay.io/k8scsi/csi-node-driver-registrar:v1.3.0,hetznercloud/hcloud-csi-driver:1.5.3,quay.io/k8scsi/livenessprobe:v1.1.0   app=hcloud-csi
kube-system   daemonset.apps/kube-proxy        5         5         5       5            5           kubernetes.io/os=linux   61m   kube-proxy                                                   k8s.gcr.io/kube-proxy:v1.22.3                                                                                              k8s-app=kube-proxy
kube-system   daemonset.apps/nodelocaldns      5         5         5       5            5           kubernetes.io/os=linux   58m   node-cache                                                   k8s.gcr.io/dns/k8s-dns-node-cache:1.17.1                                                                                   k8s-app=nodelocaldns

NAMESPACE     NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                        IMAGES                                                       SELECTOR
kube-system   deployment.apps/calico-kube-controllers           1/1     1            1           58m   calico-kube-controllers           quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers
kube-system   deployment.apps/coredns                           2/2     2            2           58m   coredns                           k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns
kube-system   deployment.apps/dns-autoscaler                    1/1     1            1           58m   autoscaler                        k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler
kube-system   deployment.apps/hcloud-cloud-controller-manager   1/1     1            1           13m   hcloud-cloud-controller-manager   hetznercloud/hcloud-cloud-controller-manager:v1.12.1         app=hcloud-cloud-controller-manager

NAMESPACE     NAME                                                         DESIRED   CURRENT   READY   AGE   CONTAINERS                        IMAGES                                                       SELECTOR
kube-system   replicaset.apps/calico-kube-controllers-684bcfdc59           1         1         1       58m   calico-kube-controllers           quay.io/calico/kube-controllers:v3.20.2                      k8s-app=calico-kube-controllers,pod-template-hash=684bcfdc59
kube-system   replicaset.apps/coredns-8474476ff8                           2         2         2       58m   coredns                           k8s.gcr.io/coredns/coredns:v1.8.0                            k8s-app=kube-dns,pod-template-hash=8474476ff8
kube-system   replicaset.apps/dns-autoscaler-5ffdc7f89d                    1         1         1       58m   autoscaler                        k8s.gcr.io/cpa/cluster-proportional-autoscaler-amd64:1.8.5   k8s-app=dns-autoscaler,pod-template-hash=5ffdc7f89d
kube-system   replicaset.apps/hcloud-cloud-controller-manager-666d7bbfcc   1         1         1       13m   hcloud-cloud-controller-manager   hetznercloud/hcloud-cloud-controller-manager:v1.12.1         app=hcloud-cloud-controller-manager,pod-template-hash=666d7bbfcc

NAMESPACE     NAME                                     READY   AGE   CONTAINERS                                                                  IMAGES
kube-system   statefulset.apps/hcloud-csi-controller   0/1     50s   csi-attacher,csi-resizer,csi-provisioner,hcloud-csi-driver,liveness-probe   quay.io/k8scsi/csi-attacher:v2.2.0,quay.io/k8scsi/csi-resizer:v0.3.0,quay.io/k8scsi/csi-provisioner:v1.6.0,hetznercloud/hcloud-csi-driver:1.5.3,quay.io/k8scsi/livenessprobe:v1.1.0
davar@carbon:~/Documents/0-0-0-0-GoStudent/0-GITHUB-tf-ansible-k8s/TEST-repo/Production/kubespray/terraform-hetzner-kubeadm$ 

### Test hcloud LB & PVC

$ kubectl apply -f hello/hello-default.yaml 
deployment.apps/hello-kubernetes created
service/hello-kubernetes created
persistentvolumeclaim/csi-pvc created
```
