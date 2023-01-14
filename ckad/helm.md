List all Helm releases across all Namespaces.
```
helm list -A --all
```

Search the hub for the bitnami wordpress chart and the bitnami repo.
```
helm search hub --list-repo-url wordpress | grep bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
```

List all added repositories.
```
helm repo list
```

Show all versions of the wordpress chart in the bitnami repository.
```
helm search repo bitnami/wordpress --versions
```

Show all configurable values for version 15.0.0 of the wordpress chart.
```
helm show values bitnami/wordpress --version=15.0.0
```

Install the wordpress chart with version 15.0.0 and modify the readinessProbe on the wordpress containers.
Set the initialDelaySeconds to 60, periodSeconds to 15 and timeoutSeconds to 10.
Name the release my-wordpress.
```
helm install my-wordpress bitnami/wordpress --version=15.0.0 --set readinessProbe.initialDelaySeconds=60 \
 --set readinessProbe.periodSeconds=15 --set readinessProbe.timeoutSeconds=10
# another way is to download the chart and provide values file
helm pull bitnami/wordpress --untar --version=15.0.0
vim custom-values.yaml
readinessProbe:
  initialDelaySeconds: 60
  periodSeconds: 15
  timeoutSeconds: 10
helm install my-wordpress ./wordpress -f custom-values.yaml
```
The Pods will be in Pending state due to not setting up the PersistentVolumeClaim but that's fine,
we won't focus on that.
Make sure that the readinessProbe is configured properly.
```
kubectl get po my-wordpress-559f64fb77-chc22 -o yaml | grep readinessProbe -A20
```

Upgrade the my-wordpress release to version 15.0.6.
```
helm upgrade my-wordpress bitnami/wordpress --version=15.0.6
```

Make sure that the correct version is deployed.
```
helm list
```

Rollback the my-wordpress Release the the previous revision
```
helm rollback my-wordpress
```

Check again and make sure that version 15.0.0 of the bitnami/wordpress chart is deployed.
```
helm list
```

Uninstall my-wordpress Release.
```
helm uninstall my-wordpress
```

Remove the bitnami repository.
```
helm repo rm bitnami
```