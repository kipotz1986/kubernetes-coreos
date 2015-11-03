# Kubernetes Configuration

Now it's time to configure our kubernetes binaries to control our kubernetes clusters running on CoreOS Cluster. 

On your workstation machine do the following steps

```sudo cp /home/user/kubernetes/cluster/ubuntu/binaries/kubectl /usr/bin/```  
```chmod a+x /usr/bin/kubectl```  
```kubectl config set-cluster my-cluster --server=http://172.16.4.12:8080```  
```kubectl config set-context default-context --cluster=my-cluster```  
```kubectl config use-context default-context```  

Now you can start play with kubernetes.