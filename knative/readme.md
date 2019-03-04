## Knative config files for Spring PetClinic

### prerequisites

1. A [Knative Serving installation](https://github.com/knative/docs/blob/master/install/README.md) version 0.4 or later

2. A demo namespace
```
kubectl create namespace demo
```

3. [Helm installed](https://docs.helm.sh/using_helm/#installing-helm)

4. A MysQL database
```
helm install --name petclinic-db --set mysqlDatabase=petclinic stable/mysql --namespace demo
```

### install

```
kubectl apply --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/knative/petclinic.yaml
```

_NOTE_: The annotation `autoscaling.knative.dev/minScale: "1"` will prevent the app from getting scaled down to 0.

### access the app

You can configure Knative Serving with a fixed IP for the istio-ingressgateway and a custom default domain. Once you configure your DNS you can the access the PetClinic app using your custom domain with a petclinic host prefix. See [Setting up a custom domain](https://github.com/knative/docs/blob/master/serving/using-a-custom-domain.md).

An alternative is to edit your `/etc/hosts` file and add `petclinic.demo.example.com` host name for the IP address that the knative-ingressgateway is listening on.

#### For a GKE cluster

For a cluster with a LoadBalancer you can look up the IP address to use with:
```
kubectl get svc istio-ingressgateway -n istio-system -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

Add `petclinic.demo.example.com` host name for that IP address to your `/etc/hosts` file.

Access the app using http://petclinic.demo.example.com.

#### For a Minikube cluster

Enable ingress addon by running:
```
minikube addons enable ingress
```

Create an Ingress so we can route traffic coming in on port 80
```
kubectl apply -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/knative/minikube-ingress.yaml
```

Look up the Minikube IP address by running:
```
minikube ip
```

Add `petclinic.demo.example.com` host name for that IP address to your `/etc/hosts` file.

Access the app using http://petclinic.demo.example.com.

### tear it all down

```
kubectl delete --namespace demo -f https://raw.githubusercontent.com/trisberg/spring-petclinic/kubernetes/knative/petclinic.yaml
helm delete --purge petclinic-db
kubectl delete namespace demo
```
