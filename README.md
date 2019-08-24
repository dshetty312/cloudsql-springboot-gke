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

kubectl create secret generic cloudsql-db-credentials     --from-literal=username=postgres     --from-literal=password=postgres --from-literal=dbname=postgres
```

# Deploy service and deployment

```
kubectl apply -f kube.yml
```

# Get the load balancer ip
```
NAME                           TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)          AGE
kubernetes                     ClusterIP      10.0.0.1     <none>           443/TCP          24m
spring-boot-postgres-gke-app   LoadBalancer   10.0.12.69   35.237.253.163   8080:30457/TCP   12m
```

# Get request on the endpoint
```
dikshithshetty@DIKSHITHs-MacBook-Pro:~/gcp-stuff/postgress-boot$ curl 35.237.253.163:8080/postgressApp/employeeList | json_pp
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   477    0   477    0     0   5735      0 --:--:-- --:--:-- --:--:--  5746
[
   {
      "employeeId" : "1",
      "employeeAddress" : null,
      "employeeName" : "Jack",
      "employeeEmail" : "jack@gmail.com"
   },
   {
      "employeeId" : "2",
      "employeeName" : "DShetty",
      "employeeAddress" : null,
      "employeeEmail" : "ds@gmail.com"
   },
   {
      "employeeEmail" : "sam@gmail.com",
      "employeeId" : "3",
      "employeeAddress" : null,
      "employeeName" : "Sam"
   },
   {
      "employeeEmail" : "mk@gmail.com",
      "employeeId" : "4",
      "employeeAddress" : null,
      "employeeName" : "MK"
   },
   {
      "employeeEmail" : "ad@gmail.com",
      "employeeId" : "5",
      "employeeName" : "AD",
      "employeeAddress" : null
   }
]
```

