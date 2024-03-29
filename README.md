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


### Stitch this all and automate using Jenkins

1. Created Jenkinsfile

```
touch Jenkinsfile
```

2. Launched and ec3 instance with userdata to install Jenkins, docker, git, awscli,
also attached an IAM role to the instance so that ec2 can access AWS resources.

```
#!/bin/bash
sudo apt update --fix-missing
sudo apt install awscli git fontconfig openjdk-17-jdk -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

3. Once the ec2 is launched, you can access the jenkins on port 8080 and public IP address for ec2

4. Go to Manage Jenkins, install all the plugins

```
git
docker plugin
AWS Credentials plugin
Kubernetes plugin
AWS :ECR plugin
```

5. Add global credentials

```
AWS Credentials -> to access aws resources
Docker login 
```

5. installed kubectl, eksctl to be able to launch and configure EKS clusters

6. Created a pipeline to be able to automate the build and deployment process, ensuring consistency and efficiency.

7. Stage 1 to fetch the code

```
stage('Fetch the code'){
            steps{
                script{
                    git branch:'main', url: 'https://github.com/CharismaticOwl/Graded-Assignment-on-Container-Orchestration.git'
                }
            }
        }
```

8. Stage 2,3 to build the docker images

```
docker build -t ${env.REPO}/${env.FRONTEND}:${env.TAG} .
```

9. Stage 4, push the code to dockerhub

```
steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                        sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                        sh "docker push ${env.REPO}/${env.FRONTEND}:${env.TAG}"
                        sh "docker push ${env.REPO}/${env.BACKEND}:${env.TAG}"
                    }
                }
            }
```

10. Stage 5, to create a cluster if none exists, and if it already exists then simply update the deployment with helm install

```
steps{
                script{
                    def ClusterStatus = sh(script:"eksctl get cluster --name LearnerReportCSclusterNEW --region ap-south-1", returnStatus:true)
                    if (ClusterStatus != 0){
                        sh "Cluster does not exits creating one."
                        sh "eksctl create cluster --name LearnerReportCSclusterNEW --region ap-south-1"
                        sh "aws eks update-kubeconfig --name LearnerReportCSclusterNEW --region ap-south-1"
                        sh "helm upgrade --install LearnReportCS-app LearnerReportCS-helm"
                    } else{
                        sh "echo 'Cluster exits, moving on with deployment.'"
                        sh "aws eks update-kubeconfig --name LearnerReportCSclusterNEW --region ap-south-1"
                        sh "helm upgrade --install LearnReportCS-app LearnerReportCS-helm"
                    }
                }
            }
```