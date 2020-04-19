#  Kubernetes (K8s)

## Introduction


 * Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
 * It groups containers that make up an application into logical units for easy management and discovery.
 * It was originally designed by Google, and is now maintained by the Cloud Native Computing Foundation.
 * The name Kubernetes originates from Greek, meaning helmsman or pilot of the ship.    
  **NOTE: I do referr kubernetes as K8s**    

 ```html
   A quick history : K8s technology was used by Google on their production environment 
   from many years by the name called "Borg". and it was written in C++.
   Once Docker came into market introducing Containerization technology, Google make this 
   Borg technology as open source for public use. Then a team of developers at Cloud Native Computing Foundation
   reimplemented this technology and written on Go lanuage and named it as "Kubernetes".  
   And released the first product release version v1.0 in 2015. 
 ```

## What can Kubernetes do for you?

 With modern web services, users expect applications to be available 24/7, and developers expect to deploy new versions of those applications several times a day.
 Containerization helps package software to serve these goals, enabling applications to be released and updated in an easy and fast way without downtime.
 Kubernetes helps you make sure those containerized applications run where and when you want, and helps them find the resources and tools they need to work.


 There are many ways to learn K8s, 
 * You can learn from the (official website)[https://kubernetes.io/docs/home/]
 * Here is the official Github repository link [https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)
 * Many online classroom courses avaialble.
 * And to get the basic idea of what K8s is just go through the youtube videos (which I preferred).

 ## So, Lets get Started!!!!!! 

  First you understand how the application deployment methods have been improvised day by day.

   ![image](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)
 
 * **Traditional deployment era:**  Early on, organizations ran applications on physical servers.There was no way to define resource boundaries for applications in a physical server, and this caused resource allocation issues. For example, if multiple applications run on a physical server, there can be instances where one application would take up most of the resources, and as a result, the other applications would underperform. A solution for this would be to run each application on a different physical server. But this did not scale as resources were underutilized, and it was expensive for organizations to maintain many physical servers.

 * **Virtualized deployment era:** As a solution, virtualization was introduced. It allows you to run multiple Virtual Machines (VMs) on a single physical server’s CPU. Virtualization allows applications to be isolated between VMs and provides a level of security as the information of one application cannot be freely accessed by another application.

Virtualization allows better utilization of resources in a physical server and allows better scalability because an application can be added or updated easily, reduces hardware costs, and much more. With virtualization you can present a set of physical resources as a cluster of disposable virtual machines.

Each VM is a full machine running all the components, including its own operating system, on top of the virtualized hardware.

 * **Container deployment era:** Containers are similar to VMs, but they have relaxed isolation properties to share the Operating System (OS) among the applications. Therefore, containers are considered lightweight. Similar to a VM, a container has its own filesystem, CPU, memory, process space, and more. As they are decoupled from the underlying infrastructure, they are portable across clouds and OS distributions.

Containers have become popular because they provide extra benefits, such as:

   * Agile application creation and deployment: increased ease and efficiency of container image creation compared to VM image use.

   * Continuous development, integration, and deployment: provides for reliable and frequent container image build and deployment with quick and easy rollbacks (due to image immutability).
   
   * Dev and Ops separation of concerns: create application container images at build/release time rather than deployment time, thereby decoupling applications from infrastructure.
   
   * Observability not only surfaces OS-level information and metrics, but also application health and other signals.
   
   * Environmental consistency across development, testing, and production: Runs the same on a laptop as it does in the cloud.
   
   * Cloud and OS distribution portability: Runs on Ubuntu, RHEL, CoreOS, on-premises, on major public clouds, and anywhere else.
   
   * Application-centric management: Raises the level of abstraction from running an OS on virtual hardware to running an application on an OS using logical resources.
   
   * Loosely coupled, distributed, elastic, liberated micro-services: applications are broken into smaller, independent pieces and can be deployed and managed dynamically – not a monolithic stack running on one big single-purpose machine.
   
   * Resource isolation: predictable application performance.
   
   * Resource utilization: high efficiency and density.
 
 ## Why you need Kubernetes and what it can do??
 
  Containers are a good way to bundle and run your applications. In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. For example, if a container goes down, another container needs to start. Wouldn’t it be easier if this behavior was handled by a system?
  
  That’s how Kubernetes comes to the rescue! Kubernetes provides you with a framework to run distributed systems resiliently. It takes care of scaling and failover for your application, provides deployment patterns, and more. For example,  **Kubernetes can easily manage a canary deployment for your system.**

  Note 
  > Canary deployments are a pattern for rolling out releases to a subset of users or servers. The idea is to first deploy the change to a small subset of servers, test it, and then roll the change out to the rest of the servers.
  
  #### Kubernetes provides you with:
  
   * **Service discovery and load balancing :**
      Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.


   * **Storage orchestration :**
      Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
 

   * **Automated rollouts and rollbacks :**
     You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.


   * **Automatic bin packing :**
     You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
   
   * **Self-healing :**
     Kubernetes restarts containers that fail, replaces containers, kills containers that don’t respond to your user-defined health check, and doesn’t advertise them to clients until they are ready to serve. 
   
   * **Secret and configuration management :**
      Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration


 ## Kubernetes Components
 When you deploy Kubernetes, you get a cluster.

 A Kubernetes cluster consists of a set of worker machines, called nodes , that run containerized applications. Every cluster has at least one worker node.
 
 The worker node(s) host the Pods that are the components of the application workload. The control plane manages the worker nodes and the Pods in the cluster. In production environments, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes, providing fault-tolerance and high availability.

 This document outlines the various components you need to have a complete and working Kubernetes cluster.
 
 Here’s the diagram of a Kubernetes cluster with all the components tied together.
 
 ![image](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)
 
 ### Control Plane Components
  
 The Control Plane’s components make global decisions about the cluster (for example, scheduling), as well as detecting and responding to cluster events (for example, starting up a new pod when a deployment’s replicas field is unsatisfied).

 Control Plane components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all Control Plane components on the same machine, and do not run user containers on this machine. See Building High-Availability Clusters for an example multi-master-VM setup
  
 #### kube-apiserver
 
 The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

 The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.


 #### etcd 
 
 Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

 If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.

 You can find in-depth information about etcd in the official documentation.

 #### kube-scheduler
 
 Control plane component that watches for newly created Pods with no assigned node , and selects a node for them to run on.

 Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

 #### kube-controller-manager
 
Control Plane component that runs controller processes.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:
 
  * **Node Controller:** Responsible for noticing and responding when nodes go down.

  * **Replication Controller:** Responsible for maintaining the correct number of pods for every replication controller object in the system.

  * **Endpoints Controller:** Populates the Endpoints object (that is, joins Services & Pods).

  * **Service Account & Token Controllers:** Create default accounts and API access tokens for new namespaces.

  
 #### cloud-controller-manager 

 cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

 cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the ` --cloud-provider ` flag to ` external ` when starting the kube-controller-manager.

 cloud-controller-manager allows the cloud vendor’s code and the Kubernetes code to evolve independently of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

 The following controllers have cloud provider dependencies:

  * **Node Controller :** For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
  * **Route Controller:** For setting up routes in the underlying cloud infrastructure
  * **Service Controller:** For creating, updating and deleting cloud provider load balancers
  * **Volume Controller:** For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

 ## Node Components

 Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

 #### kubelet
 An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod .

 The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.
 
#### kube-proxy
 kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

 kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

 kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

 #### Container Runtime

 The container runtime is the software that is responsible for running containers.

 Kubernetes supports several container runtimes: Docker , containerd , CRI-O , and any implementation of the Kubernetes CRI (Container Runtime Interface).
 

 ## Addons

Addons use Kubernetes resources (DaemonSet , Deployment , etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

Selected addons(DNS, WEB UI, Container Resource Monitoring, Cluster-level Logging) are described below;

 #### DNS

While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it.

Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.


 #### Web UI (Dashboard)

Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.


 #### Container Resource Monitoring

Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.
 

 #### Cluster-level Logging

A cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.




>Note: As K8s is a vast some of the major topics we will be covering are as follows

 * Minikube (Intro, Install, Setup, and run)

 * Create  K8s cluster in on-premise by using kubeadm and have hands on experience
 
 * Create K8s cluster on AWS cloud using KOPS
 
 * Pods, deployments and replica set
 
 * Services

 * init-container, multi-container pod creation

 * Storage (Persistant Volume and Persistant Volume Claim)  
 
 * Rolling updates 
 
 * Namespace
 
 * Resource Quota
 
 * Limit Range
 
 * Taints( Pods scheduling)
 
 * Secret and Config-map
  
 * Pods Autoscaling (Horizontal and Vertical)

  Each and every topics mentioned above will be covered in detail including theory and practical in my next further sections.
  

 ![the end](https://media-exp1.licdn.com/dms/image/C5612AQFfARfN7n97oQ/article-cover_image-shrink_600_2000/0?e=1592438400&v=beta&t=blDgnfUk7-GU-d8yuX8cBBEOxiC7d3-KghodTiBgJtM)