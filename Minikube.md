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
  
   * Now the docker engine is installed and you can verify 
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
     ``

    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
``
     
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
 

   ![ohshit](data:image/jpeg;base64,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)
 
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

   This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
  
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
         