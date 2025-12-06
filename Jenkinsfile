pipeline {
    agent any

    environment {
        AWS_REGION    = "ap-south-1"           // change if needed
        ECR_REPO_NAME = "simple-time-service" // ECR repo name
        // AWS_ACCOUNT_ID and ECR_REPO_URI will be set dynamically
    }

    stages {
        stage('Checkout') {
            steps {
                // Clones the same repo where Jenkinsfile is stored
                checkout scm
            }
        }

        stage('Resolve AWS Account & ECR URI') {
            steps {
                script {
                    // Get AWS account ID from instance role
                    def accountId = sh(
                        script: "aws sts get-caller-identity --query Account --output text",
                        returnStdout: true
                    ).trim()

                    env.AWS_ACCOUNT_ID = accountId
                    env.ECR_REPO_URI = "${accountId}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.ECR_REPO_NAME}"

                    echo "Using AWS account: ${env.AWS_ACCOUNT_ID}"
                    echo "Using ECR repo URI: ${env.ECR_REPO_URI}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh """
                      echo "Building Docker image..."
                      docker build -t ${ECR_REPO_NAME}:latest .
                    """
                }
            }
        }

        stage('Login to ECR & Push Image') {
            steps {
                sh """
                  echo "Ensuring ECR repository exists (or creating it)..."
                  aws ecr describe-repositories \
                    --repository-names ${ECR_REPO_NAME} \
                    --region ${AWS_REGION} >/dev/null 2>&1 || \
                  aws ecr create-repository \
                    --repository-name ${ECR_REPO_NAME} \
                    --region ${AWS_REGION}

                  echo "Logging in to ECR using instance role..."
                  aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                  echo "Tagging image for ECR..."
                  docker tag ${ECR_REPO_NAME}:latest ${ECR_REPO_URI}:latest

                  echo "Pushing image to ECR..."
                  docker push ${ECR_REPO_URI}:latest
                """
            }
        }

        stage('Terraform Init/Plan/Apply') {
            steps {
                dir('terraform') {
                    sh """
                      echo "Initializing Terraform..."
                      terraform init

                      echo "Planning Terraform..."
                      terraform plan -var="container_image=${ECR_REPO_URI}:latest"

                      echo "Applying Terraform..."
                      terraform apply -auto-approve -var="container_image=${ECR_REPO_URI}:latest"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful!"
            echo "➡ Now run 'terraform output alb_dns_name' in the terraform directory to get the URL."
        }
        failure {
            echo "Pipeline failed. Check the stage logs in Jenkins."
        }
    }
}
