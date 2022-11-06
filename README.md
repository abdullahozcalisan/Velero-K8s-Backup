# Velero CLI Install

Lets download the `velero` command line tool. We gonna grab `v1.9.2` release using `curl`

```
curl -L -o /tmp/velero.tar.gz https://github.com/vmware-tanzu/velero/releases/download/v1.9.2/velero-v1.9.2-linux-amd64.tar.gz
tar -C /tmp -xvf /tmp/velero.tar.gz
mv /tmp/velero-v1.9.2-linux-amd64/velero /usr/local/bin/velero
chmod +x /usr/local/bin/velero
```
* Check velero installation

```
velero --help
```

## Aws user credential configurations. (User has to S3 and EC2 permissions )

```
cat > /tmp/credentials-velero <<EOF
[default]
aws_access_key_id=<aws_access_key_id>
aws_secret_access_key=<aws_secret_access_key>
EOF
```

## Velero-v1.5.0 Install

```
    velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.5.0 \
    --bucket velero-outs \
    --backup-location-config region=eu-west-1 \
    --snapshot-location-config region=eu-west-1 \
    --secret-file /tmp/credentials-velero

```

* Check Velero configurations

```
kubectl -n velero get pods

kubectl logs deployment/velero -n velero
```
* Check Velero Backup Location
```
velero backup-location get
```
## Create a Backup (include-exclude namespaces/resources)

```
velero backup create backup1 --include-namespaces='*'  --exclude-namespaces app-ns1 --exclude-resources secrets
```


## Do a Restore

Before create velero restore, you should delete all resources or you can create new cluster for migration.
<br>
<br>
* Create Restore

```
velero restore create restore1 --from-backup backup1
```


* Describe

```
velero restore describe restore1
```

* Logs 

```
velero restore logs default-namespace-backup
```
* Check resources restored
```
kubectl get all
```

* Check Velero Backup and Restore

```
velero backup get

velero restore  get
```


## Create Schedule 
<br>

Schedule custom backup to run every 15 min 

```
velero schedule create every15min --schedule="*/15 * * * *" --include-namespaces='*'  --exclude-namespaces app-ns2 --exclude-resources secrets --ttl 0h15m0s
```
