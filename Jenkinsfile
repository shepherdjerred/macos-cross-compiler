pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
            yaml '''
            spec:
                containers:
                - name: earthly
                image: earthly/earthly
'''
        }
    }
    stages {
        stage('Install Earthly') {
            container()
            steps {
                sh 'earthly'
            }
        }
    }
}
