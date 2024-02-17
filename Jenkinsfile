
def getRepoURL() {
  sh "mkdir -p .git"
  sh "git config --get remote.origin.url > .git/remote-url"
  return readFile(".git/remote-url").trim()
}
def getCommitSha() {
  sh "mkdir -p .git"
  sh "git rev-parse HEAD > .git/current-commit"
  return readFile(".git/current-commit").trim()
}

def updateGithubCommitStatus(build, String context, String buildUrl, String message, String state) {
  // workaround https://issues.jenkins-ci.org/browse/JENKINS-38674
  repoUrl = getRepoURL()
  commitSha = getCommitSha()
  println "Updating Github Commit Status"
  println "repoUrl $repoUrl"
  println "commitSha $commitSha"
  println "build result: ${build.result}, currentResult: ${build.currentResult}"

  step([
    $class: 'GitHubCommitStatusSetter',
    reposSource: [$class: "ManuallyEnteredRepositorySource", url: repoUrl],
    commitShaSource: [$class: "ManuallyEnteredShaSource", sha: commitSha],
    errorHandlers: [[$class: 'ShallowAnyErrorHandler']],
    contextSource: [$class: "ManuallyEnteredCommitContextSource", context: context],
    statusBackrefSource: [$class: "ManuallyEnteredBackrefSource", backref: buildUrl],

    statusResultSource: [
      $class: 'ConditionalStatusResultSource',
      results: [
        [$class: 'AnyBuildResult', state: state, message: message]
      ]
    ]
  ])
}

pipeline {
    agent {
        kubernetes {
            defaultContainer 'earthly'
            inheritFrom 'default'
            yaml '''
            spec:
                containers:
                    - name: tailscale
                      image: tailscale/tailscale
                      securityContext:
                          privileged: true
                      env:
                      - name: TS_AUTHKEY
                        valueFrom:
                          secretKeyRef:
                            name: tailscale-auth-key
                            key: TS_AUTHKEY
                      - name: TS_ACCEPT_DNS
                        value: true
                      - name: TS_KUBE_SECRET
                        value:
                      - name: TS_USERSPACE
                        value: false
                    - name: earthly
                      image: earthly/earthly
                      env:
                      - name: NO_BUILDKIT
                        value: 1
                      - name: NO_COLOR
                        value: 1
                      command: ["sleep"]
                      args: ["1h"]
'''
        }
    }

    environment {
        EARTHLY_TOKEN = credentials('EARTHLY_TOKEN')
    }

    stages {
        stage('Build') {
            steps {
              sh 'earthly --sat=lamport --org=sjerred --ci --push +ci';
            }
        }
    }
}
