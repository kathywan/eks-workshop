---
title: "Cleanup"
weight: 50
draft: false
---

Cleanup wordpress helm deployment and staging namepspace

```
helm delete my-wordpress -n staging
kubectl delete namespace staging
kubectl delete storageclass staging
```

Cleanup velero namespace

```
kubectl delete namespace velero
```

Delete S3 Bucket and IAM user
```
aws s3 rb s3://$VELERO_BUCKET --force
aws iam delete-user-policy --user-name velero --policy-name velero
aws iam delete-access-key --access-key-id $VELERO_ACCESS_KEY_ID --user-name velero
aws iam delete-user --user-name velero
```

Delete files from Cloud9 environment
```
rm -f ~/environment/velero*
rm -fr ~/environment/backup-restore
sudo rm -f /usr/local/bin/velero
```

You may also want to delete the environment variables (`$VELERO_BUCKET`, `$VELERO_ACCESS_KEY_ID`, and `$VELERO_SECRET_ACCESS_KEY`) from `~/.bash_profile` as they are no longer valid.

```
cat ~/.bash_profile | sed '/^export VELERO_*/d' | tee ~/.bash_profile
```
