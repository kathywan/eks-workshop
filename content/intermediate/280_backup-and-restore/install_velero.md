---
title: "Install Velero"
weight: 20
draft: false
---

#### Install Velero CLI binary

Download the [latest release's](https://github.com/vmware-tanzu/velero/releases/latest) tarball for your client platform (Example: velero-v1.7.0-linux-amd64.tar.gz)
```
wget https://github.com/vmware-tanzu/velero/releases/download/v1.7.0/velero-v1.7.0-linux-amd64.tar.gz
```
Extract the tarball:
``` 
tar -xvf velero-v1.7.0-linux-amd64.tar.gz -C /tmp
```
Move the extracted velero binary to /usr/local/bin
```
sudo mv /tmp/velero-v1.7.0-linux-amd64/velero /usr/local/bin
```
Verify installation
```
velero version
```
Output should look something like below. We see an error getting server version because we have not installed velero on EKS cluster yet.
```
Client:
        Version: v1.7.0
        Git commit: 9e52260568430ecb77ac38a677ce74267a8c2176
<error getting server version: no matches for kind "ServerStatusRequest" in version "velero.io/v1">
```

#### Install Velero on EKS

Let's install velero on EKS with AWS plugin which will support backup to S3 and snapshot operations with EBS. All the velero resources will be created in a namespace called velero.

```
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.3.0 \
    --bucket $VELERO_BUCKET \
    --backup-location-config region=$AWS_REGION \
    --snapshot-location-config region=$AWS_REGION \
    --secret-file ./velero-credentials
```

Inspect the resources created.

```
kubectl get all -n velero
```
Output should look something like this:

```
NAME                         READY   STATUS    RESTARTS   AGE
pod/velero-fbf6dfbc8-7fqkp   1/1     Running   0          10s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/velero   1/1     1            1           10s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/velero-fbf6dfbc8   1         1         1       10s
```
