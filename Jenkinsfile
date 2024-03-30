pipeline{
    agent any

    environment{
        REPO = 'narsss1234'
        FRONTEND = 'learner-report-cs-frontend'
        BACKEND = 'learner-report-cs-backend'
        TAG = 'latest'
    }

    stages{
        stage('Fetch the code'){
            steps{
                script{
                    git branch:'main', url: 'https://github.com/CharismaticOwl/Graded-Assignment-on-Container-Orchestration.git'
                }
            }
        }

        stage('Build the Frontend'){
            steps{
                script{
                    sh "cd learnerReportCS_frontend && docker build -t ${env.REPO}/${env.FRONTEND}:${env.TAG} ."
                }
            }
        }

        stage('Build the Backend'){
            steps{
                script{
                    sh "cd learnerReportCS_backend && docker build -t ${env.REPO}/${env.BACKEND}:${env.TAG} ."
                }
            }
        }

        stage('Push the image to the public repo'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                        sh "docker login -u $dockerHubUser -p $dockerHubPassword"
                        sh "docker push ${env.REPO}/${env.FRONTEND}:${env.TAG}"
                        sh "docker push ${env.REPO}/${env.BACKEND}:${env.TAG}"
                    }
                }
            }
        }

        stage('Configure AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: 'aws' // Replace with your Jenkins credentials ID
                ]]) {
                    sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                    sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                    sh 'aws configure set region ap-south-1'
                }
            }
        }

        stage('Update the EKS cluster') {
            steps {
                script {
                    def clusterStatus = sh(script: "eksctl get cluster --name LearnerReportCSclusterNEW --region ap-south-1", returnStatus: true)
                    if (clusterStatus != 0) {
                        echo "Cluster does not exist, creating one."
                        sh "eksctl create cluster --name LearnerReportCSclusterNEW --region ap-south-1"
                    } else {
                        echo "Cluster exists, moving on with deployment."
                    }
                    sh "aws eks update-kubeconfig --name LearnerReportCSclusterNEW --region ap-south-1"
                    sh "helm upgrade --install learnreportcs LearnerReportCS-helm"
                }
            }
        }
    }
}
