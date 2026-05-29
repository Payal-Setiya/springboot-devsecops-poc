pipeline {
    agent any

    environment {
        IMAGE_NAME = "payalsetiya/devsecops-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'githu-ssh-key',
                url: 'git@github.com:Payal-Setiya/springboot-devsecops-poc.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . --exit-code 0 --severity HIGH,CRITICAL'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:$IMAGE_TAG --exit-code 0 --severity HIGH,CRITICAL'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                sed -i 's|DOCKER_IMAGE|$IMAGE_NAME:$IMAGE_TAG|g' k8s/deployment.yaml
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl rollout status deployment/devsecops-app
                """
            }
        }
    }
}
