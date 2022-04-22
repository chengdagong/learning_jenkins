pipeline {
  agent any
  environment {
    CODE_UPDATE_PR_TITLE = "v\\d+\\.\\d+\\.\\d+ - .+"
    BUILD_CONFIG_UPDATE_PR_TITLE = "CONFIGUPDATE - .+"
  }
  stages {
    stage('validate-title') {
      when {
        changeRequest target: 'main'
      }
      steps {
        script {
          def status = 'success'
          def description = 'Succeeded'
          if (!(pullRequest.title=~CODE_UPDATE_PR_TITLE || pullRequest.title=~BUILD_CONFIG_UPDATE_PR_TITLE)) {
            status = 'failure'
            description = 'The title must follow format v[major].[minor].[patch] - [summary] or CONFIGUPDATE - [summary]'
          } 

          pullRequest.createStatus(status: status,
                           context: 'ci/jenkins/pr-merge/validate-title',
                           description: description,
                           targetUrl: "${env.RUN_DISPLAY_URL}")
          
          if (status == 'failure') {
            error description
          }
        }
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
