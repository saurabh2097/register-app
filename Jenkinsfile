pipeline{
  agent { label 'Jenkins-Agent'}
  tools{
    jdk 'Java17'
    maven 'Maven3'
  }
  stages{
    stage("CleanUp Workspace"){
      steps {
        CleanWS()
      }
    }
    stage("Code"){
      steps {
          git branch: 'main', credentialsId: 'github', url: 'https://github.com/saurabh2097/register-app.git'
      }
    }
    stage("build Application"){
      steps {
        sh 'mvn clean package'
      }
    }
    stage("Test APllication"){
      steps {
        sh 'mvn tests'
      }
    }
    
}
