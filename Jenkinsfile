pipeline{
    agent any

    environment{
        REPO = 'narssss1234'
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
                    sh "echo progressing"
                }
            }
        }

        stage('update the EKS cluster'){
            steps{
                script{
                    echo 'workinprogress'
                }
            }
        }
    }
}