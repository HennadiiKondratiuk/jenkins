pipeline {
    agent any
    environment {
        //POM_VERSION = getVersion()
        //JAR_NAME = getJarName()
        AWS_ECR_REGION = 'eu-central-1'
        //AWS_ECS_SERVICE = 'ch-dev-user-api-service'
        AWS_ECS_TASK_DEFINITION = 'nginx_family'
        AWS_ECS_COMPATIBILITY = 'FARGATE'
        AWS_ECS_NETWORK_MODE = 'awsvpc'
        AWS_ECS_CPU = '256'
        AWS_ECS_MEMORY = '512'
        AWS_ECS_CLUSTER = 'hkondratiuk-ecs-cluster'
        AWS_ECS_TASK_DEFINITION_PATH = './task_def.json'
        AWS_ECS_EXECUTION_ROL = 'arn:aws:iam::319448237430:role/ecsTaskExecutionRole'
    }
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
                        def Revision = sh(script: "aws ecs describe-task-definition --task-definition nginx_family | egrep \"revision\" | sed -e 's/\"//' -e 's/,//' | awk '{print \$2}'", returnStdout: true)
                        sh "aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} --region=${AWS_ECR_REGION} | jq '.taskDefinition.containerDefinitions[0].image= \"319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images:$BUILD_ID\"' > task_def.json"
                        //sh("aws ecs register-task-definition --region ${AWS_ECR_REGION} --family ${AWS_ECS_TASK_DEFINITION} --execution-role-arn ${AWS_ECS_EXECUTION_ROL} --requires-compatibilities ${AWS_ECS_COMPATIBILITY} --network-mode ${AWS_ECS_NETWORK_MODE} --cpu ${AWS_ECS_CPU} --memory ${AWS_ECS_MEMORY} --container-definitions file://${AWS_ECS_TASK_DEFINITION_PATH}")
                        
                        //sh "cat ./task_def.json"
                        sh "aws ecs register-task-definition --cli-input-json file://./task_def.json"
                        sh "aws ecs update-service --cluster hkondratiuk-ecs-cluster --service nginx_service --task-definition nginx_family:${Revision}"
                        
                    }
                }
            }
        }
    }
}
