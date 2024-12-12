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

        stage('Deploy to OpenShift') {
            steps {
                script {
                    sh '''
                        oc login --token=eyJhbGciOiJSUzI1NiIsImtpZCI6Inpvc29nXzlMMy1tX2JELWEwamVlRlAzQ0JWUmZLRGk5UWtiZHZNWDVJUXMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJhaG1lZGVtYWQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiamVua2lucy1zYS10b2tlbi1sbmJ3eiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zLXNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZTQyMzlkM2QtMDE4ZS00ZDcyLWFjNzctNWI5N2M5OWU3MGI0Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmFobWVkZW1hZDpqZW5raW5zLXNhIn0.noj-gQYI60frHfNikBnJXxHVBUrE4TjeH0RJXmjMioVXidClBnge0ZSCLMxuHCQCalKc2aDa3-F6-Q8BMxGAOga-CqEVpv9vhmb71loV0oOndzASa52svTqzk9tX_WTRXxB8KaaVYpSwCzGRbC4rwUXTYgr8sjFXvrSVO3Tvn-URaLfjPa_AOlomU8GRMBILmcDsCUM6W7I-vNxaUYqKmIFmyZC_j2-JtOa7Mu7aiB7BM1U7NnaG40eRRnYeIvJXh3F_6uISlid_IOYhbEd0nulTi-kxJwMQvjVuYj-T0lWgCZJQ7kd-ythQ9Nlyk1SclNDSLUgVtF_Z5nAzR0WXNrjrjVnxbLngAQ6-8XA46QH0gC_XjBSbibLkHjp5ncF43uN-St16T1KhPhVQFoKW2B9PyC5X1tX8oF2_K0o0gpKOdpUzD1q_XshuK8FYiNcdtcNnht8h0EJwuJvPTJrOrV9rJXPadeFzkrWirDnM3OdBlbOCpi0uL4aUXuHf9VVBXm-nYKFCpU0JxAUliViNJXf1xJpAxIOfTB-2JiK9j-9lmXYm8wN9XRBjGJgCIzIMq4OIEAmK4H98StQ49K_xNvwKUTSdXy4-XDKBLksnyGY0tCDIuZMANI16_FdTOwJkvcL2rMbkdHsVkroLDcKr_AwUyeoLyhCWVP9YDS-GQ0Y --server=https://api.ocp-training.ivolve-test.com:6443 --insecure-skip-tls-verify
                        oc project ahmedemad
                        oc new-app ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} --name=lab-app-ivolve
                        oc scale deployment/lab-app-ivolve --replicas=3
                    '''
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
