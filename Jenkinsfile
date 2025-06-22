pipeline {
    agent { label 'slave1' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Perform SCM Checkout'
                git 'https://github.com/AbhishekPande285/star-agile-banking-finance.git'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Perform Application Build'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Perform Docker Build'
                sh "docker build -t abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} ."
                sh "docker tag abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} abhishekpande285/bankapp-eta-app:latest"
                sh 'docker image list'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo 'Login to DockerHub'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Publish the Image to DockerHub') {
            steps {
                echo 'Push Docker Image to DockerHub'
                sh 'docker push abhishekpande285/bankapp-eta-app:latest'
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    echo 'Copying deployment YAML and applying to Kubernetes Cluster'
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'Kubernetes_Master',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'kubernetesdeploy.yaml',
                                    remoteDirectory: '.', // home of devopsadmin
                                    execCommand: 'kubectl apply -f kubernetesdeploy.yaml',
                                    flatten: true
                                )
                            ],
                            verbose: true
                        )
                    ])
                }
            }
        }
    }
}
