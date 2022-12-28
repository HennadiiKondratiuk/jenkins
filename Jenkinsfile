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
                    //NEW_ECR_IMAGE=${319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID} copy the image URI
                    //TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition nginx_family --region="eu-central-1")
                    sh '''
                       aws ecs describe-task-definition --task-definition nginx_family --region="eu-central-1" | jq ‘.containerDefinitions[0].image=’\"319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID\" \ > task-def.json
                    
                       aws ecs register-task-definition --cli-input-json file://task_def.json
                       aws ecs update-service --cluster hkondratiuk-ecs-cluster --service nginx_service --task-definition nginx_family:3
                    '''
                }
            }
        }
    }
}
