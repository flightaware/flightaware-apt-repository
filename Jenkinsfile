pipeline {
    agent {label "raspberrypi"}
    options {
        disableConcurrentBuilds()
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source"
            }
        }
    }
}