pipeline {
    agent any
    environment {
        // Thay tên image của bạn vào đây
        DOCKER_IMAGE = 'sonkhieu/argocd_test'
        // Thay ID credentials bạn vừa tạo ở Bước 3 vào đây
        DOCKER_CREDS_ID = 'docker-hub-credentials'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build image với tag là số Build của Jenkins (ví dụ v1, v2...)
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDS_ID) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
