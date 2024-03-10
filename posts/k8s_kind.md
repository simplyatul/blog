# How to Set up a Local K8s Cluster Using KinD

## Use Case
The current cloud era is dominated heavily by Kubernetes (k8s). While the DevOps team manages App lifecycle (deploy, scale, upgrade, downgrade, etc.), the App developer needs to build a similar environment locally. So before shipping the App, developer is required to ensure DevOps lifecycle is not impacted by any of the changes s/he makes.  

So it is very much necessary for a Developer to have a local k8s cluster for testing. In short, this blog post demonstrates the following steps.

```
Step #1: Busniness application (App) development
Step #2: Containerize the App 
Step #3: Push image onto your local cluster
Step #4: Deploy on local k8s cluster
Step #5: Verify the App functionality
Step #6: Back to Step #1 to bug fix or enhance the App
```
## Assumptions
- You have [docker](https://www.docker.com/), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) and [KinD](https://kind.sigs.k8s.io/) installed on your host.
- You have go version ```go1.20.3``` installed on the host. However, older version should work too.
- This complete setup is build on ```Ubuntu 22.04.2``` host. If you are using another OS/platform, then you might need to modify the commands accordingly.

## Prerequisite setup
Clone the repo
```Shell
[~]# git clone https://github.com/simplyatul/godevsetup.git
[~]# cd godevsetup
```

Now using KinD, create local k8s cluster with a control plane node and two worker nodes.

```Shell
[~/godevsetup]# kind create cluster --config ./kind-local-k8s-cluster-baremin.yaml --name local-dev
Creating cluster "local-dev" ...
 ✓ Ensuring node image (kindest/node:v1.26.3) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-local-dev"
You can now use your cluster with:

kubectl cluster-info --context kind-local-dev

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

You can see that the control plane node and worker nodes are installed and running as separate containers.

```Shell
[~/godevsetup]# docker ps -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS          PORTS                       NAMES
9d2f27169e21   kindest/node:v1.26.3   "/usr/local/bin/entr…"   About a minute ago   Up 47 seconds                               local-dev-worker2
2b198d465d81   kindest/node:v1.26.3   "/usr/local/bin/entr…"   About a minute ago   Up 47 seconds                               local-dev-worker
96f3a760a02a   kindest/node:v1.26.3   "/usr/local/bin/entr…"   About a minute ago   Up 47 seconds   127.0.0.1:33519->6443/tcp   local-dev-control-plane
```

## Step 1 - App development
I have written a simple App in Go. You can do this in any language of your choice. The App exposes a ReST API on port 8080 and returns the hostname. Check the GitHub [code](https://github.com/simplyatul/godevsetup/blob/main/main.go). You can run the App locally using go run main.go and then hit the following command from another terminal.

```Shell
[~]# curl localhost:8080
Hostname: ubuntu-jammy
```

Once you complete the coding, commit it. Optionally, you can push it to the remote repo as well.

## Step 2 - Containerize the App

Build the docker image
```Shell
[~/godevsetup]# docker build -t godevsetup:latest .
```

Run it locally and check once
```Shell
[~/godevsetup]# docker run --rm --name godevsetup -p 8080:8080 godevsetup:latest

# From another terminal
[~/godevsetup]# curl localhost:8080
Hostname: 48143ca71c90
```
## Step 3 - Push image onto your local cluster
Here we will push the container image onto the cluster nodes created by KinD
```Shell
[~/godevsetup]# kind load docker-image godevsetup:latest --name local-dev
Image: "godevsetup:latest" with ID "sha256:aa7dbb2842050be8c1635ff117ef938e3cf7fb0c32d56d9a7fe16ca73fda0f62" not yet present on node "local-dev-worker2", loading...
Image: "godevsetup:latest" with ID "sha256:aa7dbb2842050be8c1635ff117ef938e3cf7fb0c32d56d9a7fe16ca73fda0f62" not yet present on node "local-dev-worker", loading...
Image: "godevsetup:latest" with ID "sha256:aa7dbb2842050be8c1635ff117ef938e3cf7fb0c32d56d9a7fe16ca73fda0f62" not yet present on node "local-dev-control-plane", loading...
```

You can inspect image is loaded
```Shell
[~/godevsetup]# docker exec local-dev-control-plane crictl images
IMAGE                                                     TAG                 IMAGE ID            SIZE
docker.io/library/godevsetup                              latest              aa7dbb2842050       869MB
registry.k8s.io/etcd                                      3.5.6-0             fce326961ae2d       103MB
registry.k8s.io/kube-apiserver                            v1.26.3             801fc1f38fa6c       80.4MB
registry.k8s.io/kube-controller-manager                   v1.26.3             cb77c367deebf       68.5MB
registry.k8s.io/pause                                     3.7                 221177c6082a8       311kB
docker.io/kindest/kindnetd:v20230330-48f316cd             <none>              a329ae3c2c52f       27.7MB
docker.io/kindest/local-path-helper:v20230330-48f316cd    <none>              37af659db0ba1       3.05MB
registry.k8s.io/coredns/coredns                           v1.9.3              5185b96f0becf       14.8MB
docker.io/kindest/local-path-provisioner:v0.0.23-kind.0   <none>              c408b2276bb76       18.7MB
registry.k8s.io/kube-proxy                                v1.26.3             eb3079d47a23a       67.2MB
registry.k8s.io/kube-scheduler                            v1.26.3             dec886d066492       57.8MB
```
## Step 4 - Deploy On Local K8s Cluster
I have created a Nodeport Service along with the deployment. Check the [yaml file](https://github.com/simplyatul/godevsetup/blob/main/godevsetup-nodeport.yaml). Create the Service and the Deployment.
```Shell
[~/godevsetup]# kubectl apply -f godevsetup-nodeport.yaml
deployment.apps/godevsetup created
service/godevsetup-nodeport created
```
You can check the Service and pods are created
```Shell
[~/godevsetup]# kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/godevsetup-78fbdfb985-4tmz2   1/1     Running   0          3m59s
pod/godevsetup-78fbdfb985-frl2d   1/1     Running   0          3m59s
pod/godevsetup-78fbdfb985-x77g4   1/1     Running   0          3m59s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/godevsetup-nodeport   NodePort    10.96.245.170   <none>        9090:30123/TCP   3m59s
service/kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          52m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/godevsetup   3/3     3            3           3m59s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/godevsetup-78fbdfb985   3         3         3       3m59s
```

You can also observe the App pods are created inside the worker nodes
```Shell
[~/godevsetup]# docker exec local-dev-worker crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                     ATTEMPT             POD ID              POD
a310b2cf0201d       aa7dbb2842050       5 minutes ago       Running             godevsetup               0                   66781797eb650       godevsetup-78fbdfb985-frl2d
fea54fa36f629       c408b2276bb76       54 minutes ago      Running             local-path-provisioner   0                   d03b938684afd       local-path-provisioner-75f5b54ffd-k7xfb
b11289f372caf       5185b96f0becf       54 minutes ago      Running             coredns                  0                   0b51dcb843021       coredns-787d4945fb-wqxst
ee14e1dc8ff83       5185b96f0becf       54 minutes ago      Running             coredns                  0                   e7a1e535bb183       coredns-787d4945fb-g7xnq
99a540c5658fb       a329ae3c2c52f       54 minutes ago      Running             kindnet-cni              0                   eb134c7507011       kindnet-gmzxc
aaebc183b9f6a       eb3079d47a23a       54 minutes ago      Running             kube-proxy               0                   a7b50875a469f       kube-proxy-6wj5v

[~/godevsetup]# docker exec local-dev-worker2 crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD
b2d32e5b552f7       aa7dbb2842050       5 minutes ago       Running             godevsetup          0                   872cff6cf3ea3       godevsetup-78fbdfb985-4tmz2
9ae704f8c8669       aa7dbb2842050       5 minutes ago       Running             godevsetup          0                   e186f3ccda549       godevsetup-78fbdfb985-x77g4
79013566e33e8       a329ae3c2c52f       54 minutes ago      Running             kindnet-cni         0                   959abab06b391       kindnet-tg2jl
0611eaced1aad       eb3079d47a23a       54 minutes ago      Running             kube-proxy          0                   49477d56a66c8       kube-proxy-96f5x
```

## Step 5 - Verify The Functionally
Since we have created a NodePort Service, we can access the service using Node’s IP.  

First, identify the Node IPs
```Shell
[~/godevsetup]# kubectl get nodes -o wide
NAME                      STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
local-dev-control-plane   Ready    control-plane   58m   v1.26.3   172.18.0.4    <none>        Ubuntu 22.04.2 LTS   5.15.0-72-generic   containerd://1.6.19-46-g941215f49
local-dev-worker          Ready    <none>          57m   v1.26.3   172.18.0.2    <none>        Ubuntu 22.04.2 LTS   5.15.0-72-generic   containerd://1.6.19-46-g941215f49
local-dev-worker2         Ready    <none>          57m   v1.26.3   172.18.0.3    <none>        Ubuntu 22.04.2 LTS   5.15.0-72-generic   containerd://1.6.19-46-g941215f49
```

The INTERNAL-IP is the IP of the worker node. Now access the API’s ReST endpoint using any of the Node’s IP and Port (Port is mentioned as ```nodePort: 30123``` in our [yaml file](https://github.com/simplyatul/godevsetup/blob/main/godevsetup-nodeport.yaml))
```Shell
[~/godevsetup]# curl 172.18.0.4:30123
Hostname: godevsetup-78fbdfb985-frl2d

[~/godevsetup]# curl 172.18.0.2:30123
Hostname: godevsetup-78fbdfb985-frl2d

[~/godevsetup]# curl 172.18.0.3:30123
Hostname: godevsetup-78fbdfb985-4tmz2
```

Cool.. We are done :slightly_smiling_face:. Now if you want to Enhance (or Bug-fix) your App, restart from Step 1. But before that, remove the Service and Deployment.
```Shell
[~/godevsetup]# kubectl delete -f godevsetup-nodeport.yaml 
deployment.apps "godevsetup" deleted
service "godevsetup-nodeport" deleted
```

## Sign Off…
This blog post covers how to set up a local k8s cluster using KinD. And how can we leverage it for the App deployment and testing.
