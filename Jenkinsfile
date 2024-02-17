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
        stage('Build') {
            steps {
                sh 'earthly --sat=lamport --org=sjerred --ci --push +ci'
            }
        }
    }
}
