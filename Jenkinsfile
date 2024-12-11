pipeline {
    // Remove the Docker agent and use the default Jenkins agent
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'task-ivolve'
        DOCKER_IMAGE_NAME = '3omda1/firstimage'
        GITHUB_REPO_URL = 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
        KIND_CLUSTER_NAME = 'minikube'
    }


    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from the repository 
                git branch: 'main', url: 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
            }
        }

       

        stage('Build Docker Image') {
            steps {
                // Build Docker image using shell command
                sh """
                    docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Login and push image
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh """
                        docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest
                        docker push ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
        stage('Deploy to Kind Cluster') {
            steps {
                script {
                    // Ensure Kind cluster exists and set context
                    sh '''
                        # Create Kind cluster if not exists
                        if ! kind get clusters | grep -q ${KIND_CLUSTER_NAME}; then
                            kind create cluster --name ${KIND_CLUSTER_NAME}
                        fi

                        # Set kubectl context
                        kubectl config use-context kind-${KIND_CLUSTER_NAME}

                        # Load image to Kind cluster
                        kind load docker-image ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} --name ${KIND_CLUSTER_NAME}

                        # Create deployment file
                        cat > deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab-app
  template:
    metadata:
      labels:
        app: lab-app
    spec:
      containers:
      - name: lab-app
        image: ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: lab-app-service
spec:
  type: NodePort
  selector:
    app: lab-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
EOF

                        # Apply deployment
                        kubectl apply -f deployment.yaml
                        
                        # Verify deployment
                        kubectl rollout status deployment/lab-app-deployment
                        kubectl get deployments
                        kubectl get services
                        kubectl get pods
                    '''
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
            // Cleanup
            sh '''
                docker logout
                docker image prune -f
            '''
        }
    }
}
