@Library('shared-library') _ 
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'task-ivolve'
        DOCKER_IMAGE_NAME = '3omda1/firstimage'
        GITHUB_REPO_URL = 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
        OPENSHIFT_PROJECT = 'ahmedemad'  
        OPENSHIFT_SERVER = 'https://api.ocp-training.ivolve-test.com:6443'  
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    buildandpushDockerImage(DOCKER_IMAGE_NAME, env.BUILD_NUMBER)
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    pushDockerImage(DOCKER_IMAGE_NAME, env.BUILD_NUMBER, DOCKER_HUB_CREDENTIALS)
                }
            }
        } // <-- Closing brace added here for the Push to Docker Hub stage

        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Use withCredentials to securely provide the kubeconfig file
                    sh '''
                        # Set the kubectl context to OpenShift using the kubeconfig file
                        oc login --token=<YOUR_TOKEN> --server=${OPENSHIFT_SERVER} --insecure-skip-tls-verify
                        oc project ${OPENSHIFT_PROJECT}
                        oc new-app ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} --name=lab-app-ivolve
                        # Ensure 3 replicas for the deployment
                        oc scale deployment/lab-app-ivolve --replicas=3
                    '''
                }
            }
        }
    }

    // Using try-catch-finally for post-execution steps
    post {
        always {
            echo "Cleaning up Docker..."
            sh '''
                docker logout
                docker image prune -f
            '''
        }
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}
