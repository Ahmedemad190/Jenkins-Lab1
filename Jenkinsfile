pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'task-ivolve'
        DOCKER_IMAGE_NAME = '3omda1/firstimage'
        GITHUB_REPO_URL = 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
        OPENSHIFT_PROJECT = 'ahmedemad'  // اسم الـ project في OpenShift
        OPENSHIFT_SERVER = 'https://api.ocp-training.ivolve-test.com:6443'  // رابط الـ OpenShift Cluster
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'task-ivolve', url: '']) {
                    sh """
                        docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest
                        docker push ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Use withCredentials to securely provide the kubeconfig file
                    withCredentials([file(credentialsId: 'openshift-sa-token', variable: 'KUBECONFIG_FILE')]) {
                        // Ensure OpenShift cluster is connected
                        sh '''
                            # Set the kubectl context to OpenShift using the kubeconfig file
                            export KUBECONFIG=$KUBECONFIG_FILE
                            oc login --token=${OPENSHIFT_TOKEN} --server=${OPENSHIFT_SERVER}
                            oc project ${OPENSHIFT_PROJECT}

                            # Deploy the image on OpenShift
                            oc new-app ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} --name=lab-app

                            # Expose the app as a service
                            oc expose svc/lab-app --port=80 --name=lab-app-service

                            # Verify the deployment
                            oc rollout status deployment/lab-app
                            oc get deployments
                            oc get services
                            oc get pods
                        '''
                    }
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
