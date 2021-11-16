---
title: "Deploy Test Application"
weight: 30
draft: false
---

#### Create namespace and install application

Let's create a new namespace and deploy an application in that namespace. We will deploy a Wordpress application and MySQL database backed by persistent volumes.

Create a namespace called 'staging'
```
kubectl create namespace staging
```

{{% notice info %}}
We will be using Amazon EBS CSI Driver to manage storage for this lab. 
{{% /notice %}}

Firstly, we will create a storage class to be used by MariaDB and Wordpress to create persistent volumes.
Copy/Paste the following commands into your Cloud9 Terminal.
```
mkdir -p ~/environment/backup-restore
cd ~/environment/backup-restore
wget https://eksworkshop.com/intermediate/280_backup-and-restore/backupandrestore.files/storage-class.yaml
```
```
kubectl apply -f storage-class.yaml
```

{{% notice note %}}
We will deploy wordpress using `Helm`. If you haven't configured Helm, follow the installation instruction in this [module](/beginner/060_helm/helm_intro/install/index.html) 
{{% /notice %}}

#### Deploy Wordpress using helm in the staging namespace
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-wordpress bitnami/wordpress --set service.type=LoadBalancer --set wordpressPassword=velerodemo --set global.storageClass=staging -n staging
```

Verify deployment
```
kubectl get all -n staging
```
Output should look like below:
```
NAME                                READY   STATUS    RESTARTS   AGE
pod/my-wordpress-84bc48c77f-zv6hn   1/1     Running   0          10m
pod/my-wordpress-mariadb-0          1/1     Running   0          10m

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                      AGE
service/my-wordpress           LoadBalancer   10.100.57.128    a3f7a5611364449139d68d18006accab-1926588337.us-east-2.elb.amazonaws.com   80:30578/TCP,443:31835/TCP   10m
service/my-wordpress-mariadb   ClusterIP      10.100.106.138   <none>                                                                    3306/TCP                     10m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-wordpress   1/1     1            1           10m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/my-wordpress-84bc48c77f   1         1         1       10m

NAME                                    READY   AGE
statefulset.apps/my-wordpress-mariadb   1/1     10m
```
Verify Persistent Volume Claims
```
kubectl get pvc -n staging
```
Output should look like below:
```
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-my-wordpress-mariadb-0   Bound    pvc-47258848-6685-433f-ba50-ff5b29c0dcf8   8Gi        RWO            staging        12m
my-wordpress                  Bound    pvc-9b41ca9f-2276-4810-9109-ed2d90f9ccb6   10Gi       RWO            staging        12m
```

Access Wordpress using the load balancer created by the Service.
```
kubectl get svc -n staging --field-selector metadata.name=my-wordpress -o=jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'
```
The output should return the load balancer's url
```
a3f7a5611364449139d68d18006accab-1926588337.us-east-2.elb.amazonaws.com
```

Access the wordpress admin application at the load balancer url with path **admin** (e.g. http://a3f7a5611364449139d68d18006accab-1926588337.us-east-2.elb.amazonaws.com/admin). Login with  username **user** and password **velerodemo**.

![Token page](/images/backupandrestore/wordpress-admin.jpg)

Create a blog post for testing and publish it

![Token page](/images/backupandrestore/blogpost.jpg)

Logout and test the published blogpost by access the load balancer's url

![Token page](/images/backupandrestore/wordpress.jpg) 