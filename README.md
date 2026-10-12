# Bài 3: Tối Ưu Tốc Độ CI Bằng Gradle Dependency Caching

## 🎯 1. Mục Tiêu
- Bật tính năng **Dependency Caching** cho Gradle trong kịch bản GitHub Actions CI thông qua tham số `cache: 'gradle'` của action `actions/setup-java@v5`.
- Giảm thiểu thời gian tải các thư viện Spring Boot từ Internet (Maven Central) qua mỗi lần chạy CI pipeline.
- Đo lường và lập báo cáo định lượng sự chênh lệch thời gian giữa lần chạy chưa có cache (Cache Miss) và lần chạy đã có cache (Cache Hit).

---

## 📁 2. Cấu Trúc Thư Mục
```text
bai3/
├── .github/
│   └── workflows/
│       └── ci.yml               # Workflow có cấu hình cache: 'gradle'
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/java/com/example/demo/
│   │   └── DemoApplication.java
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java
├── .gitignore
├── build.gradle
├── gradlew                      # Executable script (chmod 755)
├── gradlew.bat
├── settings.gradle
├── report.md                    # Báo cáo đo đạc chi tiết Cache Miss vs Cache Hit
└── README.md                    # Hướng dẫn & tổng quan kỹ thuật
```

---

## ⚙️ 3. Cấu Hình Workflow (`.github/workflows/ci.yml`)

```yaml
name: Java Spring Boot CI (Gradle with Cache)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests with Gradle Cache
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar
```

---

## 📊 4. Kết Quả Đo Đạc & Đánh Giá
Chi tiết bảng so sánh các bước thực thi được trình bày đầy đủ tại [report.md](file:///c:/hoccode/Devops%20Fundamentals/BaiTap/session11/bai3/report.md).

**Tóm tắt hiệu quả:**
- **Lần 1 (Chưa có Cache - Cache Miss)**: Tổng thời gian chạy ~55s do Gradle phải tải toàn bộ thư viện dependencies từ Maven Central.
- **Lần 2 (Đã có Cache - Cache Hit)**: Bước setup-java khôi phục cache thành công:
  ```text
  Cache restored successfully from key: setup-java-Linux-x64-gradle-...
  ```
  Tổng thời gian chạy giảm xuống chỉ còn **~17s (Tăng tốc độ hơn 3.2 lần, giảm ~69% thời gian chờ)**.

---

## 🚀 5. Hướng Dẫn Kiểm Tra Trên GitHub Actions
1. **Lần push 1**:
   - Đẩy commit lên nhánh `main`, theo dõi log của `Post Setup Java` để thấy tiến trình lưu Cache vào GitHub.
2. **Lần push 2**:
   - Thực hiện một commit nhỏ (ví dụ cập nhật comment hoặc `report.md`) và đẩy lên `main`.
   - Xem log của bước `Thiết lập môi trường Java JDK 17`, thấy dòng thông báo khôi phục cache: `Cache restored successfully from key: ...`.
