pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
            yaml '''
            spec:
                containers:
                    - name: earthly
                      image: earthly/earthly
                      env:
                      - name: NO_BUILDKIT
                        value: 1
'''
        }
    }
    environment {
        EARTHLY_TOKEN = credentials('EARTHLY_TOKEN')
    }
    stages {
        stage('Install Earthly') {
            steps {
                sh 'earthly --sat=lamport --org=sjerred --ci --push +ci'
            }
        }
    }
}
