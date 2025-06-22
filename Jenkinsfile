stage('Deploy to Kubernetes Cluster') {
    steps {
        script {
            // Check and log what's in workspace
            sh 'ls -l && cat kubernetesdeploy.yaml'

            // Use exact filename instead of wildcard for clarity
            sshPublisher(publishers: [
                sshPublisherDesc(
                    configName: 'Kubernetes_Master',
                    transfers: [
                        sshTransfer(
                            sourceFiles: 'kubernetesdeploy.yaml',   // precise name
                            removePrefix: '',                      // don’t strip folder structure
                            remoteDirectory: '/home/ubuntu',       // absolute path
                            execCommand: 'kubectl apply -f /home/ubuntu/kubernetesdeploy.yaml',
                            flatten: true                          // put file directly in remoteDirectory
                        )
                    ],
                    verbose: true
                )
            ])
        }
    }
}
