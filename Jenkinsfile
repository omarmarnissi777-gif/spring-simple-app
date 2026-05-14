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
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh 'mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=spring-simple-app \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_TOKEN'
                    }
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

        stage('Docker Build') {
            steps {
                sh 'docker build -t omarmarnissi777/spring-simple-app:v1.$BUILD_ID .'
                sh 'docker image tag omarmarnissi777/spring-simple-app:v1.$BUILD_ID omarmarnissi777/spring-simple-app:latest'
            }
        }

        stage('Image Scan') {
            steps {
                sh 'trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html omarmarnissi777/spring-simple-app:latest'
            }
        }

        stage('Upload Scan Report to AWS S3') {
            steps {
                sh 'aws s3 cp report.html s3://project-arch-omarghaithayoubmarrouki/'
            }
        }

    }
}
