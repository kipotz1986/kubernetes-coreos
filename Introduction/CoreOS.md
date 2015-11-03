# Introduction


## CoreOS Overview


CoreOS is an Open-Source and lightweight operating system based on Linux kernel and designed for providing infrastructure for clustered deployment, while focusing on automation, ease of security, high availability and scalability. CoreOS provide minimal functionality required for deploying application inside software container such as docker or rkt together with built-in mechanism for service discovery and configuration sharing.

CoreOS provide no package manager as a way to distribute payload applications, so everything needs to be installed and used via docker containers.

The DOcker run as application inside CoreOS. Containers can provide very good flexibility for application packaging and can start quickly. The following image shows the simplicity of CoreOS. Bottom part is Linux OS, the second level is etcd/fleet with docker daemon and the top level are running containers on the server.

![](https://coreos.com/assets/images/media/Host-Diagram.png)

By default CoreOS  is designed to work in a clustered form, but it also works very well as a single host. It is very easy to control and run application containers across cluster machines with fleet and use the etcd service discovery to connect them as it shown in
the following image.

![](http://infoslack.com/images/etcd-cluster.png)

CoreOS can be deployed easily on all major cloud providers, for example, Google Cloud, Amazon Web Services, Digital Ocean, and so on. It runs very well on baremetal servers as well. Moreover, it can be easily installed on a laptop or desktop with Linux, Mac OS X, or Windows via Vagrant, with VirtualBox or VMware virtual machine support.
This short overview should throw some light on what CoreOS is about and what it can do. Let's now move on to the real stuff and install CoreOS on to our Virtual Lab environment.