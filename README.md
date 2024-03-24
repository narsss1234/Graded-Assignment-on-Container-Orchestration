### Learner Report CS

## MERN Stack application

## Deployments - MongoDB, Backend, Frontend

Solution:

### Mongo DB deployment

1. Created a folder named -

```
mkdir learnerReportCS-kube-files
```

2. Created deployment and service files for the mongo

```
touch mongo.yaml
touch mongo-svc.yaml
```

3. Update deployment files

```
Replicas - 1 is enough as this is a DB
expose the containerPort 27017 - as per mongo image official documentation
```

4. Update service files

```
ClusterIP should be the service type, as we do not want to expose this pod
```

### Backend Deployment

1. Edit the config.env file

# Change the atlas url
```
ATLAS_URI="mongodb://mongodb.default.svc.cluster.local/blog_mern"
```
2. Change the port to 4200

Navigate to index.js

```
const port = process.env.PORT || 4200;

Either create a .env file and add the env variable as 4200, or change it in the or function here used in Javascript
```

3. Correct the dockerfile

Navigate to Docker file

```
EXPOSE 4200

As the backend will be listening on this, so will the container image
```

4. Build the docker image

```
docker build -t narsss1234/learner-report-cs-backend:latest .
```

5. Push the image to dockerhub

```
docker push narsss1234/learner-report-cs-backend:latest
```

6. change dir to LearnerReportCS-kube-files

```
touch backend.yaml
touch backend-svc.yaml
```

7. Update the deployment

```
Replicas - 1 is enough for now as this is dev environment
Set the image as narsss1234/learner-report-cs-backend:latest
Added imagePullPolicy to Always

```

8. Update the Service

```
Exposed backend service on NodePort 4200

we can test this on the ec2 public url - at port 4200, we will see the backend exposed
```

### Frontend Deployment

1. As per the react-readme.md, the app will run on port 3000 by default


2. Correct the dockerfile

Navigate to Docker file

```
EXPOSE 3000

As the frontend will be listening on this, so will the container image
```

3. Build the docker image

```
docker build -t narsss1234/learner-report-cs-frontend:latest .
```

4. Push the image to dockerhub

```
docker push narsss1234/learner-report-cs-frontend:latest
```

5. change dir to LearnerReportCS-kube-files

```
touch frontend.yaml
touch frontend-svc.yaml
```

6. Update the deployment

```
Replicas - 1 is enough for now as this is dev environment
Set the image as narsss1234/learner-report-cs-frontend:latest
Added imagePullPolicy to Always

```

7. Update the Service

```
Exposed backend service on Load Balancer

we can test this on the load balancer url - at port 3000, we will see the backend exposed
```


### Creatng Helm charts for the project

1. Create a Helm folder, with all the default files

```
helm create LearnerReportCS-helm
```

2. Copy all the manifest files to templates folder inside the helm folder

3. Create all the values in values.yaml file 

4. Replace the values in manifest files with 

```
{{ .Values.<value> }}
```

5. install the helm chart

```
helm install learner-report-cs-app LearnerReportCS-helm --values LearnerReportCS-helm/values.yaml
```

6. This will deploy the Stack with versioning.


### Stich this all and automate using Jenkins

