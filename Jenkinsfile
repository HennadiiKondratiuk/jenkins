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
                script{
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "aws", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        def Revision = sh(script: "aws ecs describe-task-definition --task-definition nginx_family | egrep \"revision\" | sed -e 's/\"//' -e 's/,//' | awk '{print {$2}}', returnStdout: true ")
                        sh '''
                          aws ecs describe-task-definition --task-definition nginx_family --region="eu-central-1" | jq '.taskDefinition.containerDefinitions[0].image= "319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID"' > task-def.json
                          aws ecs register-task-definition --cli-input-json file://task_def.json
                          echo ${Revision}
                        '''
                        //sh "aws ecs update-service --cluster hkondratiuk-ecs-cluster --service nginx_service --task-definition nginx_family:${Revision}"
                        
                    }
                }
            }
        }
    }
}
