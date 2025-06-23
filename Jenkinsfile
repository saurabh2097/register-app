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
        /*DOCKER_PASS = "dockerhub"*/
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}" + "-" + "${BUILD_NUMBER}"
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
        stage("Build & Tag Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: env.DOCKER_PASS, toolName: 'docker') {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
    }
}

        }
    }
}

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId:  env.DOCKER_PASS ,  toolName: 'docker') {
                        sh "docker push ${IMAGE_NAME}:latest"
            }
        }
    }
}



    }
}
