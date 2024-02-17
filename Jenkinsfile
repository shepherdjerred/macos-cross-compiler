pipeline {
    agent any
    stages {
        stage('Install Earthly') {
            steps {
                sh "wget https://github.com/earthly/earthly/releases/download/v0.8.3/earthly-linux-amd64 -O /usr/local/bin/earthly"
                sh "chmod +x /usr/local/bin/earthly"
                sh "/usr/local/bin/earthly bootstrap"
            }
        }
    }
}
