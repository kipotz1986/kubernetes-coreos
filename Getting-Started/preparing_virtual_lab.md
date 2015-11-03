# Preparing Virtual Lab


## Cluster Overview
In this lab we're going to build a CoreOS cluster that have 4 CoreOS nodes, And also we need 1 remote workstation to control our CoreOS cluster. 

| **Nodename** | **Role** | **IP Address** |
| -- | -- | -- |
| Workstation | Workstation | 172.16.4.11 |
| master1 | Kubernetes-master | 172.16.4.12 |
| minion1 | kubernetes-minion | 172.16.4.13 |
| minion2 | kubernetes-minion | 172.16.4.14 |
| minion3 | kubernetes-minion | 172.16.4.15 |


For workstation we are going to install Debian 7 and for other we are going to install CoreOS on it. Debian installation is not covered in this tutorial.

### Build and Publish Kubernetes Binaries


First we assume that you've already installed debian on workstacion vm and all basic functionality is working properly like networking that can connect to the internet.
Next, weare going to install some basic package that would be required for this lab.

```sudo apt-get install curl wget git htop make nginx```


make sure nginx is started and listening on port 80.

```service nginx status```  
```[ ok ] nginx is running.```

now clone kubernetes repository to your current working directory.

```git clone https://github.com/kubernetes/kubernetes.git``` 

Next we can build kubernetes binaries.

```cd kubernetes/cluster/ubuntu/```   
```./build.sh```

you can see now you have binaries folder in your current working directory. Now copy binaries folder to /usr/share/nginx/www/ folder.

```cp -rf binaries /usr/share/nginx/www/```  




### SSH public key for remote SSH CoreOS node

since CoreOS authentication is not using password based auth. We have to create a public ssh key to authenticate remote SSH to our CoreOS nodes in the future purpose. To Make SSH public key you have to execute it inside your workstation node.



