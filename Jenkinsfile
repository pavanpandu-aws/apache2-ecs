pipeline {
    agent any

    environment {
        ECR_REGISTRY = "634441478571.dkr.ecr.us-east-1.amazonaws.com"
        ECR_REPOSITORY = "action"
        AWS_REGION = "us-east-1"
        ECS_CLUSTER = "demo_cluster"
        ECS_SERVICE = "demo_service"
        ECS_TASK_DEFINITION = "demo_task_definition"
        IMAGE_TAG = "latest" // change this to your desired tag
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh "docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: 'my-aws-credentials']]) {
                    script {
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY"
                        sh "docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
                        sh "docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
                    }
                }
            }
        }

        stage('Update ECS Service') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: 'my-aws-credentials']]) {
                    script {
                        def latestImage = sh(returnStdout: true, script: "aws ecr describe-images --repository-name $ECR_REPOSITORY --region $AWS_REGION --query 'sort_by(imageDetails,& imagePushedAt)[-1].imageTags[0]' --output text").trim()
                        def TASK_DEFINITION = sh(returnStdout: true, script: "aws ecs describe-task-definition --task-definition $ECS_TASK_DEFINITION --region $AWS_REGION")
                        sh "aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment --region $AWS_REGION"
                    }
                }
            }
        }
    }
}
