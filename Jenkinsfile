pipeline {

    agent any

    environment {

        DOCKER_HUB = "sakshi373"

        API_IMAGE = "api-service"

        BACKEND_IMAGE = "backend-service"

        IMAGE_TAG = "${BUILD_NUMBER}"

        BASE_IMAGE = "ibmcom/ace:latest"
    }

    stages {

        stage('Checkout Code') {

            steps {

                git 'https://github.com/SakshiBonde-Eidiko/ace-enterprise-project.git'
            }
        }

        stage('Build API Image') {

            steps {

                sh '''
                cd api-service

                docker build \
                --build-arg BASE_IMAGE=$BASE_IMAGE \
                -t $DOCKER_HUB/$API_IMAGE:$IMAGE_TAG .
                '''
            }
        }

        stage('Build Backend Image') {

            steps {

                sh '''
                cd backend-service

                docker build \
                --build-arg BASE_IMAGE=$BASE_IMAGE \
                -t $DOCKER_HUB/$BACKEND_IMAGE:$IMAGE_TAG .
                '''
            }
        }

        stage('Push API Image') {

            steps {

                sh '''
                docker push $DOCKER_HUB/$API_IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('Push Backend Image') {

            steps {

                sh '''
                docker push $DOCKER_HUB/$BACKEND_IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('Update Kubernetes YAML') {

            steps {

                sh '''
                sed -i "s|API_IMAGE_PLACEHOLDER|$DOCKER_HUB/$API_IMAGE:$IMAGE_TAG|g" k8s/api-deployment.yaml

                sed -i "s|BACKEND_IMAGE_PLACEHOLDER|$DOCKER_HUB/$BACKEND_IMAGE:$IMAGE_TAG|g" k8s/backend-deployment.yaml
                '''
            }
        }

        stage('Deploy ConfigMap') {

            steps {

                sh '''
                kubectl apply -f k8s/ace-configmap.yaml
                '''
            }
        }

        stage('Deploy API') {

            steps {

                sh '''
                kubectl apply -f k8s/api-deployment.yaml

                kubectl apply -f k8s/api-service.yaml
                '''
            }
        }

        stage('Deploy Backend') {

            steps {

                sh '''
                kubectl apply -f k8s/backend-deployment.yaml

                kubectl apply -f k8s/backend-service.yaml
                '''
            }
        }

        stage('Restart Deployments') {

            steps {

                sh '''
                kubectl rollout restart deployment api-deployment

                kubectl rollout restart deployment backend-deployment
                '''
            }
        }
    }
}
