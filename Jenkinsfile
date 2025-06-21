pipeline {
    agent { label 'slave1' }

    environment {	
        DOCKERHUB_CREDENTIALS = credentials('abhishekpande285')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Cloning Repo'
                git 'https://github.com/AbhishekPande285/star-agile-banking-finance.git'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Building with Maven'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build & Tag') {
            steps {
                echo 'Docker Image Build'
                sh "docker build -t abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} ."
                sh "docker tag abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} abhishekpande285/bankapp-eta-app:latest"
            }
        }

        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh "docker push abhishekpande285/bankapp-eta-app:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'Kubernetes_Master',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '*.yaml',
                                    removePrefix: '',
                                    remoteDirectory: '.',
                                    execCommand: 'kubectl apply -f kubernetesdeploy.yaml'
                                )
                            ]
                        )
                    ])
                }
            }
        }
    }
}