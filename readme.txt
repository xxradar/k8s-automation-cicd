## Prepare K8S cluster
```
kubectl create sa cicd-access -n default
```

## As of k8s 1.25 -- need to create a non expiring SA token
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: cicd-access
  annotations:
    kubernetes.io/service-account.name: cicd-access
EOF
```
```
kubectl create clusterrolebinding cicd-access-rolebinding --clusterrole cluster-admin --serviceaccount default:cicd-access
```
## CICD Automation
Please store TOKEN in a secret in the CI/CD pipeline
```
TOKEN=$(kubectl get secret cicd-access -o go-template='{{.data.token | base64decode}}')
```
Then you should be capable invoking the CI/CD pipeline ... (you might need to specify a host
```
kubectl  --token=$TOKEN get po
```
