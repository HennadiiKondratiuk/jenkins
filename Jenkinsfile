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
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "aws", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        def deprun = '''echo "aws_access_key_id = $AWS_ACCESS_KEY_ID" > ~/.aws/credentials && echo "aws_secret_access_key = $AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials && \
                            
                            aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin https://319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images && \
                            docker pull 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID  && \
                            docker run -d 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID && \
                            rm -rf ~/.aws/credentials" '''
                        sshagent(credentials: ['Jenkins_agent_ssh_key']) {
                            sh "ssh ubuntu@10.0.1.116 '${deprun}' 
                        }
                    }
                }
            }
        }
    }
}
