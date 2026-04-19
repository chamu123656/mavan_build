pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'your-dockerhub-username'
        APP_NAME = 'my-maven-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_USER}/${APP_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_HUB_USER}/${APP_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${APP_NAME}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${APP_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${APP_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Assumes you have a k8s-deployment.yaml in your repo
                withKubeConfig([credentialsId: 'k8s-config-id']) {
                    sh "sed -i 's|REPLACE_WITH_IMAGE|${DOCKER_HUB_USER}/${APP_NAME}:${IMAGE_TAG}|g' k8s-deployment.yaml"
                    sh "kubectl apply -f k8s-deployment.yaml"
                }
            }
        }
    }
}
