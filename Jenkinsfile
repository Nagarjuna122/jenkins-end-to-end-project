pipeline {

    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '266087049963'
        ECR_REPOSITORY = 'java-cicd-demo'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        EKS_CLUSTER = 'java-cicd-cluster'
        K8S_DEPLOYMENT = 'java-cicd-demo'
        K8S_CONTAINER = 'java-cicd-demo'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build \
                    -t ${IMAGE_NAME} .
                """
            }
        }

        stage('ECR Login') {
            steps {
                sh """
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    docker push ${IMAGE_NAME}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${EKS_CLUSTER}

                    kubectl set image deployment/${k8s_DEPLOYMENT} \
                    ${K8S_CONTAINER}=${IMAGE_NAME}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    kubectl rollout status \
                    deployment/${k8s_DEPLOYMENT} \
                    --timeout=180s
                """
            }
        }
    }

    post {

        success {
            echo "=========================================="
            echo "CI/CD PIPELINE SUCCESSFUL"
            echo "Image: ${IMAGE_NAME}"
            echo "EKS Cluster: ${EKS_CLUSTER}"
            echo "=========================================="
        }

        failure {
            echo "=========================================="
            echo "CI/CD PIPELINE FAILED"
            echo "=========================================="
        }
    }
}