pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '180906/demowebapp'
        EC2_HOST = '16.170.204.177'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/Kousar0904/demowebapp.git'
            }
        }

        stage('Build WAR with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Copy Project to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} 'mkdir -p /home/ubuntu/demowebapp'
                        scp -o StrictHostKeyChecking=no -r * ubuntu@${EC2_HOST}:/home/ubuntu/demowebapp/
                    """
                }
            }
        }

        stage('Build Docker Image on EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} '
                        cd /home/ubuntu/demowebapp
                        docker build -t ${DOCKER_IMAGE}:latest .
                        docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        '
                    """
                }
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                sshagent(['aws-ec2-ssh-key']) {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} '
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker push ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            '
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
    }
}
