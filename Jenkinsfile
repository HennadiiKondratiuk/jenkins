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
                        docker.withRegistry('https://319448237430.dkr.ecr.eu-central-1.amazonaws.com', 'ecr:eu-central-1:ecr credential') {
                            app.push("${env.BUILD_ID}"}
                            //p.push("latest")
                    }
                }
            }
        }
    }
}   
