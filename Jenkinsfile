pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '180906/demowebapp'
        EC2_HOST = '51.20.5.58'
        PEM_FILE = '"D:\devopsacdemy\first_instance.pem"'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/Kousar0904/demowebapp.git'
            }
        }

        stage('Build WAR with Maven') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Test EC2 Connection') {
            steps {
                bat """
                ssh -o StrictHostKeyChecking=no -i %PEM_FILE% ubuntu@%EC2_HOST% "hostname"
                """
            }
        }

        stage('Copy Project to EC2') {
            steps {
                bat """
                ssh -o StrictHostKeyChecking=no -i %PEM_FILE% ubuntu@%EC2_HOST% "mkdir -p /home/ubuntu/demowebapp"

                scp -o StrictHostKeyChecking=no -i %PEM_FILE% -r * ubuntu@%EC2_HOST%:/home/ubuntu/demowebapp/
                """
            }
        }

        stage('Build Docker Image on EC2') {
            steps {
                bat """
                ssh -o StrictHostKeyChecking=no -i %PEM_FILE% ubuntu@%EC2_HOST% "cd /home/ubuntu/demowebapp && docker build -t %DOCKER_IMAGE%:latest . && docker tag %DOCKER_IMAGE%:latest %DOCKER_IMAGE%:%BUILD_NUMBER%"
                """
            }
        }

        stage('Docker Hub Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat """
                    ssh -o StrictHostKeyChecking=no -i %PEM_FILE% ubuntu@%EC2_HOST% "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin && docker push %DOCKER_IMAGE%:latest && docker push %DOCKER_IMAGE%:%BUILD_NUMBER%"
                    """
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
