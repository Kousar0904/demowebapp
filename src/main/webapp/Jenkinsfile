pipeline {
    agent any

    environment {
        AWS_IP = "<your-aws-public-ip>"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Copy to AWS') {
            steps {
                sshagent(['aws-ssh-key']) {
                    sh """
                        scp -o StrictHostKeyChecking=no \
                        target/myapp.war ubuntu@${AWS_IP}:/home/ubuntu/
                        
                        scp -o StrictHostKeyChecking=no \
                        Dockerfile ubuntu@${AWS_IP}:/home/ubuntu/
                    """
                }
            }
        }

        stage('Build Docker Image on AWS') {
            steps {
                sshagent(['aws-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${AWS_IP} '
                            cd /home/ubuntu &&
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . &&
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest &&
                            docker images
                        '
                    """
                }
            }
        }

    }

    post {
        success {
            echo '✅ Docker Image Built Successfully on AWS!'
        }
        failure {
            echo '❌ Pipeline Failed. Check logs above.'
        }
    }
}
