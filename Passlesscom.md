# Passwordless communication(over ssh) between Linux Machines

In todays world ecrything is automation and to process the smooth automation works the machines to be accessed from one another hassle free.

Tools like Ansible,Kubenetes,Chef it is required to communicate servers automatically, in such cases it is mandatory.

To achieve this we have to build a passwordless communication between the machine.

## Scenario based setup

Here we are going to setup passwordless communication between masternode and workernode1  

Masternode has master user and workernode1 has worker1 and worker2 users, 

Setup is as shown,

![image](https://user-images.githubusercontent.com/58899893/80068568-70d9dc00-855d-11ea-8e2e-9ecab23bf060.png)


Make the host entry in hosts file of master for visibility

In masternode as a root user  type `vi /etc/hosts` 

`(ip of workernode1)  workernode1`   

In masternode user master is added to sudoers file and gave sudo permission, in same way worker1 and worker2 users also added in sudoers file and gave sudo permission.
 
as a root user `vi /etc/sudoes`
 
make an entry as below

![image](https://user-images.githubusercontent.com/58899893/80068729-ba2a2b80-855d-11ea-8ee2-98762156cd79.png)




Workernode1 ssh file need to be configured such that it can be connected by other systems over ssh

as a root user `vi /etc/ssh/sshd_config`

enable 'PermitRootLogin yes`  enable `PasswordAuthentication yes` disable `PasswordAuthentication no` as shown below

![image](https://user-images.githubusercontent.com/58899893/80069060-50f6e800-855e-11ea-80f1-9714857070dc.png)

restart the sshd servive as `servive sshd restart`

Now from the masternode as master user if you try to login workernode1

`ssh worker1@workernode1`  it will be asking for password for worker1 user, if you provide the password it will get access. Same with worker2 user as you see below

![image](https://user-images.githubusercontent.com/58899893/80069142-7edc2c80-855e-11ea-9fc1-808e13191260.png)


So now we have to generate a ssh key by `ssh-kengen` it will create a .ssh directory in master users home directory and saves a key pair(pvt and public) i.e id_rsa.pub and id_rsa as you see

in masternode type `ssh-keygen` and enter ,enter, enter for defaults
 
![image](https://user-images.githubusercontent.com/58899893/80069486-0c1f8100-855f-11ea-97e1-cd29ddbf36bc.png)


Now we have to place the public key in machine which we want to access, i.e workernode1

While copying public key to workernode1, it will ask for password (last time)

As workernode1 has two users and we want to access both the users, so copy the public key in both users home directory.

in masternode type `ssh-copy-id -i $HOME/.ssh/id_rsa.pub worker1@workernode1` and `ssh-copy-id -i $HOME/.ssh/id_rsa.pub worker2@workernode1` this will successfully place pubic key in both users home directory.
 
Now you can see a hidden dirctory .ssh created in users home directory.

![image](https://user-images.githubusercontent.com/58899893/80068373-235d6f00-855d-11ea-854e-374e286a0b8f.png)

Try to access workernode1 from masternode.

This time you access it without asking for password

![image](https://user-images.githubusercontent.com/58899893/80070088-070f0180-8560-11ea-9919-9d9a6a2045b8.png)

## Test 

* From masternode access worker1 user for workernode1 and create a `testfile` put some content. And the same file with content will appear in workernode1's worker1 home directory.

![image](https://user-images.githubusercontent.com/58899893/80070429-8e5c7500-8560-11ea-8045-b50d219eb7e8.png)

* From masternode access worker2 user for workernode1 and create a `myfile` put some content. And the same file with content will appear in workernode1's worker2 home directory.

![image](https://user-images.githubusercontent.com/58899893/80070519-b350e800-8560-11ea-9e1b-eafcac66a1d9.png)


So now we successfully built passwordless communication between servers.


![the end](https://media-exp1.licdn.com/dms/image/C5612AQFfARfN7n97oQ/article-cover_image-shrink_600_2000/0?e=1592438400&v=beta&t=blDgnfUk7-GU-d8yuX8cBBEOxiC7d3-KghodTiBgJtM) 




    


