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
Now generate a pair of authentication keys. Do not enter a passphrase:

```ssh-keygen -t rsa```

A new public key created inside ~/.ssh folder after key generated successfully. Now print out the key and take a note for hash code shown.

```cat ~/.ssh/id_rsa.pub```

output example :  

```ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCju4ek8z9kN12F4eC1FEitmruNaGA8v5Tm+w/Ed64jAzEz1TQuZivTv6rR6c+jHjSwfYCBISW/JXnAbBkxfWeJ4xU4VII2w+Ck2BENEnc9E0MRZOTZYwcG+vbvvD+GElfsW0Eh9j7Yh9X2PmXD63Yvcdiua95kOcGMBrETtjj/t8HbuA27OLf5xDxYN+Va6wcAZBCFTOSz2v31tVvCzF693ZFpqqiGrzpH1PGbJ7knHLPEpq7ihmaNpQ9yUIiYZaelWXTLR44QtOpEulYe/FViAQTkeRImAx+OKD6MQFF4ZVT+DUccP4XG6dhV2C4FePNxv4WFCZlZskRWUl4ilu6b root@workstation```


### The CoreOS Cloud-Config

CoreOS use cloud-config file to configure such as service, network, etc. cloud-config is writen in yaml format (JSON format is not supported at this moment). for this lab purpose we are going to create 4 cloud-config file related to our scenario where master node require specific cloud-config and for 3 minions we are going to create 3 cloud config for them.

For the master node, In the /usr/share/nginx/www/ folder create master.yaml file and write down some configurations like below.

```vi /usr/share/nginx/www/master.yaml```   

master.yaml
```
#cloud-config

users:
  - name: core
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCju4ek8z9kN12F4eC1FEitmruNaGA8v5Tm+w/Ed64jAzEz1TQuZivTv6rR6c+jHjSwfYCBISW/JXnAbBkxfWeJ4xU4VII2w+Ck2BENEnc9E0MRZOTZYwcG+vbvvD+GElfsW0Eh9j7Yh9X2PmXD63Yvcdiua95kOcGMBrETtjj/t8HbuA27OLf5xDxYN+Va6wcAZBCFTOSz2v31tVvCzF693ZFpqqiGrzpH1PGbJ7knHLPEpq7ihmaNpQ9yUIiYZaelWXTLR44QtOpEulYe/FViAQTkeRImAx+OKD6MQFF4ZVT+DUccP4XG6dhV2C4FePNxv4WFCZlZskRWUl4ilu6b root@workstation
    groups:
      - sudo
    shell: /bin/bash

write-files:
  - path: /etc/conf.d/nfs
    permissions: '0644'
    content: |
      OPTS_RPC_MOUNTD=""
  - path: /opt/bin/wupiao
    permissions: '0755'
    content: |
      #!/bin/bash
      # [w]ait [u]ntil [p]ort [i]s [a]ctually [o]pen
      [ -n "$1" ] && \
        until curl -o /dev/null -sIf http://${1}; do \
          sleep 1 && echo .;
        done;
      exit $?
hostname: master
coreos:
  etcd2:
    name: master
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
    advertise-client-urls: http://172.16.4.12:2379,http://172.16.4.12:4001
    initial-cluster-token: k8s_etcd
    listen-peer-urls: http://172.16.4.12:2380,http://172.16.4.12:7001
    initial-advertise-peer-urls: http://172.16.4.12:2380
    initial-cluster: master=http://172.16.4.12:2380
    initial-cluster-state: new
  fleet:
    metadata: "role=master"
  units:
    - name: systemd-networkd.service
      command: stop
    - name: 00-ens32.network
      runtime: true
      content: |
        [Match]
        Name=ens32

        [Network]
        DNS=203.142.82.222
        Address=172.16.4.12/24
        Gateway=172.16.4.1
    - name: down-interfaces.service
      command: start
      content: |
        [Service]
        Type=oneshot
        ExecStart=/usr/bin/ip link set ens32 down
        ExecStart=/usr/bin/ip addr flush dev ens32
    - name: systemd-networkd.service
    command: restart
    - name: generate-serviceaccount-key.service
      command: start
      content: |
        [Unit]
        Description=Generate service-account key file
        [Service]
        ExecStartPre=-/usr/bin/mkdir -p /opt/bin
        ExecStart=/bin/openssl genrsa -out /opt/bin/kube-serviceaccount.key 2048 2>/dev/null
        RemainAfterExit=yes
        Type=oneshot
    - name: setup-network-environment.service
      command: start
      content: |
        [Unit]
        Description=Setup Network Environment
        Documentation=https://github.com/kelseyhightower/setup-network-environment
        Requires=network-online.target
        After=network-online.target
        [Service]
        ExecStartPre=-/usr/bin/mkdir -p /opt/bin
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/setup-network-environment -z /opt/bin/setup-network-environment https://github.com/kelseyhightower/setup-network-envi$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/setup-network-environment
        ExecStart=/opt/bin/setup-network-environment
        RemainAfterExit=yes
        Type=oneshot
    - name: fleet.service
      command: start
    - name: flanneld.service
      command: start
      drop-ins:
        - name: 50-network-config.conf
          content: |
            [Unit]
            Requires=etcd2.service
            After=etcd2.service
            [Service]
            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{"Network":"10.244.0.0/16" }'
    - name: docker.service
      command: start
    - name: kube-apiserver.service
      command: start
      content: |
        [Unit]
        Description=Kubernetes API Server
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        Requires=setup-network-environment.service etcd2.service generate-serviceaccount-key.service
        After=setup-network-environment.service etcd2.service generate-serviceaccount-key.service
        [Service]
        EnvironmentFile=/etc/network-environment
        ExecStartPre=-/usr/bin/mkdir -p /opt/bin
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/kube-apiserver -z /opt/bin/kube-apiserver https://storage.googleapis.com/kubernetes-release/release/v1.0.3/bin/linux/$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/kube-apiserver
        ExecStartPre=/opt/bin/wupiao 127.0.0.1:2379/v2/machines
        ExecStart=/opt/bin/kube-apiserver \
        --service-account-key-file=/opt/bin/kube-serviceaccount.key \
        --service-account-lookup=false \
        --admission-control=NamespaceLifecycle,NamespaceAutoProvision,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota \
        --runtime-config=api/v1 \
        --allow-privileged=true \
        --insecure-bind-address=0.0.0.0 \
        --insecure-port=8080 \
        --kubelet-https=true \
        --secure-port=6443 \
        --service-cluster-ip-range=10.100.0.0/16 \
        --etcd-servers=http://127.0.0.1:2379 \
        --public-address-override=${DEFAULT_IPV4} \
        --logtostderr=true
        Restart=always
        RestartSec=10
    - name: kube-controller-manager.service
      command: start
      content: |
        [Unit]
        Description=Kubernetes Controller Manager
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        Requires=kube-apiserver.service
        After=kube-apiserver.service
        [Service]
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/kube-controller-manager -z /opt/bin/kube-controller-manager https://storage.googleapis.com/kubernetes-release/release$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/kube-controller-manager
        ExecStart=/opt/bin/kube-controller-manager \
        --service-account-private-key-file=/opt/bin/kube-serviceaccount.key \
        --master=127.0.0.1:8080 \
        --logtostderr=true
        Restart=always
        RestartSec=10
    - name: kube-scheduler.service
      command: start
      content: |
        [Unit]
        Description=Kubernetes Scheduler
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        Requires=kube-apiserver.service
        After=kube-apiserver.service
        [Service]
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/kube-scheduler -z /opt/bin/kube-scheduler https://storage.googleapis.com/kubernetes-release/release/v1.0.3/bin/linux/$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/kube-scheduler
        ExecStart=/opt/bin/kube-scheduler --master=127.0.0.1:8080
        Restart=always
        RestartSec=10
  update:
    group: alpha
    reboot-strategy: off
```

You will notice that in ssh-authorized keys definition section. Remember that you've already make a note about your key previously. Use your SSH key as a value for this configuration. 
Now it's time to create cloud-config for our first minion.

```vi /usr/share/nginx/www/minion1.yaml```

minion1.yaml
```
#cloud-config
users:
  - name: core
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCju4ek8z9kN12F4eC1FEitmruNaGA8v5Tm+w/Ed64jAzEz1TQuZivTv6rR6c+jHjSwfYCBISW/JXnAbBkxfWeJ4xU4VII2w+Ck2BENEnc9E0MRZOTZYwcG+vbvvD+GElfsW0Eh9j7Yh9X2PmXD63Yvcdiua95kOcGMBrETtjj/t8HbuA27OLf5xDxYN+Va6wcAZBCFTOSz2v31tVvCzF693ZFpqqiGrzpH1PGbJ7knHLPEpq7ihmaNpQ9yUIiYZaelWXTLR44QtOpEulYe/FViAQTkeRImAx+OKD6MQFF4ZVT+DUccP4XG6dhV2C4FePNxv4WFCZlZskRWUl4ilu6b root@workstation
    groups:
      - sudo
    shell: /bin/bash
write-files:
  - path: /opt/bin/wupiao
    permissions: '0755'
    content: |
      #!/bin/bash
      # [w]ait [u]ntil [p]ort [i]s [a]ctually [o]pen
      [ -n "$1" ] && [ -n "$2" ] && while ! curl --output /dev/null \
        --silent --head --fail \
        http://${1}:${2}; do sleep 1 && echo -n .; done;
      exit $?
hostname: minion1
coreos:
  etcd2:
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
    advertise-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
    initial-cluster: master=http://172.16.4.12:2380
    proxy: on
  fleet:
    metadata: "role=node"
  units:
    - name: systemd-networkd.service
      command: stop
    - name: 00-ens160.network
      runtime: true
      content: |
        [Match]
        Name=ens160

        [Network]
        DNS=203.142.82.222
        Address=172.16.4.13/24
        Gateway=172.16.4.1
    - name: down-interfaces.service
      command: start
      content: |
        [Service]
        Type=oneshot
        ExecStart=/usr/bin/ip link set ens160 down
        ExecStart=/usr/bin/ip addr flush dev ens160
    - name: systemd-networkd.service
      command: restart
    - name: fleet.service
      command: start
    - name: flanneld.service
      command: start
      drop-ins:
        - name: 50-network-config.conf
          content: |
            [Unit]
            Requires=etcd2.service
            After=etcd2.service
            [Service]
            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{"Network":"10.244.0.0/16" }'
    - name: docker.service
      command: start
    - name: setup-network-environment.service
      command: start
      content: |
        [Unit]
        Description=Setup Network Environment
        Documentation=https://github.com/kelseyhightower/setup-network-environment
        Requires=network-online.target
        After=network-online.target
        [Service]
        ExecStartPre=-/usr/bin/mkdir -p /opt/bin
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/setup-network-environment -z /opt/bin/setup-network-environment https://github.com/kelseyhightower/setup-network-envi$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/setup-network-environment
        ExecStart=/opt/bin/setup-network-environment
        RemainAfterExit=yes
        Type=oneshot
    - name: kube-proxy.service
    command: start
      content: |
        [Unit]
        Description=Kubernetes Proxy
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        Requires=setup-network-environment.service
        After=setup-network-environment.service
        [Service]
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/kube-proxy -z /opt/bin/kube-proxy https://storage.googleapis.com/kubernetes-release/release/v1.0.3/bin/linux/amd64/ku$
        ExecStartPre=/usr/bin/chmod +x /opt/bin/kube-proxy
        # wait for kubernetes master to be up and ready
        ExecStartPre=/opt/bin/wupiao 172.16.4.12 8080
        ExecStart=/opt/bin/kube-proxy \
        --master=172.16.4.12:8080 \
        --logtostderr=true
        Restart=always
        RestartSec=10
    - name: kube-kubelet.service
      command: start
      content: |
        [Unit]
        Description=Kubernetes Kubelet
        Documentation=https://github.com/GoogleCloudPlatform/kubernetes
        Requires=setup-network-environment.service
        After=setup-network-environment.service
        [Service]
        EnvironmentFile=/etc/network-environment
        ExecStartPre=/usr/bin/curl -L -o /opt/bin/kubelet -z /opt/bin/kubelet https://storage.googleapis.com/kubernetes-release/release/v1.0.3/bin/linux/amd64/kubelet
        ExecStartPre=/usr/bin/chmod +x /opt/bin/kubelet
        # wait for kubernetes master to be up and ready
        ExecStartPre=/opt/bin/wupiao 172.16.4.12 8080
        ExecStart=/opt/bin/kubelet \
        --address=0.0.0.0 \
        --port=10250 \
        --hostname-override=${DEFAULT_IPV4} \
        --api-servers=172.16.4.12:8080 \
        --allow-privileged=true \
        --logtostderr=true \
        --cadvisor-port=4194 \
        --healthz-bind-address=0.0.0.0 \
        --healthz-port=10248
        Restart=always
        RestartSec=10
  update:
    group: alpha
    reboot-strategy: off
```

Cloud Config for the next minion is same as our first minion, you can copy minon1.yaml to create minon2.yaml etc. but at this point please don'tforget to change hostname and IP address definition from 172.16.4.13 to related IP address in the table we've already discussed.

Seems like everythin is prepared for CoreOS Installation. Now let's Fire up our CoreOS Cluster.

