# Talos + Cozystack

This document descripbes how to install Talos and then Cozystack on top of it.

## Talos

Reference:

- [Install talos using ISO](https://cozystack.io/docs/talos/installation/iso/)
- [Talos-Bootstrap](https://cozystack.io/docs/talos/configuration/talos-bootstrap/)

Talos is a little bit special in that it has a readonly filesystem, it's immutable.
That means it needs to be deployed in a little bit special way.

You first boot you nodes using a Talos boot ISO into "maintenence mode", the version of this is not really important but I recommend you download
and boot the ISO (metal-amd64.iso) found here: [https://github.com/cozystack/cozystack/releases](https://github.com/cozystack/cozystack/releases)

When Talos is running in maintenence mode, nothing is actually written to the system disk yet.

Talos is deployed using the talos-boostrap script that can be found here:
[https://github.com/cozystack/talos-bootstrap/](https://github.com/cozystack/talos-bootstrap/)

Talos-Bootstrap uses a Talos installer image that contains kernel settings and Talos system extensions we need to deploy
CozyStack on top of Talos.

In order to run the Talos-Bootstrap these dependencies needs to be installed:

- kubectl (not required by Talos-Bootstrap but for connection to Kubernetes API)
- talosctl (>=1.6.0)
- dialog
- nmap

Then run this script to download and make talos-bootstrap executable:

```shell
curl -LO https://github.com/cozystack/talos-bootstrap/raw/master/talos-bootstrap
chmod +x ./talos-bootstrap
sudo mv ./talos-bootstrap /usr/local/bin/talos-bootstrap
```

Now you need to prepare a folder for your cluster config:

```shell
mkdir cluster
cd cluster
```

Now you need to create a two files that are used by Talos-Bootstrap to setup your cluster.

patch.yaml

```yaml
machine:
  kubelet:
    nodeIP:
      validSubnets:
        - 192.168.100.0/24
    extraConfig:
      maxPods: 512
  kernel:
    modules:
      - name: openvswitch
      - name: drbd
        parameters:
          - usermode_helper=disabled
      - name: zfs
      - name: spl
  install:
    image: ghcr.io/cozystack/cozystack/talos:v1.9.5
  files:
    - content: |
        [plugins]
          [plugins."io.containerd.grpc.v1.cri"]
            device_ownership_from_security_context = true
          [plugins."io.containerd.cri.v1.runtime"]
            device_ownership_from_security_context = true
      path: /etc/cri/conf.d/20-customization.part
      op: create

cluster:
  network:
    cni:
      name: none
    dnsDomain: cozy.local
    podSubnets:
      - 10.244.0.0/16
    serviceSubnets:
      - 10.96.0.0/16
```

In this file you should edit the validSubnets to the subnet your nodes will be in.
Also set the image version to the version of Talos you wish to install...

![Important] The install image is a specially created one for use with CozyStack, this will deploy some extra kernel settings as well as
some Talos system extensions. Ensure you use the correct Talos Image for the version of CosyStack you want to deploy.

![Important] Dont change the dnsDomain, CozyStack requires it to be cozy.local.

patch-controlplane.yaml

```yaml
machine:
  nodeLabels:
    node.kubernetes.io/exclude-from-external-load-balancers:
      $patch: delete
cluster:
  allowSchedulingOnControlPlanes: true
  controllerManager:
    extraArgs:
      bind-address: 0.0.0.0
  scheduler:
    extraArgs:
      bind-address: 0.0.0.0
  apiServer:
    certSANs:
      - 127.0.0.1
  proxy:
    disabled: true
  discovery:
    enabled: false
  etcd:
    advertisedSubnets:
      - 192.168.100.0/24
```

In this file you should edit the advertisedSubnets to the subnet your nodes will be in.

Before continuing to deploy Talos you need to have your nodes up and running, booted into Talos "Maintenence mode".
You also should have configured hostname, dns servers, gateway and IP-address. I used DHCP for this and pinned my nodes
to the IP they should have.

Now you are ready to run Talos-Bootstrap.

```shell
talos-bootstrap install
```

![Important] If your client is not on the same subnet as the nodes, you need to point out the IP you wish to bootstrap.

```shell
talos-bootstrap install -n [Node IP]
```

Talos-Bootstrap is going to ask you for some settings:
[![screencast](https://raw.githubusercontent.com/cozystack/talos-bootstrap/2cc7b82065747e373cd914c87c8cd5c6582c5c6c/images/627123.gif)](https://asciinema.org/a/gwK85Tdr577GsxjXWo7otPFjv)

If you have added all values in advance either using the "maintenence mode" network setting or using DHCP most settings should aready be in place
but there are a couple of settings that could be good to know about:

- Cluster Name: The name you want your Kubernetes cluster to have.
- The disk you wish to install Talos to.
- Virtual IP: An IP-address all nodes uses for Kubernetes API, ensure you use the same VIP for all nodes.
- Node role: Control plane or Worker, in my case all 3 nodes are control planes, if you have more servers ensure you've got 3 control planes, the rest can be workers.
- Bootstrap ETCD, this should only be done on the first node.

When Talos-Bootstrap has finished you should have some extra files has been created in you cluster folder.

Run this to load kubeconfig inte an anvironment variable:

```shell
export KUBECONFIG=$PWD/kubeconfig
```

You can now test if Talos Kubernetes is running:

```shell
kubectl get nodes
```

If you haven´t botstrapped Talos onto all your nodes yet, go ahead and do that:

```shell
talos-bootstrap install -n [Node IP]
```

A good idea is to install [Mirantis Lens Personal edition](https://k8slens.dev/) that will let you poke around and see the complete cluster configuration.
When you have installed lens, open it and open the kubeconfig file created by talos-bootstrap.

## Cozystack

Reference:

- [Install CozyStack](https://cozystack.io/docs/get-started/)

Now that your Talos Kubernetes cluster is up and running, it's time to deploy CozyStack on it.

For this you need to create a ConfigMap yaml-file:

cozystack-config.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cozystack
  namespace: cozy-system
data:
  bundle-name: "paas-full"
  ipv4-pod-cidr: "10.244.0.0/16"
  ipv4-pod-gateway: "10.244.0.1"
  ipv4-svc-cidr: "10.96.0.0/16"
  ipv4-join-cidr: "100.64.0.0/16"
  root-host: example.org
  api-server-endpoint: https://192.168.100.10:6443
```

Edit the api-server-endpoint to your VIP address you created for the cluster.
Also, set a value for root-host, this used as the main domain for all services created under Cozystack, such as the dashboard, Grafana, Keycloak, etc.

Now it's time to deploy Cozystack but ensure the version is correct:

```shell
# Create cosy-system namespace.
kubectl create ns cozy-system

# Apply the CozyStack config to the cluster.
kubectl apply -f cozystack-config.yaml

# Apply CozyStack to the cluster - Change the version to the CozyStack version you wish to run.
kubectl apply -f https://github.com/cozystack/cozystack/releases/download/v0.29.1/cozystack-installer.yaml
```

This is going to take a while before all nodes are all up an running, you can use Lens to see when all pods are up and running.
If you dont use Lens you can track the logs of cozystack installer:

```shell
kubectl logs -n cozy-system deploy/cozystack -f
```

Wait for a while and check the status of the installation:

```shell
kubectl get hr -A
```

When the cozystack is installed and everything seems to be working you should configure storage:

1. Setup alias to access LINSTOR:

```shell
alias linstor='kubectl exec -n cozy-linstor deploy/linstor-controller -- linstor'
```

2. List your linstor nodes:

```shell
linstor node list
```

It should look like this:

```raw
+-------------------------------------------------------+
| Node | NodeType  | Addresses                 | State  |
|=======================================================|
| srv1 | SATELLITE | 192.168.100.11:3367 (SSL) | Online |
| srv2 | SATELLITE | 192.168.100.12:3367 (SSL) | Online |
| srv3 | SATELLITE | 192.168.100.13:3367 (SSL) | Online |
+-------------------------------------------------------+
```

3. List you empty disk devices:

```shell
linstor physical-storage list
```

It should look like this:

```raw
+--------------------------------------------+
| Size         | Rotational | Nodes          |
|============================================|
| 107374182400 | True       | srv3[/dev/sdb] |
|              |            | srv1[/dev/sdb] |
|              |            | srv2[/dev/sdb] |
+--------------------------------------------+
```

4. Create storage pools on your nodes (zfs) (change hostname and device name accordingly):

```shell
linstor ps cdp zfs srv1 /dev/sdb --pool-name data --storage-pool data
linstor ps cdp zfs srv2 /dev/sdb --pool-name data --storage-pool data
linstor ps cdp zfs srv3 /dev/sdb --pool-name data --storage-pool data
```

5. list the created storage pools:

```shell
linstor sp l
```

It should look like this:

```raw
+-------------------------------------------------------------------------------------------------------------------------------------+
| StoragePool          | Node | Driver   | PoolName | FreeCapacity | TotalCapacity | CanSnapshots | State | SharedName                |
|=====================================================================================================================================|
| DfltDisklessStorPool | srv1 | DISKLESS |          |              |               | False        | Ok    | srv1;DfltDisklessStorPool |
| DfltDisklessStorPool | srv2 | DISKLESS |          |              |               | False        | Ok    | srv2;DfltDisklessStorPool |
| DfltDisklessStorPool | srv3 | DISKLESS |          |              |               | False        | Ok    | srv3;DfltDisklessStorPool |
| data                 | srv1 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         | Ok    | srv1;data                 |
| data                 | srv2 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         | Ok    | srv2;data                 |
| data                 | srv3 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         | Ok    | srv3;data                 |
+-------------------------------------------------------------------------------------------------------------------------------------+
```

TODO: Continue here, time to hook up flux!
