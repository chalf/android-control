# Android Control (Signage Control)

Hệ thống quản lý, giám sát thông số phần cứng và can thiệp cài đặt từ xa cho các thiết bị Android chuyên dụng (Digital Signage, Kiosk, TV Box, thiết bị công nghiệp không có Google Play Services / GMS).

Dự án này là bản tùy biến và tinh chỉnh dành cho môi trường mạng nội bộ (LAN) và thiết bị màn hình quảng cáo, được phát triển dựa trên nền tảng mã nguồn mở **[Headwind MDM](https://h-mdm.com)**.

---

## 1. Nguồn gốc & Bản quyền (Attribution & License)

Dự án này tuân thủ các điều khoản của giấy phép **Apache License 2.0**. Mọi quyền tác giả gốc được giữ nguyên:

* **Tác giả gốc:** 
  * Copyright (c) 2019 Headwind Solutions LLC (http://www.h-sms.com)
  * Copyright (c) 2021 Vsevolod A. Mayorov (https://h-mdm.com)
* **Kho mã nguồn gốc:**
  * Server: [https://github.com/h-mdm/hmdm-server](https://github.com/h-mdm/hmdm-server)
  * Android Client: [https://github.com/h-mdm/hmdm-android](https://github.com/h-mdm/hmdm-android)
  * Docker: [https://github.com/h-mdm/hmdm-docker](https://github.com/h-mdm/hmdm-docker)
* Xem chi tiết tại [LICENSE](LICENSE) và [NOTICE](NOTICE).

---

## 2. Danh sách các tệp và thay đổi đã thực hiện (Modifications)

Để phục vụ tốt nhất cho bài toán Digital Signage trong mạng LAN và tương thích với các công cụ lập trình mới nhất, các thay đổi sau đã được áp dụng:

### A. Ứng dụng Android Client (`hmdm-android`)
1. **`app/build.gradle`:**
   * Cập nhật `BASE_URL` và `SECONDARY_BASE_URL` trỏ về máy chủ mạng LAN nội bộ (`http://192.168.1.103`).
   * Đồng bộ khóa bảo mật `REQUEST_SIGNATURE` khớp với cấu hình server Docker.
   * Giữ nguyên `minSdkVersion 26` để đảm bảo độ ổn định cao nhất cho các thư viện Jackson và AndroidX mới.
2. **`app/src/androidTest/.../ExampleInstrumentedTest.java`:**
   * Dọn dẹp các import lỗi thời từ thư viện cũ (`android.support.test.*`), chuyển sang chuẩn kiểm thử tương thích AndroidX và Gradle 8.x. Khắc phục triệt để lỗi biên dịch `package android.support.test does not exist` khi bấm build trong Android Studio.

### B. Môi trường triển khai Docker Server (`hmdm-docker`)
1. **`docker-compose.yaml`:**
   * Tắt dịch vụ `certbot` để chuyển sang chế độ thuần HTTP cho mạng LAN nội bộ (do IP LAN không cần cấp chứng chỉ SSL Let's Encrypt).
   * Ánh xạ các cổng: Web Panel qua cổng `80` (HTTP chuẩn) và `8088` (dự phòng), MQTT Push qua cổng `31000`.
   * Đổi cổng ngoài của container PostgreSQL thành `5433:5432` để tránh xung đột với các tiến trình database có sẵn trên máy host.
   * Thêm thuộc tính `platform: linux/amd64` cho service `hmdm` để container chạy mượt mà qua trình biên dịch giả lập Rosetta trên các máy Mac Apple Silicon (chip M1/M2/M3/M4).
2. **`.env`:**
   * Cấu hình các biến môi trường cho chế độ HTTP LAN: `PROTOCOL=http`, `BASE_DOMAIN=192.168.1.103`, `LOCAL_IP=192.168.1.103`.

---

## 3. Cấu trúc thư mục

```text
android-control/
├── hmdm-android/       # Mã nguồn ứng dụng Android Client (Java thuần, hỗ trợ ARMv7a/v8a, x86)
├── hmdm-server-5.41.4/       # Mã nguồn Backend Server (Java, Tomcat, Guice, Jersey, MyBatis, PostgreSQL)
├── hmdm-docker/              # Cấu hình Docker Compose triển khai Server nhanh chóng trong LAN
├── LICENSE                   # Giấy phép Apache License 2.0
├── NOTICE                    # Thông cáo bản quyền và ghi nhận tác giả gốc
└── README.md                 # Tài liệu hướng dẫn dự án
```

---

## 4. Hướng dẫn vận hành nhanh

### Bước 1: Khởi động Server (qua Docker)
```bash
cd hmdm-docker
docker compose up -d
```
* **Web Dashboard:** Truy cập qua trình duyệt tại `http://localhost` hoặc `http://<IP-LAN-của-bạn>`
* **Đăng nhập mặc định:** Tài khoản `admin` / Mật khẩu `admin`

### Bước 2: Biên dịch và cài đặt Client lên thiết bị
```bash
cd hmdm-android
./gradlew assembleOpensourceDebug
```
* File APK đầu ra tại: `app/build/outputs/apk/opensource/debug/app-opensource-debug.apk`.
* Chép APK vào USB cắm vào màn hình Signage để cài đặt.
* Ở lần mở đầu tiên, app sẽ tự động kết nối về Server LAN hoặc hiện ô xác nhận IP Server.

---

## 5. Lưu ý khi Router đổi IP mạng
Nếu máy chủ bị Router cấp IP LAN mới (ví dụ đổi từ `192.168.1.103` thành `192.168.1.150`):
1. **Trên máy chủ:** Sửa `BASE_DOMAIN` và `LOCAL_IP` trong `hmdm-docker/.env` rồi chạy `docker compose up -d`.
2. **Trên màn hình Signage:** 
   * Khi mất mạng, app sẽ tự hiện hộp thoại thông báo lỗi kèm nút **Reset**. Bấm **Reset** để nhập IP mới mà không cần cài lại app.
   * Hoặc nhấn giữ lâu (Long-press) 2-3 giây trên màn hình nền launcher -> nhập mật khẩu `12345678` -> chọn **Change Server URL**.
