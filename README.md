# cloudsql-springboot-gke
Containerized Spring boot with cloud sql on GKE.

 
export GOOGLE_APPLICATION_CREDENTIALS=key.json
* here key.json is the service account key

```
gcloud sql instances create --gce-zone us-east1-b --database-version POSTGRES_9_6 --memory 4 --cpu 2 test-postgres
 ```
```
gcloud sql instances list
```

## output:
NAME           DATABASE_VERSION  LOCATION    TIER              PRIMARY_ADDRESS  PRIVATE_ADDRESS  STATUS
test-postgres  POSTGRES_9_6      us-east1-b  db-custom-1-3840  35.227.66.225    -                RUNNABLE

```
gcloud sql databases create dikshith-test --instance=test-postgres
```

## output:

Creating Cloud SQL database...done.                                                                                                                                                                        
Created database [dikshith-test].
instance: test-postgres
name: dikshith-test
project: gcp-dataproc-dikshith


# Authorize to use gcr
```
gcloud auth configure-docker
```

# Creates the below file
$HOME/.docker/config.json


# Creating docker image of spring boot and push to gcr
```
mvn clean compile com.google.cloud.tools:jib-maven-plugin:build -Dimage=gcr.io/gcp-dataproc-dikshith/spring-boot-postgres-gke-app:v2
```

# Testing the docker image locally
```
docker run -ti --rm -p 8080:8080 gcr.io/$GCP_PROJECT_ID/spring-boot-postgres-gke-app:v2
```

# Creating kubernetes UI dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
```

# Creating secrets for cloud sql proxy side car

```
kubectl create secret generic cloudsql-instance-credentials --from-file=credentials.json=key.json

kubectl create secret generic cloudsql-db-credentials \
    --from-literal=username=postgres \
    --from-literal=password=postgres
```

