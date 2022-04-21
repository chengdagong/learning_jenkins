pipeline {
  agent none
  stages {
    stage('pre-build') {
      when {
        changeRequest target: "master"
        not {changeRequest title: "v\d+\.\d+\.\d+ - .+", comparator: 'REGEXP'}
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
