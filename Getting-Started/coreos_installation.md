# CoreOS Installation


Before you start installing CoreOS, you have to already downloaded CoreOS ISO image file from official CoreOS Download page.
https://coreos.com/os/docs/latest/booting-with-iso.html

If you're using virtual environment such as VM Ware ESXI of VirtualBox, Please mount CoreOS ISO image file to Master node and Minion nodes. If you're using bare metal infrastructure, please conduct to read users manual on how to install OS based on ISO image file, or you can burn it to DVD and install it with DVD. In this lab we're using Virtual environment.

## Master Node

Fire up master node within CoreOS ISO image already mounted. wait for few moment until shell is shown. At this point we don't need to input user and password.

in the shell please do the following command to download master.yaml from our workstation.

```wget http://172.16.4.11/master.yaml```

now you can install coreos to disk with a single command.

```sudo coreos-install -d /dev/sda -c master.yaml -C stable```

then installation progress is begin and writing to disk. The node will be restarted automatically after installation progress is completed.


## Minion Nodes

Next, we are going to install CoreOS in our all minion nodes one by one. Start our minon node make sure CoreOS ISO file is already mounted to our virtual machine. let the machine boot up and showing shell command line without password outhentication. 

first download the minon1.yaml configuration file from our workstation node.

```wget http://172.16.4.11/minion1.yaml```

now you can install coreos to disk with a single command.

```sudo coreos-install -d /dev/sda -c minion1.yaml -C stable```

The machine will reboot After installation successfully. and it will boot up but now it ask you for user and password for login. since we don't use password based authentication, we're going to ignore this things. let the machine as it is.

For next minion you can do the same step for installing CoreOS. but Please mind that you have to download different yaml file for your other minon related to your minon(X).yaml file.


## SSH to your CoreOS nodes.

Now you can remote SSH to your CoreOS nodes from your workstation machine without password, because we've already put public ssh key into our COreOS nodes. The default user for CoreOS is 'core' .

ssh to master node.
```ssh core@172.16.4.12```

after you successfully login now you can dive into your master node. Please check if the network is configured properly by running.

```ifconfig -a```

you'll notice that there is docker.0 interface and flannel interface.

Flannel is a software based overlay network. It allows for our docker container to communicate with another container across cluster. Flannel is not the only one solution for overlaying network. We can use another SDN both hardware and software . Since Flannel is the quickest way to implement and highly tested with kubernetes, we use it for simplicity.

now you can end ssh session from master and take a look for our minions. please check the network configuration and service. if network is not shown up and cannot ping to other node and to the internet, it might be a problem with cloud-config file in the network definitions. Because somehow different network hardware can cause a different interface name. If the network is up and running well but no additional service like docker and flannel started, it might be still downloading the container. From now you have to wait, while waiting you can check and monitor your service availability
with ifconfig command.

after couples of few minutes playing around with our CoreOS cluster, you have to restart all minions node. Because the docker interface needs to be manipulated after flannel service is up and running. 



