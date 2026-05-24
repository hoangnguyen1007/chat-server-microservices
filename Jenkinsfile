pipeline {
    agent any

    tools {
        jdk 'Java17'     // Tên cấu hình JDK trong Jenkins
        maven 'Maven3'   // Tên cấu hình Maven trong Jenkins
    }

    environment {
        // Tài khoản DockerHub của team
        DOCKER_USER = "hoangnguyen1007"
        DOCKER_CREDS_ID = "docker-hub-creds"

        // Version tự tăng dần
        PROJECT_VERSION = "1.0.0-${BUILD_NUMBER}"

        // Cấu hình máy chủ
        STAGING_IP = "10.0.0.101"
        PROD_IP = "10.0.0.201"
//         SSH_STAGING_CREDS = "ssh-staging-key"
//         SSH_PROD_CREDS = "ssh-prod-key"

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
                echo "2️⃣ Xây dựng thư viện dùng chung..."
                sh './mvnw clean install -pl common-lib -am -DskipTests'
            }
        }

        stage('3. Unit Test & Coverage') {
            steps {
                echo "3️⃣ Chạy kiểm thử tự động nội bộ..."
                sh './mvnw clean test'
            }
        }
        stage('Static Code Analysis (SonarQube)') {
            steps {
                echo '=== ĐANG QUÉT CHẤT LƯỢNG MÃ NGUỒN BẰNG SONARQUBE ==='
                // Chạy plugin maven sonar để đẩy dữ liệu phân tích lên server localhost:9000
                sh './mvnw sonar:sonar -Dsonar.host.url=http://sonarqube:9000'
            }
        }

        stage('Security Vulnerability Scan (Dependency-Check)') {
            steps {
                echo '=== ĐANG QUÉT LỖI BẢO MẬT CÁC THƯ VIỆN ĐANG DÙNG (CVE) ==='
                // Sử dụng OWASP Dependency-Check plugin của Maven để quét file pom.xml
                sh './mvnw org.owasp:dependency-check-maven:check'
            }
        }
        stage('4. Đóng gói Docker Images') {
            steps {
                script {
                    echo "4️⃣ Bắt đầu Build Docker Images cho 10 Services..."
                    def serviceList = env.SERVICES.split(' ')
                    for (int i = 0; i < serviceList.length; i++) {
                        def service = serviceList[i]
                        sh "docker build -t ${env.DOCKER_USER}/${service}:latest -f ${service}/Dockerfile ."
                    }
                }
            }
        }

        stage('5. Push Lên Registry') {
            steps {
                echo "5️⃣ Gửi file đóng gói lên Docker Hub..."
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDS_ID, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ID')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER_ID" --password-stdin'

                    script {
                        def serviceList = env.SERVICES.split(' ')
                        for (int i = 0; i < serviceList.length; i++) {
                            sh "docker push ${env.DOCKER_USER}/${serviceList[i]}:latest"
                        }
                    }
                }
            }
        }

        stage('6. Deploy To Staging (Test Server)') {
            steps {
                echo "6️⃣ Tự động cài đặt lên máy chủ Staging..."
                sshagent([env.SSH_STAGING_CREDS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no deploy@${STAGING_IP} '
                            cd /opt/chat-server &&
                            docker-compose -f docker-compose.staging.yml pull &&
                            docker-compose -f docker-compose.staging.yml up -d
                        '
                    """
                }
                sleep time: 40, unit: 'SECONDS'
            }
        }

        stage('7. Xin Duyệt Lên Môi Trường Thật (Manual Approval)') {
            steps {
                timeout(time: 2, unit: 'DAYS') {
                    input message: "Bạn có chắc chắn muốn đẩy phiên bản ${env.PROJECT_VERSION} này lên cho Khách Hàng dùng không?", ok: "Duyệt - Deploy To Prod"
                }
            }
        }

//         stage('8. Deploy To Production (Thực Tế)') {
//             steps {
//                 echo "8️⃣ Đang cập nhật hệ thống thật..."
//                 sshagent([env.SSH_PROD_CREDS]) {
//                     sh """
//                         ssh -o StrictHostKeyChecking=no deploy@${PROD_IP} '
//                             cd /opt/chat-server &&
//                             docker-compose -f docker-compose.prod.yml pull &&
//                             docker-compose -f docker-compose.prod.yml up -d
//                         '
//                     """
//                 }
//             }
//         }
//     }

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