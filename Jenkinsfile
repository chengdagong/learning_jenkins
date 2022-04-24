def updateGithubCommitStatus() {
  step([
    $class: 'GitHubCommitStatusSetter',
    reposSource: [$class: "ManuallyEnteredRepositorySource", url: env.GIT_URL],
    commitShaSource: [$class: "ManuallyEnteredShaSource", sha: env.GIT_COMMIT],
    contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins"],
    errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
    statusResultSource: [
      $class: 'ConditionalStatusResultSource',
      results: [
        [$class: 'BetterThanOrEqualBuildResult', result: 'SUCCESS', state: 'SUCCESS', message: 'This commit looks good'],
        [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: env.FAILURE_REASON],
        [$class: 'AnyBuildResult', state: 'FAILURE', message: 'Loophole']
      ]
    ]
  ])
}

pipeline {
  agent any
  environment {
      CODE_UPDATE_PR_TITLE = "(MAJOR|MINOR|PATCH) - .*"
      BUILD_CONFIG_UPDATE_PR_TITLE = "CONFIGUPDATE - .+"
      FAILURE_REASON = 'The title must follow format [MAJOR/MINOR/PATCH] - [summary] or CONFIGUPDATE - [summary]'
  }
  stages {
    stage('validate-title') {
      when {
        changeRequest target: 'main'
      }
      steps {
        script {
          if (!(pullRequest.title=~CODE_UPDATE_PR_TITLE || pullRequest.title=~BUILD_CONFIG_UPDATE_PR_TITLE)) {
            error env.FAILURE_REASON
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
          updateGithubCommitStatus()
      }
  }
}
