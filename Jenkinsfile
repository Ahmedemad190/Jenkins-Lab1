pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID = 'task-ivolve'                          // DockerHub credentials ID.
        imageName              = '3omda1/firstimage'                    // DockerHub repo/image name.
        k8sCredentialsID       = 'kube-cred'                             // Kubernetes credentials ID (ServiceAccount or kubeconfig).
        k8sClusterURL          = 'https://192.168.49.2:8443'            // Kubernetes Cluster URL.
        k8sNamespace           = 'default'                               // Kubernetes namespace.
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the Git repository
                    git branch: 'main', url: 'https://github.com/mujemi26/jenkins.git'
                }
            }
        }
        
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
                    // Push the Docker image to Docker Hub
                    pushDockerImage("${dockerHubCredentialsID}", "${imageName}")
                }
            }
        }

        stage('Deploy on Kubernetes Cluster') {
            steps {
                script {
                    // Deploy the image to the Kubernetes cluster
                    deployToKubernetes("${k8sCredentialsID}", "${k8sClusterURL}", "${k8sNamespace}", "${imageName}")
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    // Update Kubernetes deployment YAML
                    updateK8sDeployment("${imageName}", "${BUILD_NUMBER}")
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
            cleanWs() // Clean workspace after the build is finished
        }
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}
