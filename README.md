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
touch frontend.yaml
touch frontend-svc.yaml
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

