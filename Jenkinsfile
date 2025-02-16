pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pallavipatil107/assignment2"
        DOCKER_CREDENTIALS = "dockerhub-creds"
        SSH_CREDENTIALS = "ssh-server-creds"
        SERVER_IP = "your-server-ip"   // Replace with actual server IP
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PallaviPatil5201/Assignment-2-Pallavi-Patil.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE"
                }
            }
        }

        stage('Deploy to Web Server') {
            steps {
                sshagent(['ssh-server-creds']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@$SERVER_IP << EOF
                        docker pull $DOCKER_IMAGE
                        docker stop my-flask-app || true
                        docker rm my-flask-app || true
                        docker run -d -p 80:5000 --name my-flask-app $DOCKER_IMAGE
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully! Visit http://$SERVER_IP"
        }
        failure {
            echo "Build or Deployment failed!"
        }
    }
}
