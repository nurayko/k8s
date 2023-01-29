List all Helm releases across all Namespaces.
<details>
  <summary>Answer</summary>

```
helm list -A --all
```
</details>
<p>&nbsp;</p>
Search the hub for the bitnami wordpress chart and the bitnami repo.
<details>
  <summary>Answer</summary>

```
helm search hub --list-repo-url wordpress | grep bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
```
</details>
<p>&nbsp;</p>
List all added repositories.
<details>
  <summary>Answer</summary>

```
helm repo list
```
</details>
<p>&nbsp;</p>

Show all versions of the wordpress chart in the bitnami repository.
<details>
  <summary>Answer</summary>

```
helm search repo bitnami/wordpress --versions
```
</details>
<p>&nbsp;</p>

Show all configurable values for version 15.0.0 of the wordpress chart.
<details>
  <summary>Answer</summary>

```
helm show values bitnami/wordpress --version=15.0.0
```
</details>
<p>&nbsp;</p>

Install the wordpress chart with version 15.0.0 and modify the readinessProbe on the wordpress containers.
Set the initialDelaySeconds to 60, periodSeconds to 15 and timeoutSeconds to 10.
Name the release my-wordpress.
<details>
  <summary>Answer</summary>

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
</details>
<p>&nbsp;</p>

The Pods will be in Pending state due to not setting up the PersistentVolumeClaim but that's fine,
we won't focus on that.
Make sure that the readinessProbe is configured properly.
<details>
  <summary>Answer</summary>

```
kubectl get po my-wordpress-559f64fb77-chc22 -o yaml | grep readinessProbe -A20
```
</details>
<p>&nbsp;</p>

Upgrade the my-wordpress release to version 15.0.6.
<details>
  <summary>Answer</summary>

```
helm upgrade my-wordpress bitnami/wordpress --version=15.0.6
```
</details>
<p>&nbsp;</p>

Make sure that the correct version is deployed.
<details>
  <summary>Answer</summary>

```
helm list
```
</details>
<p>&nbsp;</p>

Rollback the my-wordpress Release the the previous revision
<details>
  <summary>Answer</summary>

```
helm rollback my-wordpress
```
</details>
<p>&nbsp;</p>

Check again and make sure that version 15.0.0 of the bitnami/wordpress chart is deployed.
<details>
  <summary>Answer</summary>

```
helm list
```
</details>
<p>&nbsp;</p>

Uninstall my-wordpress Release.
<details>
  <summary>Answer</summary>

```
helm uninstall my-wordpress
```
</details>
<p>&nbsp;</p>

Remove the bitnami repository.
<details>
  <summary>Answer</summary>

```
helm repo rm bitnami
```
</details>
<p>&nbsp;</p>