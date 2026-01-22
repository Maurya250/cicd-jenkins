pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "maurya250"
        IMAGE_NAME = "patho"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        //  ---------- Security Scan (Optional but Recommended) ---------- 
        stage("Trivy File System Scan") {
            steps {
                echo "Scanning source code for vulnerabilities..."
                sh "trivy fs . > trivy-fs-report.txt || true"
            }
        }

        // ---------- SonarQube Code Quality ---------- 
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=patho-app \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        // ---------- Build Docker Image ---------- 
        stage("Build Docker Image") {
            steps {
                echo "Building Docker Image...."
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
            }
        }

        // ---------- Trivy Image Scan ---------- 
        stage("Trivy Image Scan") {
            steps {
                echo "Scanning Docker Image..."
                sh "trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} > trivy-image-report.txt || true"
            }
        }

        // ---------- Push to Docker Hub ---------- 
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

        // ---------- Deploy Container ---------- 
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
            archiveArtifacts artifacts: 'trivy-fs-report.txt, trivy-image-report.txt', allowEmptyArchive: true
            sh "docker logout || true"
        }
    }
}
