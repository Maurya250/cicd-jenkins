pipeline {
    agent any

    environment {
        // Docker Hub username
        DOCKERHUB_USERNAME = "maurya250"
        IMAGE_NAME = "patho"
        IMAGE_TAG = "${BUILD_NUMBER}"   // unique tag for every build (1,2,3...)
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage("Checkout Code") {
            // Stage 1: Fetch code from GitHub
            steps {
                // Using Pipeline from SCM, Jenkins auto checkouts the code
                echo "Code checkout done automatically via SCM"
            }
        }

        // Stage 2: OWASP Dependency Check & Trivy File System Scan
        stage("Security Scans (FS & Dependencies)") {
            parallel {

                stage("OWASP Security Scan") {
                    steps {
                        echo "Scanning project dependencies for vulnerabilities..."
                        // 'DP-Check' is the tool name configured in Jenkins
                        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    }
                }

                stage("Trivy FS Scan") {
                    steps {
                        echo "Running Trivy file system scan..."
                        // Scans the entire project directory for vulnerabilities/secrets
                        sh "trivy fs . > trivy-fs-report.txt"
                    }
                }
            }
        }

        // Stage 3: SonarQube Code Quality Analysis
        stage("SonarQube Analysis") {
            steps {
                // 'sonar-server' is the SonarQube server name in Jenkins config
                withSonarQubeEnv('sonar-server') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=my-webapp -Dsonar.sources=."
                }
            }
        }

        // Stage 4: Docker Image Build
        stage("Build Docker Image") {
            steps {
                echo "Building Docker image..."
                // Image format: dockerhub_username/image_name:tag
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                // Also create a 'latest' tag for easy deployment
                sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
            }
        }

        // Stage 5: Trivy Image Scan
        stage("Trivy Image Scan") {
            steps {
                echo "Scanning Docker image for vulnerabilities..."
                // You can fail the pipeline using --exit-code 1 for CRITICAL issues
                sh "trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} > trivy-image-report.txt"
            }
        }

        // Stage 6: Push Image to Docker Hub
        stage("Push to Docker Hub") {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    // Using Jenkins stored credentials
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
        }

        // Stage 7: Deploy Container
        stage("Deploy Container") {
            steps {
                script {
                    echo "Deploying application container..."
                    // Remove old container if it exists
                    sh "docker rm -f pytho-container || true"
                    // Run new container using Docker Hub image
                    sh "docker run -d -p 80:80 --name pytho-container ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }

    // Post-build cleanup and reports
    post {
        always {
            // Publish OWASP Dependency Check report
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            // Archive Trivy scan reports
            archiveArtifacts artifacts: 'trivy-fs-report.txt, trivy-image-report.txt',
                             allowEmptyArchive: true
            sh "docker logout || true"
        }
    }
}
