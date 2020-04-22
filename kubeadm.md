10.240.0.8   user-node

10.240.0.8 laptop
10.240.0.9 kubeadm-workernode1
10.240.0.234 kubeadm-workernode2
10.240.0.29 kubeadm-masternode
# Kubeadm

Kubeadm helps you setup/bootstrap a minimum viable/usable Kubernetes cluster that just works. Kubeadm also supports cluster expansion, upgrades, downgrade, and managing bootstrap tokens, which are extra features, if you are comparing it with minikube.

Note
>Kubeadm limits to setup/bootstrap a machine and not about provisioning machines. Likewise, installing various nice-to-have addons, like the Kubernetes Dashboard, monitoring solutions, and cloud-specific addons, is not in scope.
Instead, we expect higher-level and more tailored tooling to be built on top of kubeadm, and ideally, using kubeadm as the basis of all deployments will make it easier to create conformant clusters.

You can install and use kubeadm on various machines: your laptop, a set of cloud servers, a Raspberry Pi, and more. Whether you’re deploying into the cloud or on-premises, you can integrate kubeadm into provisioning systems such as Ansible or Terraform.

With Kubeadm we can perform the following functions:

* kubeadm init to bootstrap a Kubernetes control-plane node

* kubeadm join to bootstrap a Kubernetes worker node and join it to the cluster

* kubeadm upgrade to upgrade a Kubernetes cluster to a newer version

* kubeadm config if you initialized your cluster using kubeadm v1.7.x or lower, to configure your cluster for kubeadm upgrade
  
* kubeadm token to manage tokens for kubeadm join

* kubeadm reset to revert any changes made to this host by kubeadm init or kubeadm join

* kubeadm version to print the kubeadm version

* kubeadm alpha to preview a set of features made available for gathering feedback from the community

Here we will use kubeadm for `init`, `join`, `version`, and `reset`. and will see all of these one by one in detail.

## Setup / components / look and feel:

The Kubernetes cluster discussed in this guide is a virtual cloud cluster, created using virtual machines on AWS cloud, on a Amazon Linux 2(Free tier) AMI.

* Create a VPC with CIDR block 10.240.0.0/16 and create a subnet with sunetting with CIDR IPV4 10.240.0.0/24.

* Create a Internet Gateway and connect it to the VPC, setup the route table(0.0.0.0/0) to Internet gateway and finally asscociate the subnet.

* Launch an instance with Amazon Linux 2 AMI with size `t2.medium` within the created VPC and subnet(set as Control-plane or master node)

* Launch two instances with same AMI with `t2.micro` within the created VPC and subnet(set as worker nodes)

 So now we are ready with 3 nodes out of which we create a cluster using Kubeadm and make 1 Master node(Control-plane) and 2 Worker nodes.

* Let suppose the user working in laptop, and hence for that we will launch one more instance for user.


![image](https://user-images.githubusercontent.com/58899893/79992953-d08aa580-84d1-11ea-8b2d-e9fa74192f7b.png)

## Key points to be noted.

* CPU: Minimum 2 CPU for master node(Control plane); Worker nodes can live with single core CPUs.
 
* Infrastructure Network: A functional virtual/physical network with some usable IP addresses (can be public or private) . This can be on any cloud provider as well. You are free to use any network / ip scheme for yourself. We will be using CIDR `10.240.0.0/16` with `/24` subnetting.

* Pod network: A network IP range completely differnt from other infrastucture and service networks. But here we set it to `10.200.0.0/16`. `It is upto us to set it to default values or our own value`

 **Please note that kubeadm does not support kubenet, so we need to use one of the CNI add-ons - such as flannel. By default Flannel sets up a pod network 10.244.0.0/16, which means that we need to pass this pod network to kubeadm init (further below); or, modify the flannel configuration with the pod network of our own choice - before actually applying it blindly**

* Service network: A network IP range completely different from other two networks, and is used by the kubernetes services. This will be considered a completely virtual network. The default service network configured by kubeadm is `10.96.0.0/12`. But we will set it to `10.32.0.0/16`.

* Firewall: Disable Firewall (including removing the firewalld package), for smooth working of the cluster but it is not recommended in organizations.

* Disk Partitioning: No swap - must disable swap partition in order for the kubelet to work properly.

* SELinux: Disable SELinux for smooth working of the cluster.

Note
>SELinux(Security Enhanced Linux) refers to the security policy or security module that has been integrated in Linux since Linux kernel v 2.6.*
Security Enhanced Linux defines the access rights of every user, application, process and file present in the system. It then monitor the activity that requires access to certain files/directory, it may be a user or an applications request to access those. The request is sent to the security server in the kernel, where it checks for the security context of the request source and the requested data. If the context are compatible, the permission is granted. If it is denied, then a message is issued in /var/log/message with avc: denied. 


## Setup:

* I am working in my laptop within the same network as a normal user. and to do so I launch one more machine same as worker node and to setup cluster I will ssh to every node. Pls refer [passwordless-ssh-login](xxx) 

* Make a hostname entry 

 kubeadm-masternode

 kubeadm-workernode01

 kubeadm-workernode02

* Set hosts in hosts file for discovery

* Remove the firewall in all the nodes. 


 `yum remove firewalld -y`


* Disable SELINUX in all the nodes

`vi /etc/selinux/config`

uncomment SELINUX=disabled


### Install container runtime (Docker) 

On all nodes (master and worker), install Docker.

```
amazon-linx-extras install docker -y
```


```
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```


### Install kubeadm, kubelet and kubectl:
On every node, install:
* kubeadm: the command to actually setup / bootstrap the cluster.
* kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
* kubectl: the command line util to talk to your cluster.


All of the above three pieces of software are available from kubernetes's yum repository. And hence create the kebernetes repository.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```


```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

The above command installs additional packages, which are:

* cri-tools (command line utility to interact with container runtime, such as docker)
* kubernetes-cni (binary files to provision container networking. The files are installed in /opt/cni/bin)
* socat (relay for bidirectional data transfer between two independent data channels, e.g. files, pipe, device, socket, program, etc.)


```
systemctl enable kubelet && systemctl start kubelet
```

At this time kubeadm is only *installed* - not *run*. Note that kubelet is set to start. Kubelet will continuously try to start and will fail (crash-loop), because it will wait for kubeadm to tell it what to do. This crashloop is expected and normal. After you initialize your master (using kubeadm), the kubelet runs normally.


## Run kubeadm on kube to setup the cluster:

SSH to the kubeadm-masternode as root, and run `kubeadm init` command. 

What `kubeadm init` does is, it goes through a series of phases, the first one being **pre-flight checks** , and last one being **installing add-ons**. These phases are shown in square brackets in the beginning of each line of the output generated by the `kubeadm init` command. Most notably, these phases are:

* Pull container images for kubernetes *control plane* (etcd, API server, Controller Manager, Scheduler)
* Setup kublet as systemd service
* Generate self signed SSL certificates for all kubernetes components
* Generate necessary kubeconfig files for components of the *control plane*, and the admin user, to be able to access API server with credentials, certificates, etc
* Create and deploy **static pod** definitions of the *control plane* components directly through **kubelet**. You will see (later) that etcd, api-server, controller-manager and scheduler are deployed as plain pods, without being part of any replication-controller, deployment, or daemon-set. These pods are setup using *host* networking.
* Create **kubelet** configuration as *config-map* in the **kube-system** namespace - as soon as the control plane boots up and reports healthy
* Mark the master as **master**  by adding labels and taints
* Create bootstrap tokens and RBAC rules so various components are able to access each other and do certain things, like automatic approval of CSRs, etc
* Install CoreDNS addon by creating it as a *deployment* - using *pod* networking
* Install kube-proxy addon by creating it as a *daemon-set* - using *host* networking


```
kubeadm init \
  --pod-network-cidr "10.200.0.0/16" \
  --service-cidr "10.32.0.0/16"
```

**Note:** You can skip `--pod-network-cidr` and `--service-cidr` it will be set to default(--pod-network-cidr 10.244.0.0/16 and --service-cidr 10.96.0.0/12). And if you want to set to your requirement then you should provide explcitly, as we provided above.


After successful of kubeadm init you will get something like this
![image](https://user-images.githubusercontent.com/58899893/79993043-eef0a100-84d1-11ea-817b-c53674e0f030.png)  
 
To perform `kubead init` the user has to be root. Non root user it is not available.

Now you can check the `kubeadm version`

![image](https://user-images.githubusercontent.com/58899893/79993174-18113180-84d2-11ea-8c4e-5486852bfe82.png)

But any non root user can work on cluster, So here I am entering as a non root user `kubernetes-admin` (kubernetes-admin user created)
 
```
[root@kubeadm-masternode ~]# su - kubernetes-admin
```
 
run `kubectl get nodes`

As expected we are getting error.


Because the user `kubernetes-admin` doenot have the kube config file and also the ownership to run. 
The kube config file is with root user, and so we are getting the error.

So get the copy of kube config file to you and allow the ownership

``` 
[kubernetes-admin@kubeadm-masternode ~]$ mkdir -p $HOME/.kube

[kubernetes-admin@kubeadm-masternode ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

[kubernetes-admin@kubeadm-masternode ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Check if you can talk to the cluster/API server:

```
[kubernetes-admin@kubeadm-masternode ~]$ kubectl get componentstatuses

NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
[kubernetes-admin@kubeadm-masternode ~]$ 


[kubernetes-admin@kubeadm-masternode ~]$ kubectl get nodes
NAME                STATUS     ROLES    AGE   VERSION
kubeadm-masternode   NotReady   master   12m   v1.12.2
[kubernetes-admin@kubeadm-masternode ~]$ 
```

At least the cluster's components are ok!

And if you check the pods created by kubernetes services all will be fine except two as below.

![image](https://user-images.githubusercontent.com/58899893/79990312-881db880-84ce-11ea-9cd6-022a3b99f3a7.png) 

# Install pod network:

At this time, if you check cluster health, you will see **Master** as **NotReady**. It is because a pod network is not yet deployed on the cluster. You must install a pod network add-on so that your pods can communicate with each other. Use one of the network addons listed in the Networking section at [https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/). Flannel is the easiest. Others can be used too.

**Important:** The network must be deployed before any applications. Also, CoreDNS will not start up before a network is installed. kubeadm only supports Container Network Interface (CNI) based networks (and does not support kubenet).


## Install flannel CNI plugin/addon:
 
For flannel CNI plugin to install, you have to run a flannel config file which is usually a yaml file.

You will get the yaml file in my github repository

Create an empty kube-flannnel.yml file, paste the content. 

At line 128 in network block , change the cidr to your requirement (Ours is `10.200.0.0/16`) and save the file

Run the file to install flannel CNI plugin,so the pod network could come up. Flannel sets up as a *daemon-set* and uses *host* networking of the node.


`kubectl apply -f kube-flannel.yml`

![image](https://user-images.githubusercontent.com/58899893/79990187-5d336480-84ce-11ea-9ad4-1bd928c6f65e.png)


## Check cluster health:


```
[kubernetes-admin@kubeadm-masternode ~]$ kubectl get nodes
NAME                STATUS   ROLES     AGE   VERSION
kubeadm-masternode   Ready    master   55m   v1.12.2
[kubernetes-admin@kubeadm-masternode ~]$ 
```

And also all the pod service will be running as shown
![image](https://user-images.githubusercontent.com/58899893/79989988-20676d80-84ce-11ea-8a1d-f1b278a3b44c.png)

Add worker nodes to cluster. successful  `kubeadm init` gave instruction on how to add nodes as worker node to cluster. We just follow that.

SSH to the worker nodes and perform the `kubeadm join`

Once it is successful, you will get something like this.

![image](https://user-images.githubusercontent.com/58899893/79994700-eb5e1980-84d3-11ea-8feb-f68ca75c2fad.png)


Once you added worker nodes to cluster, if you check now `kubectl get nodes`

![image](https://user-images.githubusercontent.com/58899893/79992665-6ffb6880-84d1-11ea-91bd-82cc7333acd4.png)


Initially in the cluster only `kubernetes service` will be running as you see

![image](https://user-images.githubusercontent.com/58899893/79992833-a5a05180-84d1-11ea-96e2-fee8ff000173.png)
 

### WOW......SUPER!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
 
We are ready with our cluster.

#### A simple handson with the cluster.


* Create a deployment `web-server` with simple nginx image.

* Create a deployment `db-server` with simple redis image.

* Check the deployments.

`kubectl get deployments -o wide`
![image](https://user-images.githubusercontent.com/58899893/79992591-53f7c700-84d1-11ea-8fad-cb6f80d8a5e7.png)

* To access the application from client or any browser(end user) expose the deployment `web-server` as NodePort service at port 80

`kubectl expose deployment web-server --type=NodePort --port 80`

![image](https://user-images.githubusercontent.com/58899893/79993535-7b02c880-84d2-11ea-957f-d3af3bd8ec4d.png)

As you see in the image web-server service is exposed as NodePort with port `31523` ( Yours will be different)

* Get the full status of the cluster to know the deployments,services, and pods running

`kubectl get all -o wide`

![image](https://user-images.githubusercontent.com/58899893/79990096-442ab380-84ce-11ea-9e64-39ec9dc7e734.png)

As port is exposed with `31523` we can test whether application is running or not.

* Access the application in worker node 1
 
Any browser hit 'public ip of workernode1':31523

![image](https://user-images.githubusercontent.com/58899893/79993637-9968c400-84d2-11ea-9158-7006afe410bf.png)

 
* Access the application in worker node 2
 
 Any browser hit 'public ip of workernode2':31523

  
So Finally our K8s cluster will be like this

![image](https://user-images.githubusercontent.com/58899893/80005533-53672c80-84e1-11ea-9923-0064e9315755.png)



![the end](https://media-exp1.licdn.com/dms/image/C5612AQFfARfN7n97oQ/article-cover_image-shrink_600_2000/0?e=1592438400&v=beta&t=blDgnfUk7-GU-d8yuX8cBBEOxiC7d3-KghodTiBgJtM)


 


 













