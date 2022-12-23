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
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "aws", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
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
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "aws", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script{
                    def deprun = '''
                        env  && \
                        aws ecr get-login-password --region eu-central-1 && \
                        eval ${docker pull 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID} && \
                        eval ${docker run -d 319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID}
                        '''
                        sshagent(credentials: ['Jenkins_agent_ssh_key']) {
                            sh "ssh ubuntu@10.0.1.116 '${deprun}' "
                        }
                    }
                }
            }
        }
    }
}
