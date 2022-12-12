pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
         stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
         }

        stage('Build') { 
            steps { 
                script{
                 app = docker.build("my-image:${env.BUILD_ID}")
                }
            }
        }
        stage('Test'){
            steps {
                 echo 'Empty'
            }
        }
        stage('Deploy') {
            steps {
                script{
                       sh 'aws ecr get-login-password --region eu-central-1 | sudo docker login --username AWS --password-stdin https://319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images'
                       sh 'sudo docker tag "my-image:${env.BUILD_ID}" 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images'
                       sh 'docker push https://319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images'
                    
                }
            }
        }
    }
}   
