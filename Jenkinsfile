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
    stages {
        stage('Install Earthly') {
            steps {
                sh 'earthly'
            }
        }
    }
}
