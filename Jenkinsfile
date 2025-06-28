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
        DOCKER_PASS = "dockerhub" // <-- Credential ID must be valid in Jenkins
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
                    sh """
                        trivy image ${IMAGE_NAME} > trivy-image-scan.txt || true
                        """
                    }
                archiveArtifacts artifacts: 'trivy-image-scan.txt', onlyIfSuccessful: false
                        }
                }


        stage("CleanUp Artifact") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
}
