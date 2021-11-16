---
title: "Backup and Restore"
weight: 40
draft: false
---

#### Backup staging namespace using Velero

We will back up all the resources in the staging namespace using velero. You can also include and/or exclude resources using various filters, even specify backup orders. Refer to velero [documentation](https://velero.io/docs/v1.7/index.html) for more options.

```
velero backup create staging-backup --include-namespaces staging
```

Check the status of backup
```
velero backup describe staging-backup
```

The output should look like below. Check if the Phase is Completed and if the snapshots are created.
```
Name:         staging-backup
Namespace:    velero
Labels:       velero.io/storage-location=default
Annotations:  velero.io/source-cluster-k8s-gitversion=v1.20.7-eks-d88609
              velero.io/source-cluster-k8s-major-version=1
              velero.io/source-cluster-k8s-minor-version=20+

Phase:  Completed

Errors:    0
Warnings:  0

Namespaces:
  Included:  staging
  Excluded:  <none>

Resources:
  Included:        *
  Excluded:        <none>
  Cluster-scoped:  auto

Label selector:  <none>

Storage Location:  default

Velero-Native Snapshot PVs:  auto

TTL:  720h0m0s

Hooks:  <none>

Backup Format Version:  1.1.0

Started:    2021-11-16 04:05:56 +0000 UTC
Completed:  2021-11-16 04:05:59 +0000 UTC

Expiration:  2021-12-16 04:05:56 +0000 UTC

Total items to be backed up:  58
Items backed up:              58

Velero-Native Snapshots:  2 of 2 snapshots completed successfully (specify --details for more information)
```

Access Velero S3 bucket using AWS Managment Console and verify if 'staging-backup' have been created.
![Title Image](/images/backupandrestore/velero-bucket.jpg)

#### Simulate a disaster

Let's delelte the 'staging' namespace to simulate a disaster
```
kubectl delete namespace staging
```

Verify that MariaDB and Wordpress are deleted. The command below should return *No resources found.*
```
kubectl get all -n staging
```

#### Restore staging namespace

Run the velero restore command from the backup created. It may take a couple of minutes to restore the namespace. 
```
velero restore create --from-backup staging-backup
```
You can check the restore status using the command below:
```
velero restore get
```
Check restore STATUS in the output.
```
NAME                            BACKUP           STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
staging-backup-20211116041004   staging-backup   Completed   2021-11-16 04:10:05 +0000 UTC   2021-11-16 04:10:08 +0000 UTC   0        0          2021-11-16 04:10:05 +0000 UTC   <none>
```

Verify if deployment, statefulset, services and pods are restored
```
kubectl get all -n staging
```
Output will look something like below:
```
NAME                                READY   STATUS    RESTARTS   AGE
pod/my-wordpress-84bc48c77f-zv6hn   1/1     Running   0          76s
pod/my-wordpress-mariadb-0          1/1     Running   0          76s

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)                      AGE
service/my-wordpress           LoadBalancer   10.100.227.77   a0f8ee30fbcb24156aa470104d0ad5ff-1822813947.us-east-2.elb.amazonaws.com   80:32608/TCP,443:30468/TCP   76s
service/my-wordpress-mariadb   ClusterIP      10.100.63.63    <none>                                                                    3306/TCP                     76s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-wordpress   1/1     1            1           76s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/my-wordpress-84bc48c77f   1         1         1       76s

NAME                                    READY   AGE
statefulset.apps/my-wordpress-mariadb   1/1     76s

```

Access Wordpress using the load balancer created by the Service.
```
kubectl get svc -n staging --field-selector metadata.name=my-wordpress -o=jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}'
```
The output should return the load balancer's url which will be different from the previous url when wordpress was originally deployed.
```
a0f8ee30fbcb24156aa470104d0ad5ff-1822813947.us-east-2.elb.amazonaws.com
```

Access the wordpress application at load balancer's url and verify if the blog post you created is restored.
