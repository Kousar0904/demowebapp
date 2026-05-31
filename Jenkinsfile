pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = '180906/demowebapp'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('Build WAR with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image on AWS') {
            steps {
                sh '''
                    # Connect to AWS instance and build image
                    ssh -o StrictHostKeyChecking=no ubuntu@16.170.204.177 << 'EOF'
                    cd /workspace/demowebapp
                    docker build -t ${DOCKER_IMAGE}:latest .
                    docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    EOF
                '''
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', 
                                                   usernameVariable: '180906', 
                                                   passwordVariable: 'Kousar@09')]) {
                    sh 'echo ${DOCKER_PASS} | docker login --username ${DOCKER_USER} --password-stdin'
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed!'
        }
    }
}
