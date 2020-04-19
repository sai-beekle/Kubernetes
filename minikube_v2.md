# Minikube

 ## Introduction:

 * Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

  *Note: Minikube creates a cluster inside VM only*
 
 * Single cluster means one master node and only one worker node created within the VM.

 * Minikube usually used for testing puropse, locally on our machine.

![image](https://3.bp.blogspot.com/-lpJa8w6XMKY/XFhoY5dDQ8I/AAAAAAAADC0/t2Ojfo2CM_wbe1TGNo8jq5JE6JJYJbYbQCLcBGAs/s640/minikube-architecture.png)
 
  
  ## Install and work with Minikube
 * My Host machine is Windows OS 
 
 * I am going to create a VM (Linux Centos) on Virtual Box. You can find the centos image along with username and password [here](https://www.linuxvmimages.com/images/centos-7/)

 * Open Virtual Box and import the image and create a centos VM. Start the VM and login
 
 * To get user friendly hands on experience of the VM get its access via putty ssh. download [putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) and install.
 
 * on your VM type `ifconfig` and get the ip address as shown below
  
  ![screenshot](https://user-images.githubusercontent.com/58899893/79641891-f0f7ee80-81b7-11ea-9a9b-92bdc2fa0799.png)
 
 * Open putty and enter this ip and connect to the VM. Give username and password and get into VM
 
 * As you are logged into as centos user. switch user as root user  type `sudo -i`

 * Create a user (give your name ) and set password for user 
   `adduser <username>`
   `passwd <username>`  it will prompt for set a password. type the password you want to set and enter . Again retype the same password and enter. Password set successfully.

 * Give root permission for the user (Be the root user)
  type `vi /etc/sudoers` and enter
  once the file opens type `<esc>:set nu` and enter then file will be shown in lines
  at line no 100 add an entry and type   
                                   `<username> ALL=(ALL)  NOPASSWD: ALL` save and quit. `<esc>:wq!` enter.

 * Switch to user as <username> `su   - <username>' enter if prompt for password provide it.
 
 * Now you logged in as normal user in VM and our VM is ready for Minikube install and setup.
 
 * As minikube manages creates the cluster and for managing the containers within the cluster we need docker engine ready and running. So we have to insatll latest docker engine in our centos machine, along with start and enable it.

  ## Install docker engine
   
  *We are installing the docker engine using the repository*

  Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.
 
  1 SET UP THE REPOSITORY
  
  ```
  $ sudo yum install -y yum-utils

  $ sudo yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
```
2 Install the docker packages.
    `$ sudo yum install docker-ce docker-ce-cli containerd.io` while installing if asks for additional permission provide yes
  
   Now the docker engine is installed and you can verify 
        `docker --version`

   * At this point docker engine is just installed but not running and thus you will not be provisioned to manage the containers.
       you can verify by running docker commands like `docker ps` output shows like *Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?*
      
   * Thus we have to start the docker engine by typing `systemctl start docker`
        Docker engine is started, and if machine got rebooted then once again we have to manually start the docker engine. To avoid this we have to enable it by typing `systemctl enable doacker`
    
  * Now try to type docker command `docker ps`
  
      OOOOppppppppppppsssssssssssssssssssss!!!!!!!!!!!!!!!!!!!! NOT WORKING 
      Getting the error *Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied*
     
   NOTE:
   
   We logged in and working as non root user.The Docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user **root**. The Docker daemon always runs as the **root** user.
      To resolve this issue create a Unix group called `docker` and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the `docker` group. 

               

   * To create the `docker` group and add your user:

        1 Create the `docker` group.
          `$ sudo groupadd docker`

        2 Add your user to the `docker` group.
          `$ sudo usermod -aG docker $USER`

        3  To activate the changes made 
           `newgrp docker`
    
     * Now run the docker commands `docker ps`
       
Yayyyyy:) Its working
      
   * To verify docker engine is working fine you can check by creating a simle container

        `docker run hello-world`
        This will pull the latest image of hello-world and create a container out of it and displays as below
   

      ![image](https://user-images.githubusercontent.com/58899893/79645177-543f4c00-81cb-11ea-881f-5f6a107eeca9.png)

       This means docker is working fine with docker registry(docker hub repo)

      

   So now we have successfully installed the docker engine in our VM and it is working fine.
   
   But as you know K8s is a cluster and to communicate with the cluster we need some sort of command line tool.
    Here in K8s `kubectl` is a such CLI (Command Line Interface) tool.

 
  3 Install and Setup kubectl
 
   * The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.   
   * Kubectl is a command line tool for controlling Kubernetes clusters. kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

   1 Install kubectl binary with curl on Linux
     
     

    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

     
   The above mentioned command is to get the latest version of kubectl as it include   ```
           `curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt` 
            ``` and if you want to install a specific version then replace this with version number in the command line  

   For example, to download version v1.18.0 on Linux, type:

   `curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl`     
     
   2 The above command install kubectl and creates a file in your home directory as non execute permission as below you can check
            ![image3](https://user-images.githubusercontent.com/58899893/79665175-fdd60b80-81d2-11ea-9284-ad17135251c7.png)
  
  So we have to make the kubectl binary executable
      `chmod +x ./kubectl` 
        Now it is became executable as you see 
          ![image4](https://user-images.githubusercontent.com/58899893/79668669-26aad080-81d4-11ea-9118-009fc3fc0040.png)

   
3 Move the binary in to your PATH.
    
   `sudo mv ./kubectl /bin/kubectl`
     The ablove command moves the file *kubectl* from your home directory to the `bin/` 
     you can check by typing `ls -l` in your home directory you cannot find *kubectl* file


  4 Test to ensure the version you installed is up-to-date:

   `kubectl version --client`
  
   If everything is great, you will get the output something like this below
    ![image5](https://user-images.githubusercontent.com/58899893/79669044-8f934800-81d6-11ea-9534-7a668bac9d97.png) 
  
   If you type `kubectl get nodes` you will get the error message as *error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable*
   as we didnot created cluster yet.
     To create a cluster we have to install **Minikube**



  4 Install Minikube 
    Install Minikube via direct download( This is for Linux)
  `curl -Lo minikubehttps://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`
                                                                                                             
   The above command install minikube and creates a file in your home directory as non execute permission as same as kubectl, you can check the same way as we did it for kubectl.
     So we have to make the minikube binary executable
      `chmod +x minikube`
      
  Move the binary in to your PATH.
    `sudo mv kubectl /bin/`
    
   The ablove command moves the file *kubectl* from your home directory to the `bin/` 
     you can check by typing `ls -l` in your home directory you cannot find *minikube* file
    
  Now Minikube is installed, so start the minikube with specifying the VM driver. here I am not using any VM drivers. So I just make it as none.

  `$ sudo minikube start --driver=none`
   

   Guys..... I get an error saying *minikube v1.9.2 on Centos 7.7.1908
 Using the none driver based on user configuration
X Sorry, Kubernetes v1.18.0 requires conntrack to be installed in root's path* 
    

  So Now I will install the conntrack package
     
   `$ sudo yum install conntrack -y`

   Again I will start the minkube 
                `$ sudo minikube start --driver=none`
 

 
 
Getting the error as **Number of required CPU mismatch** as shown below

  ![image6](https://user-images.githubusercontent.com/58899893/79670989-742f3980-81e4-11ea-8499-93805fdf9f71.png)

   To resolve the issue I have to exit the VM change the VM setting and make CPU as 2

   So now I am back.....................
  
   again type `$ sudo minikube start --driver=none`
 

   HURRAYYY...............

  Minikube started and created a single node cluster as you see  below.
  
  ![image7](https://user-images.githubusercontent.com/58899893/79672362-c2493a80-81ee-11ea-89bd-a8783c10d8c1.png)

    
  Now you can check the cluster status 
     
   `minikube status`

  We are getting the output as **There is no local cluster named "minikube" - To fix this, run: "minikube start"**

 What does it mean!!! just now only minikube was started right???

 This is because kubectl and minikube configuration will be stored in /root and we are accesing it as non root user. So we are getting this error
 
  Just switch to rrot user and `ls -al` you will get `.kube` and `.minikube` hidden directories as below
    
  ![image8](https://user-images.githubusercontent.com/58899893/79672607-41d80900-81f1-11ea-8601-4f8387b63b23.png)

 #### There are two ways to resolve this issue
 
   * First Way:
    To use kubectl or minikube commands as non root user, relocate the directories from root to specific user's home directory
     `$ sudo mv /root/.kube $HOME`
     
     `$ sudo mv /root/.minikube $HOME`
     
 change the owner permission

   `sudo chown -R $USER $HOME/.kube`
   
   `sudo chown -R $USER $HOME/.minikube`
    
    
* Second Way:
This can also be done automatically by setting the env var `CHANGE_MINIKUBE_NONE_USER=true`
  
  *I prefer the second one*

  **The above mentioned issue is not raised if you are working as root user**

Now you can check the status 
   
 ![image](https://user-images.githubusercontent.com/58899893/79674351-f5e09080-81ff-11ea-84b4-e2bf9e43d976.png)

 You can check the node

 ![image](https://user-images.githubusercontent.com/58899893/79674376-3b04c280-8200-11ea-9e0b-2f1da379842e.png)

Work on minikube
 
  ![image](https://user-images.githubusercontent.com/58899893/79674444-f0377a80-8200-11ea-8ed9-9bd69f511797.png)

  ![image](https://user-images.githubusercontent.com/58899893/79674483-46a4b900-8201-11ea-9291-982379efe364.png)

delete the cluster

![image](https://user-images.githubusercontent.com/58899893/79674533-a8fdb980-8201-11ea-99bd-7e73b116be1d.png)


![the end](https://media-exp1.licdn.com/dms/image/C5612AQFfARfN7n97oQ/article-cover_image-shrink_600_2000/0?e=1592438400&v=beta&t=blDgnfUk7-GU-d8yuX8cBBEOxiC7d3-KghodTiBgJtM) 
         