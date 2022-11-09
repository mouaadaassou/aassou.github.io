---
layout: post
author: Moaad Aassou
tags: [cloud, infrastructure]
title:  "Multi-Node Cluster with Kind"
---

# Running Kubernetes Cluster In Docker Containers Using KIND - Part 1

![KIND](/images/kind-logo.png)

Many Companies are using containers to run their applications in production, due to its capabilities to ship and run applications in a portable and a secure way, thanks for Linux Control-Group and namespaces.
Especially in the era of the microservices architecture, where you divide your big-monolith application into multiple applications, each application does one thing, and does it well.

But running containers in production requires some level of coordination, technically speaking, we need an orchestrator that automates and manages the lifecycle management of containers, like restarting an unhealthy container, scale the number of the instances based on the instance resource consumption (CPU usage, RAM consumption and so on and so forth).

Kubernetes is the de-facto Container Orchestration Platform designed for the enterprise, it can run different workloads, like stateless and/or stateful application, cron-jobs, jobs, and list goes on.

Kubernetes is a great orchestration tool, but it's really hard to install on your local machine, since it needs a lot of tools and software pieces.

There are many alternatives to install Kubernetes on your local machine like Minikube, KIND, K3s/K3d and Rancher.

Based on my own experience, Kind - which stands for Kubernetes In Docker - is the easiest way to run Kubernetes on your local machine.

Unlike Minikube, which requires installing a hypervisor like VirtualBox, or VMWare, and then launching
a single node cluster inside a VM, Kind is easy to use and uses Docker containers to work smoothly.

Minikube also doesn't support multi-node cluster, Kind does.

For that reason, we will use KIND in this lab to install and run Kubernetes using Docker Containers.

> kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI - Official Documentation

### Prerequisites:
* Docker
* golang - you can download it from [here](https://go.dev/dl/)

## Installing Kind:
Once you have installed the prerequisites, you can now use the <b>golang</b> binary to download and install kind as follows:

```bash
go install sigs.k8s.io/kind@v0.17.0
```

After installing <b>Kind CLI</b>, we can try to create a cluster, we will name it <i>Kind-cluster</i> by passing <b>--name</b> argument, use the following command:

```bash
kind create cluster --name kind-cluster
```

The output should be similar to:

```bash
Creating cluster "kind-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-cluster

Have a nice day! 👋
```
Now, we have created a cluster with a single node (Docker container). This node will host all the Kubernetes components like control plane, scheduler, etcd and API Server.

### Interacting with your Cluster:
To interact with your cluster, you need to have kubectl.Internally, kubectl communicates with the cluster control plane through Kubernetes API.

You can install kubectl using this [link](https://kubernetes.io/docs/tasks/tools/).

Once installed, we can try to list all pods in all namespaces within our cluster:
```bash
kubectl get pods --all-namespaces
```
As you can see below, the cluster contains many system pods:
```bash
NAMESPACE            NAME                                                 READY   STATUS    RESTARTS   AGE
kube-system          coredns-565d847f94-4bb7h                             1/1     Running   0          16m
kube-system          coredns-565d847f94-grgrz                             1/1     Running   0          16m
kube-system          etcd-kind-cluster-control-plane                      1/1     Running   0          16m
kube-system          kindnet-w9mnw                                        1/1     Running   0          16m
kube-system          kube-apiserver-kind-cluster-control-plane            1/1     Running   0          16m
kube-system          kube-controller-manager-kind-cluster-control-plane   1/1     Running   0          16m
kube-system          kube-proxy-9nk6l                                     1/1     Running   0          16m
kube-system          kube-scheduler-kind-cluster-control-plane            1/1     Running   0          16m
local-path-storage   local-path-provisioner-684f458cdd-jdc7l              1/1     Running   0          16m
```
As you can see from the output, all these pods except pods local-path-provisioner-684f458cdd-jdc7l are control plane components.

> The control plane components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment replicas field is unsatisfied).

Now, let's delete the single node cluster to move to the next part of the lab
```bash
kind delete clusters kind-cluster
```

Until now, we have created a single cluster, but usually to simulate production, we need to have a cluster with multiple nodes, not only one node.
For that, KIND enables us to control the cluster configuration via a YAML file to configure the cluster's nodes.

The YAML file configuration, follows the K8s conventions. an example of configuring kind cluster is as follows:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-cluster
```

Here we are creating a cluster with `kind-cluster` as name.

Now, we will try to create a cluster with 5 nodes, 2 nodes as control plane and 3 nodes as worker nodes.
We can do that by adding `nodes` object to the YAML configuration file,

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-cluster
nodes:
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```

After defining the nodes, we can apply this configuration while creating the cluster using the following command:
```bash
kind create cluster --config multi-node-cluster.YAML

Creating cluster "kind-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦 📦
 ✓ Configuring the external load balancer ⚖️
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining more control-plane nodes 🎮
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-cluster

Thanks for using kind! 😊
```

Let's try to list the cluster nodes using kubectl:
```bash
kubectl get nodes

NAME                          STATUS   ROLES           AGE     VERSION
kind-cluster-control-plane    Ready    control-plane   2m20s   v1.25.3
kind-cluster-control-plane2   Ready    control-plane   2m7s    v1.25.3
kind-cluster-worker           Ready    <none>          67s     v1.25.3
kind-cluster-worker2          Ready    <none>          67s     v1.25.3
kind-cluster-worker3          Ready    <none>          67s     v1.25.3
```
As you can see from the output, we have now 5 nodes with 2 as control-plan and 3 as worker nodes.

Now you have a ready to use cluster on which you can deploy your applications.

We can conclude that Kind is a very powerful tool, that enables us to run kubernetes locally either in single node mode or multi-node mode.

In the next Lab, we will try to use the multi-node cluster by deploying some applications using k8s resources like deployment, service and configMap.
