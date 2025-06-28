pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "saurabh021"
        DOCKER_PASS = "dockerhub" // Jenkins में ये credential ID होना चाहिए
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}-${RELEASE}"
    }

    stages {
        stage("CleanUp Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Code") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/saurabh2097/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("Test Application") {
            steps {
                sh 'mvn test'
            }
        }

        stage("Sonarqube-Analysis") {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
            }
        }

        stage("OWASP") {
            steps {
                withCredentials([string(credentialsId: 'ndv-key', variable: 'NVD_KEY')]) {
                    dependencyCheck additionalArguments: "--scan ./ --disableYarnAudit --disableNodeAudit -Dnvd.api.key=${NVD_KEY}", odcInstallation: 'DP-check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage("Build & Tag Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: env.DOCKER_PASS, toolName: 'docker') {
                        sh "docker build -t ${IMAGE_NAME}:latest ."
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: env.DOCKER_PASS, toolName: 'docker') {
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage("Scan Docker Image with Trivy") {
            steps {
                script {
                    echo "🔍 Running Trivy scan on ${IMAGE_NAME}:latest"
                    def status = sh(
                        script: "trivy image ${IMAGE_NAME}:latest > trivy-image-scan.txt",
                        returnStatus: true
                    )
                    if (status != 0) {
                        echo "⚠️ Trivy scan failed or found issues. Exit code: ${status}"
                    } else {
                        echo "✅ Trivy scan completed successfully."
                    }
                }
                archiveArtifacts artifacts: 'trivy-image-scan.txt', onlyIfSuccessful: false
            }
        }

        stage("CleanUp Artifact") {
            steps {
                script {
                    echo " Removing Docker image"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }
    }

    post {
        always {
            echo " Pipeline finished with status: ${currentBuild.currentResult}"
        }
        failure {
            echo " One or more stages failed. Check logs above."
        }
    }
}
