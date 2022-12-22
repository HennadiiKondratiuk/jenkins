pipeline {
    agent any
    stages {
        stage('Build') { 
            steps { 
                script{
                 app = docker.build("my-image:${env.BUILD_ID}")
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                       sh '''
                       aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin https://319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images
                       docker tag my-image:$BUILD_ID 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID
                       docker push 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID
                       '''     
                }
            }
        }
        stage('Pull and Run') {
            steps {
                script{
                    sshagent(credentials: ['Jenkins_agent_ssh_key']) {
                        sh '''
                            aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
                            docker pull 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID
                            docker run -d 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID
                        '''
                    }
                }
            }
        }
    }
}
