pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "maurya250"
        IMAGE_NAME = "patho"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage("Checkout Code") {
            steps {
                echo "Code checkout done automatically via SCM"
            }
        }

        stage("Security Scans (FS & Dependencies)") {
            parallel {

                stage("OWASP Security Scan") {
                    steps {
                        echo "Scanning project dependencies for vulnerabilities..."
                        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                                        odcInstallation: 'DP-Check'
                    }
                }

                stage("Trivy FS Scan") {
                    steps {
                        echo "Running Trivy file system scan..."
                        sh "trivy fs . > trivy-fs-report.txt"
                    }
                }
            }
        }

        stage("SonarQube Analysis") {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=my-webapp \
                    -Dsonar.sources=. \
                    -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
            }
        }

        stage("Trivy Image Scan") {
            steps {
                echo "Scanning Docker image for vulnerabilities..."
                sh "trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} > trivy-image-report.txt"
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Deploy Container") {
            steps {
                echo "Deploying application container..."
                sh "docker rm -f pytho-container || true"
                sh "docker run -d -p 80:80 --name pytho-container ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            archiveArtifacts artifacts: 'trivy-fs-report.txt, trivy-image-report.txt',
                             allowEmptyArchive: true
            sh "docker logout || true"
        }
    }
}
