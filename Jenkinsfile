pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1'
    }
    stages {

        stage('Checkout git') {
            steps {
                git branch: 'main', url: 'https://github.com/omarmarnissi777-gif/spring-simple-app.git'
            }
        }

    }
}
