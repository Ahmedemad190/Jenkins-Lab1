pipeline {
    // Remove the Docker agent and use the default Jenkins agent
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'task-ivolve'
        DOCKER_IMAGE_NAME = '3omda1/firstimage'
        GITHUB_REPO_URL = 'https://github.com/Ahmedemad190/Jenkins-Lab1.git'
        MINIKUBE_CLUSTER_NAME = 'minikube'
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
                withDockerRegistry([credentialsId: 'task-ivolve', url: '']) {
                    sh """
                        docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest
                        docker push ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Ensure Minikube cluster is running
                    sh '''
                        # Start Minikube if it's not running
                        if ! minikube status | grep -q "Running"; then
                            minikube start --driver=docker
                        fi

                        # Set the kubectl context to Minikube
                        kubectl config use-context minikube

                        # Load image to Minikube's Docker daemon
                        minikube image load ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}

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
