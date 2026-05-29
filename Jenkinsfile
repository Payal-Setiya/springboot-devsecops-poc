pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "payalsetiya/devsecops-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        SONAR_URL = "http://16.171.14.22:9000"
        NEXUS_URL = "http://51.21.167.105:8081"
        NEXUS_REPO = "maven-releases"
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

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=devsecops-app \
                    -Dsonar.projectName=devsecops-app \
                    -Dsonar.host.url=$SONAR_URL \
                    -Dsonar.token=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs . --exit-code 0 --severity HIGH,CRITICAL'
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                    JAR_FILE=\$(ls target/*.jar | head -1)

                    curl -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file \$JAR_FILE \
                    $NEXUS_URL/repository/$NEXUS_REPO/com/example/devsecops-app/${BUILD_NUMBER}/devsecops-app-${BUILD_NUMBER}.jar
                    """
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image $DOCKER_IMAGE:$IMAGE_TAG --exit-code 0 --severity HIGH,CRITICAL'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                sed -i 's|DOCKER_IMAGE|$DOCKER_IMAGE:$IMAGE_TAG|g' k8s/deployment.yaml
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl rollout status deployment/devsecops-app
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
