pipeline {
  agent any
  stages {
    stage('build') {
      when {
        anyOf {
          branch "main*"; 
          changeRequest()
        }
      steps {
        echo 'It\'s main or pull request!'
      }
    }
  }
}
