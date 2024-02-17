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
                      - name: TAILSCALE_AUTH_KEY
                        value: $credentials('TAILSCALE_AUTH_KEY')
                    - name: earthly
                      image: earthly/earthly
                      env:
                      - name: NO_BUILDKIT
                        value: 1
                      command: ["sleep"]
                      args: ["1h"]
'''
        }
    }
    environment {
        EARTHLY_TOKEN = $credentials('EARTHLY_TOKEN')
    }
    stages {
        stage('Install Earthly') {
            steps {
                sh 'earthly --sat=lamport --org=sjerred --ci --push +ci'
            }
        }
    }
}
