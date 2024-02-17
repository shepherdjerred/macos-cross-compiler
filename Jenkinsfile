pipeline {
    agent {
        docker {
            image 'earthly/earthly'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh "earthly --ci --push +ci"
            }
        }
    }
}
