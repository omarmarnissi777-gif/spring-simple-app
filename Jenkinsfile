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
                sh 'mvn install'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/**/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-server') {
                    sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=devsecops-project-key \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$sonarlogin'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    }
}
