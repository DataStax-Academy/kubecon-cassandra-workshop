
![OK](https://github.com/DataStax-Academy/kubecon-cassandra-workshop/blob/master/3-materials/images/00-screenplay.png?raw=true)

# Requirements and prerequisites

To practice kubernetes we need kubernetes, but it's a complex distributed system. For the learning purposes we use a special tool called `kind` - [Kubernetes IN Docker](https://github.com/kubernetes-sigs/kind). 

*"kind is a tool for running local Kubernetes clusters using Docker container "nodes". Kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI. Please refer to [Reference Documentation](https://kind.sigs.k8s.io/docs/user/quick-start/) for more detailed instructions."*

To make workshop as simple and useful as possible for you, we prepared a special cloud training instance you should have received already. It's a linux-based virtual instance running in AWS with all the prerequisites installed:

* Docker (Docker is an open-source project that automates the deployment of software applications inside containers by providing an additional layer of abstraction and automation of OS-level virtualization on Linux.)
* Docker Compose (Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure the application's services and performs the creation and start-up process of all the containers with a single command. The docker-compose CLI utility allows users to run commands on multiple containers at once, for example, building images, scaling containers, running containers that were stopped, and more.)
* Kubernetes command-line tool, Kubectl (The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.)
* `watch` (During the workshop some bootstrap can last for a few minutes and the utilty `watch` helps to see the evolutions.)
* Kind (`kind` is a tool for running local Kubernetes clusters using Docker container “nodes”.)

**Let's check that the installation is successful.** We will mark with a blue book the command (📘) and a green book (📗) to show expected result.

📘 **Command to execute**
```bash
# Check existing KinD clusters
kind get clusters
```

📗 **Expected output**
```
kind-cassandra
```

📘 **Execute the following command to list nodes in k8s cluster**
```bash
kubectl get nodes
```
📗 **Expected output**
```
NAME                           STATUS   ROLES    AGE    VERSION
kind-cassandra-control-plane   Ready    master   2m4s   v1.17.0
kind-cassandra-worker          Ready    <none>   86s    v1.17.0
kind-cassandra-worker2         Ready    <none>   88s    v1.17.0
kind-cassandra-worker3         Ready    <none>   88s    v1.17.0
kind-cassandra-worker4         Ready    <none>   88s    v1.17.0
kind-cassandra-worker5         Ready    <none>   88s    v1.17.0
```
As you see, we have only one 'physical' node but 6-nodes Kubernetes cluster, 1 master and 5 workers.

## Create namespace

Cass-operator is built to watch over pods running Casandra or DSE in a Kubernetes namespace. We need to create a namespace for the cluster. For the rest of this guide, we will be using the namespace `cass-operator`. You can pick any name you like but you would have to change the commands accordingly. We recommend to stick to the `cass-operator` namespace.

📘 **Execute the following command to create the namespace**
```bash
kubectl create ns cass-operator
```

📗 **Expected output**
```
namespace/cass-operator created
```

## Create storageClass

Kubernetes uses the `StorageClass` resource as an abstraction layer between pods needing persistent storage and the storage resources that a specific Kubernetes cluster can provide. We recommend using the fastest type of networked storage available. Let's create one for your environment.

📘 **Execute the following command to list existing storageClass**
```bash
kubectl get storageclass
```

📗 **Expected output**
```bash
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11m
```

📘 **Execute the following command describe `default` storageClass**
```bash
kubectl describe storageclass standard
```

📗 **Expected output** (may vary)
```
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

We are are interested in a few fields to intialize the `yaml` to create a storageClass. You can find more information in the [Reference Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/) and the [dedicated page](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/#changing-the-default-storageclass) to edit those.

One thing thing to note here is `volumeBindingMode: WaitForFirstConsumer`. The default value is `Immediate` and should not be used. It can prevent Cassandra pods from being scheduled on a worker node. If a pod fails to run and its status reports a message like, had volume node affinity conflict, then check the `volumeBindingMode` of the `StorageClass` being used

This file should be adapted tor reflect your *Kubernetes* environment. We have several samples in the repository for [kind](02-storageclass-kind.yaml), [Minikube](03-storageclass-minikube.yaml) and [GKE](04-storageclass-gke.yaml).

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: server-storage
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

📘 **Execute the following command to create a new `storageClass` named `server-storage`**
For the rest of this guide, we would assume you have defined a `StorageClass` and named it `server-storage`. 
```bash
kubectl -n cass-operator apply -f ./0-setup-your-cluster-datastax/02-storageclass-kind.yaml
```

📘 **7e. Execute the following command to list the storageClass**
```
kubectl -n cass-operator get storageClass
```
📗 **Expected output**
```
NAME                       PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
server-storage (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  25m
standard (default)         rancher.io/local-path   Delete          WaitForFirstConsumer   false                  88m
```

## Congratulations, your are done with the prerequisites.

Please proceed to [the next step](../1-cassandra/README.MD)!

![OK](https://github.com/DataStax-Academy/kubecon-cassandra-workshop/blob/master/3-materials/images/welldone.jpg?raw=true)

