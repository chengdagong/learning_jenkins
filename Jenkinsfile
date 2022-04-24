def updateGithubCommitStatus(context, failureReason) {
  // workaround https://issues.jenkins-ci.org/browse/JENKINS-38674
  step([
    $class: 'GitHubCommitStatusSetter',
    reposSource: [$class: "ManuallyEnteredRepositorySource", url: env.GIT_URL],
    commitShaSource: [$class: "ManuallyEnteredShaSource", sha: env.GIT_COMMIT],
    contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/" + context],
    errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
    statusResultSource: [
      $class: 'ConditionalStatusResultSource',
      results: [
        [$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: 'This commit looks good'],
        [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: failureReason],
        [$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
      ]
    ]
  ])
}

pipeline {
  agent any
  environment {
    CODE_UPDATE_PR_TITLE = "v\\d+\\.\\d+\\.\\d+ - .+"
    BUILD_CONFIG_UPDATE_PR_TITLE = "CONFIGUPDATE - .+"
    FAILURE_REASON = 'The title should be in format ...'
  }
  stages {
    stage('validate-title') {
      when {
        changeRequest target: 'main'
      }
      steps {
        bat "set"
        script {
          if (!(pullRequest.title=~CODE_UPDATE_PR_TITLE || pullRequest.title=~BUILD_CONFIG_UPDATE_PR_TITLE)) {
            error "PR title not valid"
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
  
  post {
      always {
          updateGithubCommitStatus('pr-merge', env.FAILURE_REASON)
      }
  }
}
