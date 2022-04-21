pipeline {
  agent any
  stages {
    stage('build') {
      when {
        anyOf {
          branch "master"; 
          changeRequest()
        }
      }
      steps {
        echo 'It\'s main or pull request!'
      }
    }
  }
}
