pipeline {
  agent any
  stages {
    stage('pre-build') {
      when {
        changeRequest target: 'main'
        expression {!(pullRequest.title=~"v\\d+\\.\\d+\\.\\d+ - .+")}
      }
      steps {
        error 'The title must be following such format: v[major].[minor].[patch] - [comment]'
      }
    }
    
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
