def getScriptOutput(content) {
    def output = bat(returnStdout: true, script: content)
    return output.substring(output.indexOf(content) + content.length()).trim()
}

def getLatestReleseVersion() {
    def tag = getScriptOutput('git tag -l v*.*.*').split()[-1]
    def ret = []
    for (String v in tag.substring(1).split("\\.")) {
        ret.add(Integer.parseInt(v))
    }
    return ret
}

def tagExistsForCurrentHead() {
    def output = getScriptOutput('git tag --points-at HEAD')
    if (output) {
        return true
    } else {
        return false
    }
}

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
        branch "main"
        not {changeRequest()}
      }
      steps {
        script {
            if (!tagExistsForCurrentHead()) {
                def message = bat(returnStdout: true, script: 'git log -1').split('\n')[-1].trim()
                def version = getLatestReleseVersion()
                if (message.indexOf('MAJOR') == 0) {
                    version[0]++
                } else if (message.indexOf('MINOR') == 0) {
                    version[1]++
                } else if (message.indexOf('PATCH') == 0) {
                    version[2]++
                }
                    
                if (version != getLatestReleseVersion()) {
                    def newTag = "v${version[0]}.${version[1]}.${version[2]}"
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'holly-shit', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                            bat("git tag ${newTag}")
                            bat("git push https://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@${GIT_URL.substring(8)} --tags")
                    }
                }
            }
        }
      }
    }
  }
  
  post {
      always {
          updateGithubCommitStatus()
      }
  }
}
