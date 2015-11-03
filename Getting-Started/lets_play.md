# Let's Play

Before we try kubernetes we can do installation for kubernetes-ui. it is optional.
To install kubernetes-ui follow this steps :

```kubectl create -f /home/user/kubernetes/cluster/addons/kube-ui/kube-ui-rc.yaml```   

```kubectl create -f /home/user/kubernetes/cluster/addons/kube-ui/kube-ui-svc.yaml --validate=false```

now in your browser navigate to http://172.16.4.12:8080/ui it will show you a nice look Dashboard. If the dashboard is not showing, it might be the cluster is downloading kube-ui docker images to one of our minion.

Lets test out what we can do with kubernetes.

```kubectl run nginx --image=nginx --port=80```   

expose it as a service

```kubectl expose rc nginx --port=80```   

Run the following command to obtain the IP of this service we just created. There are two IPs, the first one is internal (CLUSTER_IP), and the second one is the external load-balanced IP.

```kubectl get service nginx```   

Alternatively, you can obtain only the first IP (CLUSTER_IP) by running:

```kubectl get svc nginx --template={{.spec.clusterIP}}```   

Hit the webserver with the first IP (CLUSTER_IP):

```curl <insert-cluster-ip-here>```

Now try to scale up the nginx you created before:

```kubectl scale rc nginx --replicas=3```  

And list the pods

```kubectl get pods```   

You should see pods landing on the newly added machine.