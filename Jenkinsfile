pipeline {
    agent any

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'cd api-service && docker build -t api-service:v1 .'
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/api-deployment.yaml'
                sh 'kubectl apply -f k8s/api-service.yaml'
		sh 'kubectl apply -f k8s/api-ingress.yaml'
                sh 'kubectl rollout restart deployment api-deployment'
		
            }
        }

    }
}
