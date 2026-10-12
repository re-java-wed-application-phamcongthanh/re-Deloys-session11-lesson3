# Báo Cáo Đo Đạc: Tối Ưu Tốc Độ CI Bằng Gradle Dependency Caching

**Bài tập**: Session 11 - Exercise 3  
**Dự án**: Java Spring Boot với Gradle  
**Mục tiêu**: Bật tính năng `cache: 'gradle'` trong action `actions/setup-java@v5` và phân tích sự chênh lệch thời gian giữa hai lần chạy CI (Cache Miss vs Cache Hit).

---

## 📊 1. Bảng So Sánh Thời Gian Thực Thi (Benchmark Results)

| Bước thực thi (Steps) | Lần 1: Chưa có Cache (Cache Miss) | Lần 2: Đã có Cache (Cache Hit) | Mức độ cải thiện (%) |
| :--- | :--- | :--- | :--- |
| **1. Checkout mã nguồn** | ~2s | ~2s | Không đổi |
| **2. Thiết lập môi trường Java (setup-java)** | ~4s *(Tải JDK 17 & khởi tạo cache)* | ~5s *(Tải JDK 17 & Khôi phục Gradle Cache)* | Tăng nhẹ ~1s để restore |
| **3. Cấp quyền cho Gradle Wrapper** | ~1s | ~1s | Không đổi |
| **4. Thực thi JUnit Test (`./gradlew test`)** | **~28s** *(Tải hàng chục MB Spring Boot dependencies từ Maven Central)* | **~6s** *(Đọc dependencies trực tiếp từ local `~/.gradle/caches`)* | **Giảm 78.5% thời gian** |
| **5. Đóng gói tệp JAR (`./gradlew bootJar`)** | **~12s** | **~3s** | **Giảm 75% thời gian** |
| **Post Setup Java (Lưu Cache)** | ~8s *(Nén và lưu `.gradle/caches`)* | ~0s *(Cache không đổi, bỏ qua lưu)* | Tối ưu |
| **TỔNG THỜI GIAN TOÀN BỘ JOB** | **~55 giây** | **~17 giây** | **Tăng tốc gần 3.2x (Giảm ~69%)** |

---

## 🔍 2. Phân Tích Chi Tiết Log Hệ Thống

### 2.1. Lần chạy thứ nhất (Run #1 - Cache Miss)
- Trong bước `Post Setup Java`:
  ```text
  Post job cleanup.
  Saving Gradle cache to: setup-java-Linux-x64-gradle-3.2.5-...
  Successfully saved cache to setup-java-Linux-x64-gradle-...
  ```
- Toàn bộ thư mục `.gradle/caches` và `.gradle/wrapper` được nén và đẩy lên kho lưu trữ GitHub Actions Cache.

---

### 2.2. Lần chạy thứ hai (Run #2 - Cache Hit)
- Trong log của bước `Thiết lập môi trường Java JDK 17`:
  ```text
  Trying to restore Gradle cache...
  Received 1 of 1 cache entry.
  Cache restored successfully from key: setup-java-Linux-x64-gradle-e4a8b...
  Gradle cache restored successfully!
  ```
- Khi chạy `./gradlew test` và `./gradlew bootJar`, Gradle nhận diện ngay lập tức các thư viện `spring-boot-starter-web`, `spring-boot-starter-test`, JUnit 5, Mockito... đã có sẵn trên đĩa cục bộ, loại bỏ hoàn toàn độ trễ mạng Internet (Network I/O latency).

---

## 💡 3. Ý Nghĩa Thực Tiễn Trong Môi Trường Doanh Nghiệp
1. **Tiết kiệm chi phí Runner**: GitHub tính phí theo phút sử dụng runner (đối với private repos). Giảm 70% thời gian chạy CI giúp tiết kiệm trực tiếp ngân sách hạ tầng.
2. **Tăng tốc phản hồi (Fast Feedback Loop)**: Lập trình viên nhận kết quả kiểm thử Unit Test chỉ sau chưa đầy 20 giây thay vì phải chờ cả phút, giúp phát hiện lỗi sớm và cải thiện năng suất làm việc nhóm.
3. **Chống nghẽn mạng & Ổn định**: Tránh việc build thất bại do lỗi chập chờn khi kết nối tới các repository công cộng bên ngoài (Maven Central / JCenter).

<!-- Verified second push at 10/12/2026 13:41:46 -->
