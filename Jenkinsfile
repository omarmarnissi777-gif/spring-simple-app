pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1'
        jdk 'java-17'
    }
    stages {

        stage('Checkout git') {
            steps {
                git branch: 'main', url: 'https://github.com/omarmarnissi777-gif/spring-simple-app.git'
            }
        }

        stage('Build & JUnit Test') {
            steps {
                sh 'java -version'
                sh 'mvn install'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }

    }
}
