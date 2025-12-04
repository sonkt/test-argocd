pipeline {
    agent any
    environment {
        // Thay tên image của bạn vào đây
        DOCKER_IMAGE = 'sonkhieu/argocd_test'
        // Thay ID credentials bạn vừa tạo ở Bước 3 vào đây
        DOCKER_CREDS_ID = 'docker-hub-credentials'
        EMAIL = 'sonkt@outlook.com'
        GIT_CREDS = 'github-token-creds'        
        GIT_REPO_URL = 'github.com/sonkt/test-argocd.git'
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
        stage('Update Manifest (GitOps)') {
            steps {
                script {
                    // Cấu hình Git trong Jenkins
                    sh "git config user.email '${env.EMAIL}'"
                    sh "git config user.name 'Jenkins Bot'"
                    
                    // Thay thế version cũ bằng version mới trong file deployment.yaml
                    // Lệnh sed này tìm dòng image: ... và thay số đuôi
                    sh "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' deployment.yaml"
                    
                    // Commit và Push ngược lại GitHub
                    withCredentials([usernamePassword(credentialsId: GIT_CREDS, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh "git add deployment.yaml"
                        sh "git commit -m 'Jenkins update image version to v${env.BUILD_NUMBER}'"
                        // Push dùng Token
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL} HEAD:main"
                    }
                }
            }
        }
    }
}
