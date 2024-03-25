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
                git branch: 'main', url: 'https://github.com/CharismaticOwl/Graded-Assignment-on-Container-Orchestration.git'
            }
        }

        stage('Build the Frontend'){
            steps{
                script{
                    docker.build(${env.REPO}/${env.FRONTEND}, "-f learnerReportCS_frontend .")
                }
            }
        }

        stage('Build the Backend'){
            steps{
                script{
                    docker.build(${env.REPO}/${env.BACKEND}, "-f learnerReportCS_backend .")
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