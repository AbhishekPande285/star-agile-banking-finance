pipeline {
    agent { label 'slave1' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {

        stage('SCM Checkout') {
            steps {
                echo 'Checking out code from GitHub'
                git 'https://github.com/AbhishekPande285/star-agile-banking-finance.git'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Building the Spring Boot Application'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} ."
                sh "docker tag abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} abhishekpande285/bankapp-eta-app:latest"
                sh 'docker images'
            }
        }

        stage('DockerHub Login') {
            steps {
                echo 'Logging into DockerHub'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker Image to DockerHub'
                sh 'docker push abhishekpande285/bankapp-eta-app:latest'
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'Kubernetes_Master',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: '*.yaml',
                                        remoteDirectory: '/home/ubuntu/star-agile-banking-finance',
                                        execCommand: 'kubectl apply -f /home/ubuntu/star-agile-banking-finance/kubernetesdeploy.yaml',
                                        execTimeout: 120000,
                                        flatten: false
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            mail to: 'abhipande285@gmail.com',
                 subject: "✅ Build #${env.BUILD_NUMBER} Succeeded - Banking App",
                 body: "Good news! The Banking App pipeline completed successfully.\n\nCheck console: ${env.BUILD_URL}"
        }
        failure {
            mail to: 'abhipande285@gmail.com',
                 subject: "❌ Build #${env.BUILD_NUMBER} Failed - Banking App",
                 body: "Something went wrong in the pipeline. Please review the Jenkins logs here:\n\n${env.BUILD_URL}"
        }
    }
}
