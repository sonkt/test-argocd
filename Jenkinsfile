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
        stage('Update Manifest (GitOps)') {
            steps {
                script {
                    // 1. Cấu hình định danh cho Git bot
                    sh "git config user.email '${EMAIL}'"
                    sh "git config user.name 'Jenkins Bot'"
                    
                    // 2. Debug: In ra dòng image hiện tại để kiểm tra
                    echo "--- TRƯỚC KHI UPDATE ---"
                    sh "cat deployment.yaml | grep image"
                    
                    // 3. Sửa file: Tìm dòng chứa 'image:', thay thế bằng version mới
                    // Lưu ý: Lệnh này thay thế bất kể nội dung cũ là gì
                    sh "sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' deployment.yaml"
                    
                    // 4. Debug: In ra kết quả sau khi sửa
                    echo "--- SAU KHI UPDATE ---"
                    sh "cat deployment.yaml | grep image"

                    // 5. Commit và Push ngược lên GitHub
                    withCredentials([usernamePassword(credentialsId: GIT_CREDS, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git add deployment.yaml
                            # Chỉ commit nếu có sự thay đổi
                            git diff-index --quiet HEAD || git commit -m 'Jenkins update image version to v${env.BUILD_NUMBER}'
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL} HEAD:main
                        """
                    }
                }
            }
        }
    }
}
