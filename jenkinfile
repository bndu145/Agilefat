pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = '23mis0686-webpage:latest'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests..."
                sh '''
                    docker run -d --name test-app -p 8888:80 ${DOCKER_IMAGE}
                    sleep 3
                    curl -f http://localhost:8888/ || exit 1
                    docker stop test-app
                    docker rm test-app
                '''
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                sh '''
                    kubectl set image deployment/university-dept-web \
                        university-dept-web=${DOCKER_IMAGE} --record || true
                    kubectl apply -f k8s-deployment.yaml
                    kubectl rollout status deployment/university-dept-web --timeout=3m
                '''
            }
        }
        
        stage('Verify') {
            steps {
                echo "Verifying deployment..."
                sh 'kubectl get pods -l app=university-dept-web'
            }
        }
    }
    
    post {
        always {
            sh 'docker rm -f test-app 2>/dev/null || true'
        }
        success {
            echo "✓ Deployment successful!"
        }
        failure {
            echo "✗ Pipeline failed"
        }
    }
}
