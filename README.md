# linc-devops-test
Describe steps to create a minimum environment for Jenkins.

### Prerequisites
- `docker`
- `kubectl`
- `helm v3`

### Kubernetes
Run k0s upon Docker
```
docker run -d --name k0s-controller --hostname controller --privileged -v /var/lib/k0s -p 6443:6443 k0sproject/k0s
```

Get `kubeconfig`
```
docker exec k0s-controller cat /var/lib/k0s/pki/admin.conf > kubeconfig
```

Get cluster info
```
kubectl --kubeconfig kubeconfig cluster-info
```

### Jenkins
Create namespace `jenkins`
```
kubectl --kubeconfig kubeconfig create ns jenkins
```

Create directory for persistence
```
docker exec k0s-controller mkdir /mnt/jenkins
docker exec k0s-controller chown 1000:1000 /mnt/jenkins
```

Apply pv
```
kubectl --kubeconfig kubeconfig apply -f templates/PersistentVolume.yaml
```

Apply pvc
```
kubectl --kubeconfig kubeconfig apply -f templates/PersistentVolumeClaim.yaml
```

Install Jenkins
```
helm upgrade --install --kubeconfig kubeconfig -n jenkins jenkins jenkins/jenkins --set persistence.existingClaim=jenkins
```

Get the Jenkins URL
```
kubectl --kubeconfig kubeconfig -n jenkins port-forward service/jenkins 8080
```

Get the 'admin' user password
```
kubectl get secret --kubeconfig kubeconfig --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode
```
