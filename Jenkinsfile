pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/myapp"
        KUBE_NAMESPACE = "default"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git 'https://github.com/<your-username>/blue-green-deployment.git'
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    def tag = "green-${env.BUILD_ID}"
                    sh "docker build -t $DOCKER_IMAGE:$tag ."
                    sh "docker push $DOCKER_IMAGE:$tag"
                }
            }
        }

        stage('Deploy to Green') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s/green-deployment.yaml
                    kubectl set image deployment/green-deployment myapp=$DOCKER_IMAGE:green-${env.BUILD_ID} --namespace=$KUBE_NAMESPACE
                    """
                }
            }
        }

        stage('Test Green') {
            steps {
                echo 'Testing the green environment...'
                // You can add health check or test scripts here
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                script {
                    sh "kubectl patch service myapp-service -p '{\"spec\":{\"selector\":{\"app\":\"myapp\",\"version\":\"green\"}}}'"
                }
            }
        }

        stage('Clean Up Blue') {
            steps {
                sh "kubectl delete deployment blue-deployment || true"
            }
        }
    }
}
