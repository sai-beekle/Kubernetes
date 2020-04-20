# Create K8s multinode cluster on cloud(AWS) using KOPS

* Kops (Kubernetes Operations) is an easy to use, open source tool used for creating, destroying and upgrading Kubernetes clusters and the underlying infrastructure in a Prodution Environment. It is supported on AWS and works via CLI. There are some other ways to deploy a Kubernetes cluster, but Kops manages the entire process including the infrastructure and is very reliable and easy.

* You can create your cluster in an existing or new VPC either with a pubic or private topology.

* You will need to configure IAM permissions and an S3 bucket for the KOPS_STATE_STORE. 

* The KOPS_STATE_STORE is an storage system that stores your cluster configuration and state.

* KOPS_STATE_STORE is the source of truth for your Kops managed clusters - if anything happens to this bucket - you will be unable to manage your cluster using Kops.

* Kops can generate Terraform configurations, and then you can apply them using the terraform tools. 
 
* Kops provides High Availability supports. Kops is able to create multiple kubernetes masters, so in the event of a master instance failure, the kubernetes API server will continue to operate.
 
*Note: Kops provide high availability within the avaialabilty zone. High availabilty feature is compromised in multi availability zones.*
 

>As Master Node stores the information in EBS volume, if Master node terminated a new master node will be created and the EBS volume of terminated master node is mounted on the new master node. But EBS volume is limited or restricted within the Avaialability zone. So if a new master node is created in another AZ then the EBS volume i.e terminated master node info will not be avaialble for the newly created master node.


## Creating cluster
 
### Launch user machine

 Launch and EC2 instance in AWS and while creating look at in which VPC and in which zone you are creating. (I am creating Amazon Linuc AMI which is a free tier eligible)

*Above created machine is just a user machine and it is used to create the cluster. This is not the master node and for better understanding I refer this node as `user node`*
 
### Create role and attach to user machine

 K8s cluster creation in AWS uses some of the AWS services like S3 bucket storage, Route53, EC2 , IAM and thus to communicate to cluster the user node has the access for all these. So we need to create role and we have to attach it to the user node

### Login to user macgine and get CLI
 
Login to the user node and get into aws cli mode          
   `aws configure`
  As we have attached role to the user node you can enter into CLI without the secret key.
  While going in CLI mode it will prompt for region code provide the proper region code.

### Install and Setup kubectl
 

  * Install kubectl binary with curl on Linux
     
     
```
 curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
     
   The above mentioned command is to get the latest version of kubectl as it include   ```
           `curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt` 
            ``` and if you want to install a specific version then replace this with version number in the command line  

   For example, to download version v1.18.0 on Linux, type:

   `curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl`     
     
  * The above command install kubectl and creates a file in your home directory as non execute permission as below you can check
           
  
    So we have to make the kubectl binary executable
      `chmod +x ./kubectl` 
        Now it is became executable as you see 

   
   * Move the binary in to your PATH.
    
   `sudo mv ./kubectl /bin/kubectl`
     The ablove command moves the file *kubectl* from your home directory to the `bin/` 
     you can check by typing `ls -l` in your home directory you cannot find *kubectl* file


   * Test to ensure the version you installed is up-to-date:

     `kubectl version --client`


### Install Kops
   
 * Download the latest release with the command:
 ```
 curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
 ``` 
 * Make the kops binary executable
  
  `chmod +x kops-linux-amd64`

 * Move the kops binary in to your PATH.

  `sudo mv kops-linux-amd64 /usr/local/bin/kops`

### Create a route53(Hosted Zone) domain for your cluster 
 kops uses DNS for discovery, both inside the cluster and outside, so that you can reach the kubernetes API server from clients.

 kops has a strong opinion on the cluster name: it should be a valid DNS name. By doing so you will no longer get your clusters confused, you can share clusters with your colleagues unambiguously, and you can reach them without relying on remembering an IP address.

 Here I am creating a private hosted zone named `sai.beekle`, while creating select the VPC and region where you wanted to create the cluster.
 
 You should see the 4 NS records that Route53 assigned your hosted zone. as you see in AWS maangement console

 ![image](https://user-images.githubusercontent.com/58899893/79763314-8a9ed780-8341-11ea-9458-e00af848a202.png)

###  Create an S3 bucket to store your clusters state

* kops lets you manage your clusters even after installation. To do this, it must keep track of the clusters that you have created, along with their configuration, the keys they are using etc. This information is stored in an S3 bucket. S3 permissions are used to control access to the bucket.

* Multiple clusters can use the same S3 bucket, and you can share an S3 bucket between your colleagues who are working on the same clusters - this is much easier than passing around kubecfg files. But anyone with access to the S3 bucket will have administrative access to all your clusters, so you don?t want to share it beyond the operations team.

* Usually bucket name refers to the doamin name and as my domain name is `sai.beekle` , I will create a bucket named `engineer.sai.beekle`. 
 
* Create the S3 bucket using `aws s3 mb s3://engineer.sai.beekle`

* In Kops KOPS_STATE_STORE is a storage system that stores your cluster configuration and state. and thus we have to redirect our s3 bucket `engineer.sai.beekle` to `KOPS_STATE_STORE`
  
   `export KOPS_STATE_STORE=s3://engineer.sai.beekle`

### Create ssh keys in user node
 
 As the cluster is managed by the user node, create a ssh keys, it will generate key pairs so taht once cluster is created user node can easily access the cluster nodes.
 
  run `ssh-keygen` enter, enter, enter till keys generates. 

 
### Build cluster configuration
  
 * Create kubernetes cluster definitions on S3 bucket

    kops will create the configuration for your cluster. Note that            it only creates the configuration, it does not actually create the cloud resources.  we have to do that in the next step with    a                                    
	                 `kops update cluster`
 This give you an opportunity to review the configuration or change it.

   type `kops create cluster --cloud=aws --zones=ap-southeast-1b --name=cluster.engineer.sai.beekle --dns-zone=sai.beekle --dns private` 
 
 * Create kubernetes cluster `cluster.engineer.sai.beekle`
 
   type `kops update cluster cluster.engineer.sai.beekle --yes`


 *  Validate the cluster
 

   type `kops validate cluster cluster.engineer.sai.beekle`

 ![image](https://user-images.githubusercontent.com/58899893/79785464-87b3df00-8361-11ea-9a4c-1a7cfcc00891.png)
 





 As you see by default one master node and two worker nodes created with some configuration. 
 
 If we want to create our own kind of cluster(number of worker nodes, type of node, volume of node, defined security groups for nodes etc) then we have to pass all these parameters while creating the cluster.
 
 As you see now i will create a cluster with kops create cluster with master-node-size= t2.medium ,master-volume-size= 10 GB, worker node count= 10 , node-size= t2.micro,  node-volume-size= 8GB 

```
kops create cluster --name=cube.engineer.sai.beekle --cloud=aws \
--zones="ap-south-1a,ap-south-1b,ap-south-1c" \
--dns-zone=sai.beekle \
--dns=private \
--master-size= "t2.medium" \
--master-volume-size= 10 \
--master-public-name= "Master Node" \
--node-count= 10 \
--node-size= "t2.micro" \
--node-volume-size= 8
```

cluster will be created with 1 Master node and 10 worker nodes distributed among ap-south-1a,ap-south-1b,ap-south-1c zones with our defined type and volume size.

 * Create kubernetes cluster  cube.engineer.sai.beekle
 
   type  `kops update cluster cube.engineer.sai.beekle --yes`  

 * Validate cluster  cube.engineer.sai.beekle
 

   type `kops validate cluster cube.engineer.sai.beekle`

 ![image](https://user-images.githubusercontent.com/58899893/79786873-b6cb5000-8363-11ea-936f-757c8cf75cf1.png)

 * To check the nodes

  type `kubectl get nodes`
 
 ![image](https://user-images.githubusercontent.com/58899893/79790211-082a0e00-8369-11ea-815e-f2c522971495.png)

 

 ##### In CLI we just type two commands that is `kops create` , `kops update` and cluster is created. But at backend many steps will be taken place in AWS
 
 * A VPC is created in the region by name same as cluster name and some random CIDR block will be assigned to VPC(we can assign the CIDR block as we wanted).

 ![image](https://user-images.githubusercontent.com/58899893/79788363-2b06f300-8366-11ea-8184-c00c154e2f7f.png)
 
 * As many zones we have defined while creating the cluster those many Subnets will be created within the cluster VPC and CIDR block is assigned ( in our case we defined 3 zones and hence 3 subnets created)

  ![image](https://user-images.githubusercontent.com/58899893/79788900-fd6e7980-8366-11ea-8145-f2ec767c76dd.png)
 
 * An Internet gateway will be craeted and it will be attached to the cluster VPC

 ![image](https://user-images.githubusercontent.com/58899893/79789124-5c33f300-8367-11ea-9ce0-af1147004d8b.png)
 
 * Route tables will be created(main and non main) and subnets will be associated with the route table

 * Security groups will be created for master nodes and worker nodes( we can assign the security group for master node or worker node while creating cluster)
 
 * EC2 instance created as master node and worker node within the cluster VPC 
 
  ![image](https://user-images.githubusercontent.com/58899893/79787354-79b38d80-8364-11ea-8604-4e434dda5eb1.png)
 
 * volumes will be created and attched to the nodes(we can assign the volume type as we required while creating the cluster). and also etcd volumes will be created.

  ![images](https://user-images.githubusercontent.com/58899893/79790714-ccdc0f00-8369-11ea-9193-84e0a15c541e.png)
 
 * Elastic Network inetrfaces will be created.
 ![image](https://user-images.githubusercontent.com/58899893/79791244-9f439580-836a-11ea-94af-ccafa986d21a.png)

 ## A simple handson on the K8s Cluster
  
 **We are going to install and run a simple nginx webserver to our cluster and test whether it is accessible by the end user or not** 

 * Before creating the cluster we have created the ssh keys in user node by `ssh-keygen` which create a pair of keys, that will help us to get access to the cluster nodes with the `user node`
 
 * Login to the master node 
  type `ssh root@<public ip of master node>` 
  it will ask for login as admin so 
  type again  `ssh admin@<public ip of master node>` and then switch to root user.

 * Now you are in master node , create a deployment `web` with desired replicas of 15
  type `kubectl run  web --image=nginx --port 80 --replicas=15`
  ![image](https://user-images.githubusercontent.com/58899893/79792730-06624980-836d-11ea-8e54-338b665c2d28.png)

  The above command create a deployment web which inturn crates pods in worker nodes randomly and pods inturn create a container and run the container.
  
  we can check the pods, type `kubectl get pods -o wide`
  ![image](https://user-images.githubusercontent.com/58899893/79793141-b6d04d80-836d-11ea-9808-0940736b2462.png)

 * Conatiner is created and it is up means application is running , now the question is how to access it from client machine(from browser for end user)
  For that we have to expose the deployment as Load balancer so that it will give a DNS name and with the DNS name end user can access our application
  type `kubectl expose deployment web --type=LoadBalancer`
 
  Once we expose our deployment as Load balancer internally, NodePort and ClusterIP created. All the worker nodes  will be registerd with the LoadBalancer
  ![image](https://user-images.githubusercontent.com/58899893/79794770-79b98a80-8370-11ea-9596-73ff06d02912.png)

 If we expose our deployment as NodeBalancer then a Load Balancer will create in the cluster as you can see in AWS management console.
  ![image](https://user-images.githubusercontent.com/58899893/79794985-dae15e00-8370-11ea-91ca-cc8507eb95df.png)
 
* You can check all (deployments , services) 
 
  `kubectl get all`
  ![images](https://user-images.githubusercontent.com/58899893/79795475-9c986e80-8371-11ea-89eb-a392dc9a8629.png)

* Test our application
  from any of the browser hit the DNS name that was generated while exposing deployment as LoadBalancer. we will directed to the nginx web server's welcome page.
 ![image](https://user-images.githubusercontent.com/58899893/79795898-4a0b8200-8372-11ea-9be1-a832e409395e.png)

 ![the end](https://media-exp1.licdn.com/dms/image/C5612AQFfARfN7n97oQ/article-cover_image-shrink_600_2000/0?e=1592438400&v=beta&t=blDgnfUk7-GU-d8yuX8cBBEOxiC7d3-KghodTiBgJtM)
  




    
