def sendSlackNotification() {
    if (currentBuild.currentResult == "SUCCESS") {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
        slackSend(channel: "#notifications", tokenCredentialId: 'slack-token', color: 'good', message: "${buildSummary}")
    } else {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n"
        slackSend(channel: "#notifications", tokenCredentialId: 'slack-token', color: 'danger', message: "${buildSummary}")
    }
}

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

        stage('Docker Push') {
            steps {
                withVault(configuration: [
                    skipSslVerification: true,
                    timeout: 60,
                    vaultCredentialId: 'vault-cred',
                    vaultUrl: 'http://127.0.0.1:8200'],
                    vaultSecrets: [[
                        path: 'secrets/creds/docker',
                        secretValues: [
                            [vaultKey: 'username'],
                            [vaultKey: 'password']
                        ]
                    ]]) {
                    sh "docker login -u ${username} -p ${password}"
                    sh 'docker push omarmarnissi777/spring-simple-app:v1.$BUILD_ID'
                    sh 'docker push omarmarnissi777/spring-simple-app:latest'
                    sh 'docker rmi omarmarnissi777/spring-simple-app:v1.$BUILD_ID omarmarnissi777/spring-simple-app:latest'
                }
            }
        }

        stage('Deploy to k8s') {
            steps {
                script {
                    kubernetesDeploy configs: 'spring-boot-deployment.yaml', kubeconfigId: 'kubernetes'
                }
            }
        }

    }
    post {
        always {
            sendSlackNotification()
        }
    }
}
