pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID = 'task-ivolve'                          // DockerHub credentials ID.
        imageName              = '3omda1/firstimage'         // DockerHub repo/image name.
        openshiftCredentialsID = 'openshift-sa-token'                    // service account token credentials ID
        openshiftClusterURL    = 'https://api.ocp-training.ivolve-test.com:6443' // OpenShift Cluster URL.
        openshiftProject       = 'ahmedemad'                         // OpenShift project name.
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Navigate to the directory that contains Dockerfile
                    buildDockerImage("${imageName}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Navigate to the directory that contains Dockerfile
                    pushDockerImage("${dockerHubCredentialsID}", "${imageName}")
                }
            }
        }

        stage('Deploy on OpenShift Cluster') {
            steps {
                script { 
                    deployToOpenShift("${openshiftCredentialsID}", "${openshiftClusterURL}", "${openshiftProject}", "${imageName}")
                }
            }
        }
        stage('Update Deployment YAML') {
            steps {
                script {
                    updateDeployment("${imageName}", "${BUILD_NUMBER}")
                }
            }
        }
    }

 post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
        always {
            sh '''
                docker logout
                docker image prune -f
            '''
        }
    }
}
