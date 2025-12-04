pipeline {
    agent any
    environment {
        // --- CẤU HÌNH ---
        DOCKER_IMAGE = 'sonkhieu/argocd_test'       
        DOCKER_CREDS = 'docker-hub-credentials' 
        GIT_CREDS = 'github-token-creds'        
        GIT_REPO_URL = 'github.com/sonkt/test-argocd.git' 
        EMAIL = 'sonkt@outlook.com'             
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'echo "Running tests..."'
            }
        }

        stage('Build & Push Docker') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    
                    // Dùng env.DOCKER_CREDS
                    docker.withRegistry('', env.DOCKER_CREDS) {
                        def appInfo = docker.build("${env.DOCKER_IMAGE}:${imageTag}")
                        appInfo.push()
                        appInfo.push('latest')
                    }
                }
            }
        }

        stage('Update Manifest (GitOps)') {
            steps {
                script {
                    sh "git config user.email '${env.EMAIL}'"
                    sh "git config user.name 'Jenkins Bot'"
                    
                    echo "--- TRƯỚC KHI UPDATE ---"
                    sh "cat deployment.yaml | grep image"
                    
                    // Lệnh sed (Đã thêm env.)
                    sh "sed -i 's|image:.*|image: ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}|g' deployment.yaml"
                    
                    echo "--- SAU KHI UPDATE ---"
                    sh "cat deployment.yaml | grep image"

                    // Dùng env.GIT_CREDS
                    withCredentials([usernamePassword(credentialsId: env.GIT_CREDS, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                            git add deployment.yaml
                            git diff-index --quiet HEAD || git commit -m 'Jenkins update image version to v${env.BUILD_NUMBER}'
                            
                            # --- SỬA LỖI TẠI DÒNG NÀY ---
                            # Đã thêm env. vào trước user và pass
                            git push https://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@${env.GIT_REPO_URL} HEAD:main
                        """
                    }
                }
            }
        }
    }
}
