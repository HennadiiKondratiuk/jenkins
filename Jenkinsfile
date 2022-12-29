pipeline {
    agent any
    environment {
        AWS_ECR_REPO = 'https://319448237430.dkr.ecr.eu-central-1.amazonaws.com/hkondratiuk-images'
        AWS_ECR_REGION = 'eu-central-1'
        AWS_ECS_SERVICE = 'nginx_service'
        AWS_ECS_CLUSTER = 'hkondratiuk-ecs-cluster'
        AWS_ECS_TASK_DEFINITION = 'task_def.json'
        AWS_ECS_TASK_DEF_NAME = 'nginx_family'
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
                       aws ecr get-login-password --region ${AWS_ECR_REGION} | docker login --username AWS --password-stdin ${AWS_ECR_REPO}
                       docker tag my-image:$BUILD_ID ${AWS_ECR_REPO}:$BUILD_ID
                       docker push ${AWS_ECR_REPO}:$BUILD_ID
                    '''     
                }
            }
        }
        stage('Pull and Run') {
            steps {
                script{
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "aws", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        
                        sh "jq '.containerDefinitions[0].image= \"${AWS_ECR_REPO}:$BUILD_ID\"' ${AWS_ECS_TASK_DEFINITION} > tmp.json && mv tmp.json ${AWS_ECS_TASK_DEFINITION}"
                        //sh("aws ecs register-task-definition --region ${AWS_ECR_REGION} --family ${AWS_ECS_TASK_DEFINITION} --execution-role-arn ${AWS_ECS_EXECUTION_ROL} --requires-compatibilities ${AWS_ECS_COMPATIBILITY} --network-mode ${AWS_ECS_NETWORK_MODE} --cpu ${AWS_ECS_CPU} --memory ${AWS_ECS_MEMORY} --container-definitions file://${AWS_ECS_TASK_DEFINITION_PATH}")
                        sh "aws ecs register-task-definition --cli-input-json file://${AWS_ECS_TASK_DEFINITION}"
                        def Revision = sh(script: "aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEF_NAME} | egrep \"revision\" | sed -e 's/\"//' -e 's/,//' | awk '{print \$2}'", returnStdout: true)
                        sh "aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEF_NAME}:${Revision}"
                        
                    }
                }
            }
        }
    }
}
