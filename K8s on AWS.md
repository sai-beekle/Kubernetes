# Create K8s multinode cluster on cloud(AWS) using KOPS

* Kops (Kubernetes Operations) is an easy to use, open source tool used for creating, destroying and upgrading Kubernetes clusters and the underlying infrastructure in a Prodution Environment. It is supported on AWS and works via CLI. There are some other ways to deploy a Kubernetes cluster, but Kops manages the entire process including the infrastructure and is very reliable and easy.

* You can create your cluster in an existing or new VPC either with a pubic or private topology.

* You’ll need to configure IAM permissions and an S3 bucket for the KOPS_STATE_STORE. 

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
 
 You should see the 4 NS records that Route53 assigned your hosted zone.

###  Create an S3 bucket to store your clusters state

* kops lets you manage your clusters even after installation. To do this, it must keep track of the clusters that you have created, along with their configuration, the keys they are using etc. This information is stored in an S3 bucket. S3 permissions are used to control access to the bucket.

* Multiple clusters can use the same S3 bucket, and you can share an S3 bucket between your colleagues who are working on the same clusters - this is much easier than passing around kubecfg files. But anyone with access to the S3 bucket will have administrative access to all your clusters, so you don’t want to share it beyond the operations team.

* Usually bucket name refers to the doamin name and as my domain name is `sai.beekle` , I will create a bucket named `engineer.sai.beekle`. 
 
* Create the S3 bucket using `aws s3 mb s3://engineer.sai.beekle`

* In Kops KOPS_STATE_STORE is a storage system that stores your cluster configuration and state. and thus we have to redirect our s3 bucket `engineer.sai.beekle` to `KOPS_STATE_STORE`
  
   `export KOPS_STATE_STORE=s3://engineer.sai.beekle`

### Create ssh keys in user node
 
 As the cluster is managed by the user node, create a ssh keys, it will generate key pairs so taht once cluster is created user node can easily access the cluster nodes.
 
  run `ssh-keygen` enter, enter, enter till keys generates. 

 
### Build cluster configuration
  
 * Create kubernetes cluster definitions on S3 bucket

  kops will create the configuration for your cluster. Note that it only creates the configuration, it does not actually create the cloud resources - you’ll do that in the next step with a `kops update cluster`. This give you an opportunity to review the configuration or change it.

   `kops create cluster --cloud=aws --zones=ap-southeast-1b --name=cluster.engineer.sai.beekle --dns-zone=sai.beekle --dns private` 
 
 * Create kubernetes cluster 
 
  `kops update cluster cluster.engineer.sai.beekle --yes`

 * Validate your cluster
 
  `kops validate cluster`
 
 * To list nodes
 
  `kubectl get nodes`

