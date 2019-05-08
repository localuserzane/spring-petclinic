## Deploying Spring PetClinic with kf

### prerequisites

1. A [Knative Serving installation](https://github.com/knative/docs/blob/master/install/README.md) version 0.5 or later

2. The `kf` binary, build from [source](https://github.com/poy/kf.git)

3. Kubernetes Service Catalog installed, install [using Helm](https://kubernetes.io/docs/tasks/service-catalog/install-service-catalog-using-helm/)
  run:
  ```
  helm install . --name catalog --namespace catalog
  ```

4. The GCP Service Broker installed from the [develop branch](https://github.com/GoogleCloudPlatform/gcp-service-broker/tree/develop/deployments/helm/gcp-service-broker)
  run:
  ```
  helm dependency update
  helm install . --name broker --namespace catalog
  ```

5. The `svcat` CLI [installed](https://github.com/kubernetes-incubator/service-catalog/blob/master/docs/install.md#manual)

### sample Spring Boot app deployment

#### enable Istio Egress for GCP

First the Google Cloud Platform APIs:
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: googleapis
spec:
  hosts:
  - "*.googleapis.com"
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: HTTPS
EOF
```

And then the metadata server:
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: metadata-server
spec:
  hosts:
  - metadata.google.internal
  - 169.254.169.254
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
EOF
```

#### provision a MySQL database and create the svcat binding

```
svcat provision petclinic-db --namespace demo --class google-cloudsql-mysql --plan mysql-db-n1-standard-1
svcat bind petclinic-db --namespace demo --secret-name binding-petclinic
```

#### enable Istio Egress to Google Cloud SQL instance:
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cloud-sql
spec:
  hosts:
  - $(kf vcap-services petclinic | jq '."google-cloudsql-mysql"[0].credentials.host')
  location: MESH_EXTERNAL
  ports:
  - number: 3307
    name: mysql
    protocol: TCP
EOF
```

#### create the build-template we need to build a Spring Boot app

Copy the [Cloud Native Buildback template](https://github.com/knative/build-templates/blob/master/buildpacks/cnb.yaml) from the Knative Build project and change the name to be `buildpack`

Then apply it:

```
kubectl apply -f cnb.yaml
```

#### checkout and prepare the app

Checkout the [Spring PetClinic sample app](https://github.com/spring-projects/spring-petclinic) and add the following dependencies to the `pom.xml`

Add the following dependencies:

```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-gcp-starter</artifactId>
      <version>1.1.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-gcp-starter-sql-mysql</artifactId>
      <version>1.1.1.RELEASE</version>
    </dependency>
```
 or checkout the [kf branch](https://github.com/trisberg/spring-petclinic/tree/kf) of this [fork](https://github.com/trisberg/spring-petclinic/)

#### deploy the app:

```
kf push petclinic --container-registry gcr.io/$(gcloud config get-value core/project) \
 -e SPRING_PROFILES_ACTIVE="cloud" \
 -e SPRING_CLOUD_GCP_SQL_ENABLED='true' \
 -e VCAP_APPLICATION='{"instance_index":0,"name":"petclinic"}' \
 -e VCAP_SERVICES="$(kf vcap-services petclinic)" \
 -e SPRING_CLOUD_GCP_SQL_ENABLED='true' \
 -e SPRING_CLOUD_GCP_SQL_DATABASE_NAME='${vcap.services.kf-binding-petclinic-mydb.database_name}' \
 -e SPRING_CLOUD_GCP_SQL_INSTANCE_CONNECTION_NAME='${vcap.services.kf-binding-petclinic-mydb.ProjectId}:${vcap.services.kf-binding-petclinic-mydb.region}:${vcap.services.kf-binding-petclinic-mydb.instance_name}' \
 -e SPRING_CLOUD_GCP_PROJECT_ID='${vcap.services.kf-binding-petclinic-mydb.ProjectId}'
```
