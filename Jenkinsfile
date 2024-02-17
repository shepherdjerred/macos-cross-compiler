pipeline {
    agent any
    stages {
        stage('Install Earthly') {
            steps {
                sh "sudo apt update"
                sh "sudo apt install -y wget"
                sh "sudo wget https://github.com/earthly/earthly/releases/download/v0.8.3/earthly-linux-amd64 -O /usr/local/bin/earthly"
                sh "sudo chmod +x /usr/local/bin/earthly"
                sh "/usr/local/bin/earthly bootstrap"
            }
        }
    }
}
