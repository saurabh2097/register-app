pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    stages {
        stage("CleanUp Workspace") {
            steps {
                cleanWs()  // ✅ Corrected function name (was "CleanWS()")
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
                sh 'mvn test'  // ✅ Fixed command (was 'mvn tests')
            }
        }
    }
}
