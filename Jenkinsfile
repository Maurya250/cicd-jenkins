pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "maurya250"
        IMAGE_NAME = "patho"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("Build Docker Image") {
            steps {
                echo "Building Docker Image...."

                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                script {
                    echo "Pushing to Docker Hub..."

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage("Deploy Container") {
            steps {
                script {
                    echo "Deploying Application...."

                    sh "docker rm -f patho-container || true"
                    sh "docker run -d -p 80:80 --name patho-container ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            sh "docker logout || true"
        }
    }
}
