# Running Kubernetes Cluster In Docker Containers Using KIND - Part 1
Many Companies are using containers to run their application in production, defining boundaries between applications, and enabling the control on app level - you can control
the CPU and memory amount to give for each container. but running container in production requires some level of coordination - or speaking technically, we need an orchestrator that control and manage your containers,
for example restarting an unhealthy container, scale the number of the instances based on the instance's resource consumption - CPU, RAM, ...

Kubernetes is the de-facto Container Orchestration Platform - there is also Docker Swarm from the Docker Inc, Mesos, which is - designed for the enterprise, 
it can run different workloads - stateless apps, stateful apps, cron-jobs, jobs, and much more...

In This Lab, we will use KIND to install and run Kubernetes using Docker Containers - This is why we call it KIND, Kubernetes In Docker.
I choose Kind, because it is a lightweight tool, unlike minikube which requires installing a hypervisor - VirtualBox, or VMWare - and then launching
a single node cluster inside a VM, and also it does not support multi-node cluster.
> kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI - Official Documentation

### Pre-Requisites:
* Docker 
* go - you can download it from [here](https://go.dev/dl/)

## Installing Kind:
Once you have installed the pre-requisites, you can now use the go binary to download and install kind as follow:

```bash
go install sigs.k8s.io/kind@v0.17.0
```

After installing kind CLI, we can try to create a cluster, and name it kind-cluster by passing --name argument - use the following command:
```bash
kind create cluster --name kind-cluster
```
below command output:
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
Now, we have created a cluster with a single node - single docker container. This container will host all the kubernetes components - control plane, scheduler, ...
kind uses Kubeadm to boot each container node - [kubeadm](https://github.com/kubernetes/kubeadm) is a tool built to provide best-practice "fast paths" for creating Kubernetes clusters.

You can see the cluster nodes using docker CLI:
```bash
docker container ls
```
the output will show that, you have one container running, which is your single node:
```bash
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                       NAMES
cf0c5d092282   kindest/node:v1.25.3   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   127.0.0.1:54180->6443/tcp   kind-cluster-control-plane
```

### Interacting with your Cluster:
To interact with your cluster, you need to have kubectl, for communicating with a Kubernetes cluster's control plane, using the Kubernetes API.
you can install kubectl using this [link](https://kubernetes.io/docs/tasks/tools/).

Once installed, we can try to list all pods in our cluster -- in all namespaces:
```bash
kubectl get pods --all-namespaces
```
the single cluster containers many pods - these pods are called system pods
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
as you can see from the output, all these pods - except pods local-path-provisioner-684f458cdd-jdc7l - are control plane's components.
> The control plane's components make global decisions about the cluster (for example, scheduling), as well as detecting and responding 
to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

let's delete the cluster to move to the next part of the lab
```bash
kind delete clusters kind-cluster
```

Until now, we have created a single cluster. but usually to simulate production, we need to have a cluster with multiple nodes, not only one node.
for that, kind enable us to control the cluster configuration via a yaml file to configure the cluster.
The configuration yaml file, follows the K8s conventions. an example of configuring kind cluster is as follows:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-cluster
```
here we are creating a cluster with name `kind-cluster` - its equivalent for the previous command where we create our first kind cluster.

Now, we will try to create a cluster with 5 instances - 2 as control plane, and 3 nodes as worker nodes. we can do that by adding `nodes` object to the configuration YAML file,

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

after defining the nodes, we can apply this configuration while creating the cluster using the following command:
```bash
kind create cluster --config multi-node-cluster.yaml

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

let's try to list the cluster node using kubectl:
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

As you can conclude, Kind is a very powerful tool that, enables you running kubernetes locally, and support multi-node topology.
In the next Lab, we will try to deploy some applications using k8s resources - deployment, service, LB, ... - and then exposing them to the outside - outside the cluster.