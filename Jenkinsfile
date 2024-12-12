@Library('shared-library') _
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'task-ivolve'
        DOCKER_IMAGE_NAME = '3omda1/firstimage'
        GITHUB_REPO_URL = 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
        openshiftCredentialsID = 'token-oc'
        openshiftClusterURL = 'https://api.ocp-training.ivolve-test.com:6443'
        openshiftProject = 'ahmedemad'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: GITHUB_REPO_URL
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
        } // <-- Fixed closing brace here for the Push to Docker Hub stage

        stage('Deploy on OpenShift Cluster') {
            steps {
                script {
                    deployToOpenShift("${openshiftCredentialsID}", "${openshiftClusterURL}", "${openshiftProject}", "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
    }

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
