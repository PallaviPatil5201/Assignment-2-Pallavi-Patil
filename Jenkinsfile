pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pallavipatil5201/flask-webserver"
        DOCKER_CREDENTIALS = "docker-hub-credentials"
        DEPLOY_SERVER = "your-server-ip"
        DEPLOY_USER = "ubuntu"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/PallaviPatil5201/flask-webserver.git'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest tests/' // Run tests (optional)
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: DOCKER_CREDENTIALS, url: 'https://index.docker.io/v1/']) {
                    sh "docker push $DOCKER_IMAGE"
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['server-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_SERVER << EOF
                        docker pull $DOCKER_IMAGE
                        docker stop flask-app || true
                        docker rm flask-app || true
                        docker run -d --name flask-app -p 80:5000 $DOCKER_IMAGE
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful! Access the app at http://$DEPLOY_SERVER"
        }
        failure {
            echo "Build or Deployment Failed!"
        }
    }
}
