## Prepare K8S cluster
### Create a service account for CI/CD pipeline
```
kubectl create sa cicd-access -n default
```

### As of k8s 1.25 -- need to create a non expiring SA token
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
### Create ClusterRoleBinding
```
kubectl create clusterrolebinding cicd-access-rolebinding --clusterrole cluster-admin --serviceaccount default:cicd-access
```
## CI/CD Automation
Please store TOKEN in a secret in the CI/CD pipeline
```
TOKEN=$(kubectl get secret cicd-access -o go-template='{{.data.token | base64decode}}')
```
Then you should be capable invoking the CI/CD pipeline ... (you might need to specify a host
```
kubectl  --token=$TOKEN --server=https://127.0.0.1:6443 get po
```

### To be tested  
To add it to GithubAction
````
...

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Kubernetes CLI
      uses: steebchen/kubectl@v1
      with:
        version: 'latest'
        kubeconfig: ${{ secrets.TOKEN }}

    - name: Deploy to Kubernetes
      run: |
        kubectl --token=$TOKEN --server=https://127.0.0.1:6443 --insecure-skip-tls-verify  apply -f path/to/manifest.yml

```
