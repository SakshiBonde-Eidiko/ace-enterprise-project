pipeline {

    agent any

    environment {

        IMAGE_NAME = "api-service"
        IMAGE_TAG = "${BUILD_NUMBER}"
        BASE_IMAGE = "ibmcom/ace:latest"
    }

    stages {

        stage('Checkout Code') {

            steps {

                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Build Docker Image') {

            steps {

                echo 'Building ACE Docker image...'

                sh '''
                cd api-service

                docker build \
                --build-arg BASE_IMAGE=$BASE_IMAGE \
                -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Verify Docker Image') {

            steps {

                echo 'Verifying Docker image...'

                sh '''
                docker images
                '''
            }
        }

        stage('Update Kubernetes Deployment') {

            steps {

                echo 'Updating Kubernetes deployment YAML...'

                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$IMAGE_NAME:$IMAGE_TAG|g" \
                k8s/api-deployment.yaml
                '''
            }
        }

        stage('Deploy ConfigMap') {

            steps {

                echo 'Deploying ConfigMap...'

                sh '''
                kubectl apply -f k8s/ace-configmap.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {

            steps {

                echo 'Deploying application to Kubernetes...'

                sh '''
                kubectl apply -f k8s/api-deployment.yaml
                kubectl apply -f k8s/api-service.yaml
                '''
            }
        }

        stage('Restart Deployment') {

            steps {

                echo 'Restarting Kubernetes deployment...'

                sh '''
                kubectl rollout restart deployment api-deployment
                '''
            }
        }

        stage('Verify Deployment') {

            steps {

                echo 'Verifying Kubernetes deployment...'

                sh '''
                kubectl get pods
                kubectl get svc
                kubectl get deployment
                '''
            }
        }

        stage('Health Check') {

            steps {

                echo 'Checking service URL...'

                sh '''
                minikube service api-service --url
                '''
            }
        }
    }

    post {

        success {

            echo 'CI/CD Pipeline Executed Successfully!'
        }

        failure {

            echo 'Pipeline Failed.'
        }

        always {

            echo 'Pipeline Execution Completed.'
        }
    }
}
