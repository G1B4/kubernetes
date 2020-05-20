# kubernetes
Kubernetes playground environment, vagrant file will create a Master and 2 Workers nodes.   KubeAdm is used to create the cluster.

## Cloning the Repo
git clone https://github.com/G1B4/kubernetes.git

## Helm (https://helm.sh/docs/intro/install/)  

Select the version of Helm you want to install to from (https://github.com/helm/helm/releases).
Then, run the following command from master node: 

wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz  
tar zxf helm-v3.2.1-linux-amd64.tar.gz 
cd linux-amd64  
sudo mv helm /usr/local/bin/helm 
which helm (to check it was moved properly)  
helm version --short  
helm repo add stable https://kubernetes-charts.storage.googleapis.com/  
helm search repo phpmyadmin (to check if repo was added correctly)  
helm install gabophpadmin stable/phpmyadmin  


To migrate from a previous version of helm to version 3, see the following video https://youtu.be/aAPtT4uaY1o?list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0

## Create NFS Server

Login to Kmaster.

sudo mkdir /srv/nfs/kubedata -p  
sudo chown nfsnobody: /srv/nfs/kubedata/   
sudo yum install -y nfs-utils   
sudo systemctl enable nfs-server   
sudo systemctl start nfs-server   
sudo systemctl status nfs-server   
sudo vi /etc/exports -> add the following: /srv/nfs/kubedata    *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)   
sudo exportfs -rav   
  
**Testing** -> login into a worker node and run the following:   
sudo mount -t nfs 172.42.42.100:/srv/nfs/kubedata /mnt   
mount | grep kubedata   
sudo umount /mnt   

## Create Dynamic NFS Provisioning ( https://blog.exxactcorp.com/deploying-dynamic-nfs-provisioning-in-kubernetes/ )


cd /  
cd /home/vagrant/kubernetes/yamls/nfs-provisioning  
kubectl create -f 1-rbac.yaml -f 2-class.yaml -f 3-deployment.yaml -f 4-pvc-nfs.yaml  

**Or you can run everything in multipe steps and verify each of them:** 
kubectl create -f 1-rbac.yaml 
kubectl create -f 2-class.yaml  
kubectl create -f 3-deployment.yaml (Change IP for Kmaster Ip address) 
kubectl get all  
kubectl describe pod nfs-client-provisioner-<XXXX>  
kubectl get pv,pvc  (There aren't any of them) 
ls /srv/nfs/kubedata/ (There's nothing here too)  
kubectl create -f 4-pvc-nfs.yaml
kubectl get pvc,pv

**Testing** 
kubectl create -f busybox-pv-nfs.yaml (create a pod to test persist volume claim)  
kubectl exec -it busybox -- ./bin/sh (create a file using touch)  
Goto folder srv/nfs/kubedata (you will see the volume and the file there)

 
  




## Prometheus   ( https://youtu.be/CmPdyvgmw-A?list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0  - minute: 13 )
cd /home/vagrant/kubernetes/yamls/prometheus  
helm search repo prometheus  
kubectl create namespace prometheus  
helm install prometheus stable/prometheus  --values prometheus.values --namespace prometheus  

**Notes:**  
Create config file run the folowing : helm inspect values stable/prometheus > prometheus.values   
Edit the file: vi prometheus.values (Change type for Cluster it to NodePort and add a nodePort:32322 for service prometheus-server)  

## Grafagna  ( https://youtu.be/CmPdyvgmw-A?list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0  - minute: 19 )  
cd /home/vagrant/kubernetes/yamls/grafana  
helm search repo grafana  
kubectl create namespace grafana  
helm install grafana stable/grafana  --values grafana.values --namespace grafana  

**Notes:**  
Create config file run the folowing : helm inspect values stable/grafana > grafana.values   
Edit the file: vi grafana.values (Change type for Cluster it to NodePort and add a nodePort:32323 value for service prometheus-server, change adminPassword: grafana1234, change persistence to true)


## Metal LB



## Dashboard (https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

Since dashboard service is ClusterIP, can't be accessed from outside. So, we need to edit the service and convert it to NodePort. 

Steps:  
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard  (to check how the service is actually configured)  
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard  (change type to NodePort, add nodePort: 32000 below targetPort)  

cd kubernetes/dashboard  
kubectl create -f sa_cluster_admin.yaml  
kubectl -n kube-system describe sa dashboard-admin  
kubectl -n kube-system describe secret dashboard-admin-token-cd9bb  

Access to the dashboard at: https://172.42.42.101:32323/ and paste the token of the previous step.   

## TroubleShooting  

https://docs.bitnami.com/tutorials/troubleshoot-kubernetes-deployments/  

