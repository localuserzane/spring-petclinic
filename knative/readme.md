## Knative config files for Spring PetClinic

### prerequisites

A Knative Serving installation

See https://github.com/knative/docs/blob/master/install/README.md

A demo namespace

```
kubectl create namespace demo
kubectl label namespace demo istio-injection=enabled
```

A MysQL database

```
helm install --name petclinic-db --set mysqlDatabase=petclinic stable/mysql --namespace demo
```

### install

```
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/knative/petclinic.yaml
```

### access the app

You can configure Knative Serving with a fixed IP for the knative-ingressgateway and a custom default domain. Once you configure your DNS you can the access the PetClinic app using your custom domain with a petclinic host prefix. See [Setting up a custom domain](https://github.com/knative/docs/blob/master/serving/using-a-custom-domain.md).

An alternative is to edit your `/etc/hosts` file and add a host name for the IP address that the knative-ingressgateway is listening on.

For a GKE cluster use:
```
kubectl get svc knative-ingressgateway -n istio-system -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

For minikube you can use the Minikube IP address, get that by running:
```
minikube ip
```

And then, when accessing the app use port `32380` in the app URL.

### tear it all down

```
kubectl delete --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/knative/petclinic.yaml
helm delete --purge petclinic-db
kubectl delete namespace demo
```
