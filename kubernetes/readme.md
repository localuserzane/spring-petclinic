## Kubernetes config files for Spring PetClinic

### prerequisites

A Kubernetes cluster

A demo namespace

```
kubectl create namespace demo
```

A MysQL database

```
helm install --name petclinic-db --set mysqlDatabase=petclinic stable/mysql --namespace demo
```

### install

For a GKE cluster with LoadBalancer support use:

```
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/kubernetes/petclinic-svc.yaml
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/kubernetes/petclinic-deployment.yaml
```

For a Minikube cluster using NodePort:

```
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/kubernetes/petclinic-svc-nodeport.yaml
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/kubernetes/petclinic-deployment.yaml
```


### access the app

For a GKE cluster look up the IP using:
```
kubectl get svc petclinic -n demo -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

For minikube you can use the minikube IP address, get that by running:
```
minikube ip
```

And then, when accessing the app use the IP and port `32323` in the app URL.

### tear it all down

```
helm delete --purge petclinic-db
kubectl delete namespace demo
```
