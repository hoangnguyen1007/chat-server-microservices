pipeline {
    agent any

    environment {
        // Định nghĩa thư mục làm việc hiện tại để Jenkins chạy lệnh docker-compose
        COMPOSE_PROJECT_NAME = 'chat-server-system'
    }

    stages {
        // -------------------------------------------------------------
        // GIAI ĐOẠN 1: KÉO SOURCE CODE VÀ BIÊN DỊCH MAVEN
        // -------------------------------------------------------------
        stage('Maven Build & Package') {
            steps {
                echo '=== BẮT ĐẦU BIÊN DỊCH TOÀN BỘ MICROSERVICES BẰNG MAVEN ==='
                // Chạy lệnh Maven đóng gói, bỏ qua chạy thử nghiệm test để đẩy nhanh tốc độ
                sh './mvnw clean package -DskipTests'
            }
        }

        // -------------------------------------------------------------
        // GIAI ĐOẠN 2: BUILD LẠI CÁC IMAGE VÀ DEPLOY CONTAINER BẰNG COMPOSE
        // -------------------------------------------------------------
        stage('Docker Compose Deploy') {
            steps {
                echo '=== DÙNG DOCKER COMPOSE ĐỂ TỰ ĐỘNG KHỞI CHẠY HỆ THỐNG ==='

                // Ý nghĩa câu lệnh:
                // --build: Bảo Docker kiểm tra xem file .jar vừa được Maven build xong, có thì bốc đút vào Image mới luôn
                // -d: Chạy ngầm các container
                // --remove-orphans: Dọn dẹp, xóa bỏ các container cũ thừa thãi không còn khai báo trong file compose
                sh 'docker compose up --build -d --remove-orphans'

                echo '=== HỆ THỐNG MICROSERVICES ĐÃ ĐƯỢC JENKINS CẬP NHẬT THÀNH CÔNG ==='
            }
        }
    }

    // ĐOẠN CẤU HÌNH BÁO CÁO KẾT QUẢ CUỐI PHIÊN
    post {
        success {
            echo '🎉 Chúc mừng! Quy trình CI/CD hoàn thành xuất sắc, hệ thống đã lên sóng.'
        }
        failure {
            echo '❌ Lỗi rồi! Kịch bản build thất bại. Hãy kiểm tra lại log của Maven hoặc Docker.'
        }
    }
}