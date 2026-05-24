pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        DOCKER_USER = "hoangnguyen1007"
        DOCKER_CREDS_ID = "docker-hub-creds"
        PROJECT_VERSION = "1.0.0-${BUILD_NUMBER}"
        STAGING_IP = "10.0.0.101"
        PROD_IP = "10.0.0.201"
        SERVICES = "auth-service channel-service gateway-service log-service messaging-service notification-service presence-service role-service server-service user-profile-service file-service"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                echo "1️⃣ Kéo mã nguồn mới nhất từ GitHub..."
                git branch: 'main', url: 'https://github.com/hoangnguyen1007/chat-server-microservices'
            }
        }

        stage('2. Build Common Lib') {
            steps {
                echo "2️⃣ Xây dựng thư viện dùng chung common-lib..."
                sh 'cd common-lib && mvn clean install'
            }
        }

        stage('3. Run Automated Tests') {
            steps {
                echo "3️⃣ Khởi chạy toàn bộ Unit Test hệ thống..."
                sh 'mvn test'
            }
        }

        stage('4. Static Code Analysis') {
            steps {
                echo "4️⃣ Quét chất lượng source code với SonarQube..."
                sh 'mvn sonar:sonar -Dsonar.host.url=http://sonarqube:9002'
            }
        }

        stage('5. Security Vulnerability Scan') {
            steps {
                echo "5️⃣ Quét lỗ hổng bảo mật thư viện第三方 bằng OWASP Dependency-Check..."
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        stage('6. Compile, Package & Push to DockerHub') {
            steps {
                echo "6️⃣ Đóng gói ứng dụng JAR và đẩy Docker Image lên mạng công khai..."
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"

                    sh """
                        for service in ${SERVICES}; do
                            echo "📦 Đang xử lý dịch vụ: \$service..."
                            cd \$service
                            mvn clean package -DskipTests
                            docker build -t ${DOCKER_USER}/\$service:${PROJECT_VERSION} .
                            docker tag ${DOCKER_USER}/\$service:${PROJECT_VERSION} ${DOCKER_USER}/\$service:latest
                            docker push ${DOCKER_USER}/\$service:${PROJECT_VERSION}
                            docker push ${DOCKER_USER}/\$service:latest
                            cd ..
                        done
                    """
                }
            }
        }

        stage('7. Deploy To Staging (Môi trường Thử Nghiệm)') {
            steps {
                echo "7️⃣ Tiến hành cập nhật máy chủ Staging qua kết nối an toàn SSH..."
                sh "echo 'Simulating SSH deployment to Staging Environment...'"
                sleep time: 10, unit: 'SECONDS'
            }
        }

        stage('8. Xin Duyệt Lên Môi Trường Thật (Manual Approval)') {
            steps {
                timeout(time: 2, unit: 'DAYS') {
                    input message: "Bạn có chắc chắn muốn đẩy phiên bản ${env.PROJECT_VERSION} này lên cho Khách Hàng dùng không?", ok: "Duyệt - Deploy To Prod"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Xuất Sắc! Pipeline chạy thành công 100%!"
        }
        failure {
            echo "❌ Pipeline bị Lỗi rồi! DevOps vào kiểm tra ngay!"
        }
        always {
            cleanWs()
            sh 'docker system prune -f'
        }
    }
}