pipeline {
    agent { label 'slave1' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerloginid')
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Checking out source code...'
                git 'https://github.com/AbhishekPande285/star-agile-banking-finance.git'
            }
        }

        stage('Application Build') {
            steps {
                echo 'Running Maven Build...'
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} ."
                sh "docker tag abhishekpande285/bankapp-eta-app:${BUILD_NUMBER} abhishekpande285/bankapp-eta-app:latest"
                sh 'docker image list'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo 'Logging in to DockerHub...'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing Docker Image...'
                sh 'docker push abhishekpande285/bankapp-eta-app:latest'
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'Kubernetes_Master',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '*.yaml',
                                    removePrefix: '',
                                    remoteDirectory: '.',
                                    execCommand: 'kubectl apply -f kubernetesdeploy.yaml',
                                    execTimeout: 120000
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: false
                        )
                    ])
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Good job Abhishek!

Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} succeeded.

Check details: ${env.BUILD_URL}
""",
                to: 'abhipande285@gmail.com'
            )
        }

        failure {
            emailext(
                subject: "❌ Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Hi Abhishek,

Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} failed.

Check logs: ${env.BUILD_URL}
""",
                to: 'abhipande285@gmail.com'
            )
        }
    }
}
