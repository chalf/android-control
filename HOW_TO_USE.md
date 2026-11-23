# Tổng hợp các lệnh quản trị & điều khiển Android Signage (Headwind MDM)
http://localhost

---

## 1. Các lệnh Push điều khiển từ xa trên Web Dashboard
*(Truy cập: **Devices -> nút `...` -> Push messages** hoặc **Functions -> Push messages -> New message**)*

| Type (Loại lệnh) | Payload (JSON) | Tác dụng & Điều kiện |
| :--- | :--- | :--- |
| **`configUpdated`** | *(Để trống)* | Ép thiết bị tải lại và áp dụng cấu hình mới ngay lập tức |
| **`runApp`** | `{"pkg": "com.android.chrome"}` | Mở từ xa một ứng dụng theo Package ID *(hỗ trợ thêm `"action"`, `"data"`, `"extra"`)* |
| **`reboot`** | *(Để trống)* | Khởi động lại thiết bị từ xa *(cần quyền Device Owner hoặc Root)* |
| **`runCommand`** | `{"command": "lệnh shell"}` | Thực thi lệnh shell trực tiếp trên Android *(kết quả trả về trong **Logs**, tối đa 200 ký tự)* |
| **`intent`** | `{"action": "android.intent.action.VIEW", "data": "https://example.com"}` | Gọi System Intent (mở link web, mở màn hình Cài đặt, v.v.) |
| **`uninstallApp`** | `{"pkg": "app.package.id"}` | Gỡ cài đặt ứng dụng từ xa *(cần quyền Device Owner)* |
| **`clearAppData`** | `{"pkg": "app.package.id"}` | Xóa sạch dữ liệu và cache của ứng dụng |
| **`grantPermissions`** | `{"pkg": "app.package.id"}` | Tự động cấp quyền runtime cho ứng dụng *(cần Device Owner hoặc `SYSTEM_PRIVILEGES`)* |
| **`deleteFile`** | `{"path": "/ten_thu_muc/ten_file"}` | Xóa 1 file trong bộ nhớ ngoài (`ExternalStorageDirectory`) |
| **`deleteDir`** | `{"path": "/ten_thu_muc"}` | Xóa toàn bộ thư mục và file bên trong |
| **`purgeDir`** | `{"path": "/ten_thu_muc", "recursive": "1"}` | Làm sạch các file bên trong thư mục (giữ lại thư mục gốc) |
| **`adminPanel`** | *(Để trống)* | Mở màn hình Admin Panel trên thiết bị Android |
| **`exitKiosk`** | *(Để trống)* | Tạm thoát chế độ khóa Kiosk Mode |
| **`permissiveMode`** | *(Để trống)* | Bật chế độ cho phép tạm thời (bỏ qua chặn app) |
| **`clearDownloadHistory`** | *(Để trống)* | Xóa lịch sử các file đã tải về trong database của app Android |

### Một số lệnh Shell hay dùng với `runCommand`
*(Xem kết quả trả về tại **Functions -> Logs**)*
* Liệt kê các app bên thứ 3 đã cài: `{"command": "pm list packages -3"}`
* Kiểm tra dung lượng bộ nhớ trống: `{"command": "df -h /data"}`
* Kiểm tra thông tin CPU/phần cứng: `{"command": "getprop ro.product.cpu.abi"}`
* Kiểm tra IP mạng của thiết bị: `{"command": "ip addr show wlan0"}` *(hoặc `eth0` cho mạng dây)*

---

## 2. Các lệnh Docker quản lý Server trên máy Mac
*(Chạy trong thư mục `~/repos/signage_control/hmdm-docker`)*

```bash
# Khởi chạy server (hoặc tạo lại container để nạp file .env mới)
docker compose up -d

# Xem nhật ký hoạt động (log) của Server theo thời gian thực
docker compose logs -f hmdm

# Dừng toàn bộ hệ thống Server (giữ nguyên dữ liệu)
docker compose down

# Kiểm tra trạng thái các container đang chạy
docker ps
```

---

## 3. Các lệnh xử lý khi máy Mac bị đổi IP mạng
*(Ví dụ: đổi từ `192.168.1.134` sang `192.168.1.150`)*

```bash
# 1. Kiểm tra IP LAN hiện tại của máy Mac
ipconfig getifaddr en0

# 2. Cập nhật file cấu hình (sửa thủ công 2 file sau sang IP mới):
#    - ~/repos/signage_control/hmdm-docker/.env  (sửa BASE_DOMAIN và LOCAL_IP)
#    - ~/repos/signage_control/hmdm-docker/volumes/hmdm-config/ROOT.xml  (sửa base.url)

# 3. Cập nhật lại đường dẫn tải APK trong Database PostgreSQL
docker exec hmdm-docker-postgresql-1 psql -U hmdm -d hmdm -c "UPDATE applicationversions SET url = REPLACE(url, '192.168.1.134', '192.168.1.150') WHERE url LIKE '%192.168.1.134%';"

# 4. Áp dụng cấu hình mới cho container
cd ~/repos/signage_control/hmdm-docker && docker compose up -d
```

### Cách đổi Server URL trên thiết bị Android khi đổi IP:
* **Cách 1 (Thao tác trực tiếp trên màn hình Signage — Dùng khi máy đã mất kết nối với IP cũ):**
  1. Tìm **biểu tượng Chiếc chìa khóa** hơi mờ ở sát mép phải, chính giữa theo chiều dọc màn hình.
  2. **Nhấn giữ lâu (Long-press)** vào biểu tượng Chiếc chìa khóa đó *(hoặc bấm nhanh liên tiếp **6 lần trong 3 giây**)*.
  3. Nhập mật khẩu quản trị (mặc định: **`12345678`**) -> bấm **OK**.
  4. Trong **Admin Panel**, bấm **Change Server URL** -> nhập `http://<IP-mới>` -> bấm **Save**.
* **Cách A (Chuyển hàng loạt từ Web Dashboard — Chỉ dùng khi thiết bị vẫn đang online với IP cũ):**
  1. Vào tab **Configurations** -> bấm sửa cấu hình đang gán cho thiết bị -> chọn tab con **`MDM Settings`**.
  2. Điền địa chỉ mới (ví dụ `http://192.168.1.150`) vào ô **`New server URL`** -> bấm **Save**. Thiết bị sẽ tự động chuyển sang server mới.
* **Cách B (Mở Admin Panel hoặc Reset app từ xa qua `Push messages` — Chỉ dùng khi thiết bị vẫn đang online):**
  * Vào **Functions -> Push messages -> New message** -> chọn thiết bị -> chọn 1 trong 2 lệnh:
    * **`adminPanel`** *(để trống Payload)*: Bật thẳng màn hình **Admin Panel** (có nút **Change Server URL**) lên thiết bị mà không cần nhấn biểu tượng Chìa khóa hay nhập mật khẩu.
    * **`clearAppData`** với Payload `{"pkg": "com.hmdm.launcher"}`: Tự động xóa dữ liệu app MDM để quay về màn hình hỏi Server URL ban đầu.

> **Important — Lưu ý nguyên tắc mạng:**
>
> Cả Cách A và Cách B ở trên chỉ thực hiện được khi Signage vẫn đang online (đèn xanh), kết nối với Server (tức là bạn gửi lệnh trước khi máy Mac đổi sang IP mới).
>
> Nếu máy Mac đã đổi sang IP mới rồi và Signage đang bị kẹt ở IP cũ (mất kết nối, đèn đỏ), Dashboard không thể gửi gói tin xuống thiết bị được nữa.
>
> Lúc đó trên Signage bắt buộc phải dùng Cách 1 (nhấn giữ icon Chiếc chìa khóa ở mép phải màn hình → Change Server URL). Cách này luôn chạy độc lập trên thiết bị kể cả khi mất mạng và không cần mở Settings của Android.
---

## 4. Các lệnh Build APK & Cấp quyền Device Owner qua ADB

```bash
# Build file APK Debug (trong thư mục hmdm-android-6.40)
cd ~/repos/signage_control/hmdm-android-6.40
./gradlew assembleOpensourceDebug
# File đầu ra: app/build/outputs/apk/opensource/debug/app-opensource-debug.apk

# Cấp quyền tối cao Device Owner cho app qua cáp USB / ADB (yêu cầu máy chưa đăng nhập tài khoản Google nào)
adb shell dpm set-device-owner com.hmdm.launcher/.AdminReceiver
```

---

## 5. Các tab ở trang quản trị
| Tab | Tên tiếng Việt | Chức năng chính |
|---|---|---|
| **Biểu tượng lưới** *(Tab đầu tiên)* | **Tổng quan (Summary)** | Dashboard biểu đồ thống kê nhanh: tổng số thiết bị, bao nhiêu máy đang Online/Offline, tình trạng cài đặt app hoặc cảnh báo lỗi. |
| **Devices** | **Thiết bị** | **Nơi thao tác hàng ngày:** Xem danh sách toàn bộ màn hình signage, trạng thái mạng (xanh/đỏ), địa chỉ IP, cấu hình đang chạy. Từ đây bạn có thể bấm **Reboot** (khởi động lại), **Lock** (khóa màn hình), xem thông số máy, hoặc gán cấu hình cho từng máy. |
| **Configurations** | **Cấu hình** | **Bộ não điều khiển luật chơi:** Bạn tạo các bộ luật (ví dụ: *"Cấu hình Sảnh Chính"*, *"Cấu hình Thang Máy"*). Bên trong bạn chọn: những app nào được cài/chạy, app nào tự bật khi mở nguồn (Run after install / Run at boot), hướng xoay màn hình (Ngang/Dọc), bật/tắt Wi-Fi/Bluetooth, khóa Kiosk Mode, chặn mở Cài đặt... |
| **Files** | **Tệp tin** | **Kho chứa media & file cấu hình:** Dùng để upload video quảng cáo (.mp4), hình ảnh (.jpg/.png), hoặc file .json/.txt lên server, sau đó chỉ định thư mục lưu trên Android (ví dụ `/Movies/Ads/`). Khi gán vào Configurations, các máy Signage sẽ tự động đồng bộ file về đúng thư mục đó. |
| **Settings** *(Menu xổ xuống)* | **Cài đặt hệ thống** | - **Language:** Có sẵn **Tiếng Việt** (bạn có thể vào đây đổi giao diện sang Tiếng Việt).<br>- **Default Design:** Đổi hình nền, màu sắc, cỡ chữ màn hình Launcher.<br>- **Groups / Users / Roles:** Gom nhóm thiết bị theo chi nhánh/khu vực, tạo thêm tài khoản quản lý. |
| **Functions** *(Menu xổ xuống)* | **Chức năng mở rộng (Plugins)** | - **Device Info:** Xem báo cáo chi tiết thông số phần cứng (CPU, RAM, bộ nhớ trong, pin/nguồn, MAC, mạng) gửi từ các máy.<br>- **Device Log:** Xem trực tiếp nhật ký hệ thống (logcat) của màn hình Android gửi về web để bắt lỗi từ xa mà không cần cắm cáp USB!<br>- **Push / Messaging:** Bắn tin nhắn hiển thị lên màn hình hoặc lên lịch gửi lệnh. |

---

## 6. Hướng dẫn sử dụng tab Applications (Kho ứng dụng nội bộ)

### A. Phân biệt 2 loại ứng dụng trong tab này
1. **App hệ thống (`Version = 0`, không có URL tải):**
   * Là các app đã có sẵn trong ROM của máy Android (*Chrome, Settings, Camera, Calculator...*).
   * App Android sẽ **không tải hay cài đặt** các app này. Chúng được liệt kê sẵn chỉ để bạn dùng trong **Configurations** nhằm **Allow / Block** (Cho phép / Chặn) hoặc ẩn/hiện icon trên màn hình chính.
2. **App cài đặt từ xa (`Version != 0`, có URL tải):**
   * Là các file `.apk` do bạn upload lên (hoặc của Headwind MDM). Hệ thống sẽ tạo link tải nội bộ (`http://<IP-Server>/files/...apk`) để đẩy xuống các thiết bị Signage tự tải về và cài đặt.

### B. Các bước thêm App mới (Upload APK)
1. Vào tab **Applications** -> bấm nút **Add**.
2. Tại mục **Type**, chọn loại ứng dụng:
   * **`Application`** *(Ứng dụng)*: Để upload file `.apk` hoặc `.xapk` cài đặt lên thiết bị.
   * **`Web-page`** *(Trang web)*: Để tạo một lối tắt (shortcut) trên màn hình chính mở thẳng vào một đường link web (hoặc `file:///storage/emulated/0/...`).
   * **`System action`** *(Hành động hệ thống)*: Tạo lối tắt gọi một Action hệ thống.
3. Upload file `.apk` từ máy tính của bạn -> Server sẽ tự động phân tích và điền sẵn **Package ID**, **Application Name**, và **Version**.
4. Kiểm tra mục **Native code** (Kiến trúc CPU): mặc định để **`None (universal APK)`** / **`Universal APK`** để chạy trên mọi chip ARMv7a/v8a/x86.
5. Thiết lập các tùy chọn hiển thị (**Show icon**, **Run after install**...) rồi bấm **Save** để lưu vào kho.

### C. Gắn App vào Configuration để đẩy xuống thiết bị *(Bắt buộc)*
> **Lưu ý:** Việc bấm **Add** ở tab `Applications` mới chỉ là đưa file APK vào kho chứa. Muốn thiết bị tự động tải về và cài đặt, bạn phải thực hiện ở tab **Configurations** (xem chi tiết ở mục 7 bên dưới):
1. Vào tab **Configurations** -> bấm biểu tượng sửa (bút chì) tại cấu hình đang gán cho thiết bị (ví dụ: `Common - Minimal`).
2. Chọn tab con **Applications** *(Ứng dụng)* -> tìm app bạn vừa thêm.
3. Tại cột **Actions**, đổi từ **`Do not install`** (*Không cài*) sang **`Install`** (*Cài đặt*).
4. Bấm **Save** -> Server sẽ gửi lệnh Push để thiết bị tải APK về và cài đặt ngay lập tức.

### D. Cập nhật phiên bản mới (Update Version), Gỡ cài đặt & Xóa khỏi kho
* **Nâng cấp app:** Tại tab **Applications**, bấm vào biểu tượng **Versions** ở dòng app cần nâng cấp -> upload file `.apk` mới hơn -> vào **Configurations** -> tab con **Applications**, bấm nút **`Upgrade`** (hoặc chọn phiên bản mới) rồi bấm **Save** để đồng loạt các máy tự cập nhật.
* **Gỡ bỏ app từ xa trên thiết bị:** Vào **Configurations** -> tab con **Applications** -> đổi cột **Actions** của app đó thành **`Delete`** (*Xóa*) rồi bấm **Save** *(yêu cầu thiết bị đã cấp quyền Device Owner hoặc Root để gỡ âm thầm)*.
* **Xóa app khỏi kho `Applications` (khi nút thùng rác bị làm mờ / disable):**
  * **Nguyên nhân:** Nút xóa bị khóa vì app đó đang được tham chiếu trong ít nhất một **Configuration** (kể cả khi đang đặt là `Install` hay `Delete`).
  * **Cách mở khóa để xóa:** Bấm vào biểu tượng **Danh sách (3 gạch ngang)** trên dòng app đó (hoặc vào **Configurations -> tab con `Applications`**) -> đổi Action của app thành **`Do not install`** (*Không cài*) -> bấm **Save** để ngắt liên kết. Sau đó quay lại tab **Applications**, nút thùng rác sẽ sáng lên để xóa.

### E. Hỗ trợ cài đặt trực tiếp file `.xapk` (Split APKs)
* **Thao tác:** Upload trực tiếp file `.xapk` vào tab **Applications** -> chọn **`Install`** trong **Configurations** y hệt như file `.apk` bình thường (không cần giải nén thủ công).
* **Cơ chế:** Trên thiết bị Android, MDM Agent sẽ tự động giải nén tất cả các file `.apk` con (như `base.apk`, `config.arm64_v8a.apk`...) bên trong gói `.xapk` và dùng `PackageInstaller` gộp cài đặt trong 1 phiên duy nhất.
* **Lưu ý:** Hỗ trợ tốt chuẩn Split APKs (tải từ APKPure, APKCombo...); không tự copy thư mục dữ liệu `Android/obb` nếu gói `.xapk` đó là game nặng.

---

## 7. Hướng dẫn sử dụng tab Configurations (Cấu hình thiết bị)

Đây là nơi tạo các "bộ luật" áp dụng cho từng nhóm thiết bị Signage.

### A. Các thao tác ở màn hình danh sách Configurations
* **Add** *(Thêm)*: Tạo mới một cấu hình trống hoàn toàn *(Lưu ý trong code: nếu tạo trống, bạn sẽ phải tự thêm lại toàn bộ các app hệ thống cần thiết)*.
* **Copy** *(Sao chép — biểu tượng 2 tờ giấy, **Khuyên dùng**)*: Nhân bản từ cấu hình mẫu `Common - Minimal` rồi sửa lại cho nhanh, tránh thiếu app hệ thống.
* **Change** *(Sửa — biểu tượng bút chì)*: Mở trình chỉnh sửa chi tiết cấu hình.
* **Delete** *(Xóa — biểu tượng thùng rác)*: Xóa cấu hình *(không thể xóa cấu hình mặc định id = 1)*.
* **QR code**: Tạo mã QR cài đặt tự động gắn kèm cấu hình này cho thiết bị mới tinh (Android 7+ có camera & GMS).

### B. Chi tiết 6 tab con bên trong trình chỉnh sửa Configuration
Khi bấm sửa (**Change**) một cấu hình, bạn sẽ thấy 6 tab con sau:

1. **`Common settings`** *(Cài đặt chung)*:
   * Tên cấu hình, mô tả, **Administrator password** *(mật khẩu vào Admin Panel trên Android, mặc định `12345678`)*.
   * Điều khiển phần cứng & hệ thống: **Bluetooth**, **Wi-Fi**, **GPS**, **Mobile data**, **USB storage**, **Brightness** *(độ sáng)*, **Screen timeout** *(thời gian tắt màn hình)*, **Orientation** *(xoay màn hình ngang/dọc)*, **Timezone** *(múi giờ)*.
   * Thiết lập cập nhật & Push: **Auto-Update**, **Push notifications** *(để `MQTT` khi chạy trong mạng LAN)*.
2. **`Design Settings`** *(Cài đặt thiết kế)*:
   * Tùy chỉnh màu nền, màu chữ, ảnh nền, kích thước icon (`SMALL`, `MEDIUM`, `LARGE`) và tiêu đề hiển thị trên Launcher.
3. **`Applications`** *(Ứng dụng)*:
   * Quản lý danh sách app chạy trên thiết bị. Có nút **Add**, bộ lọc **Search**, checkbox **Show system apps** *(Hiển thị ứng dụng hệ thống)*.
   * Cột **Actions** của từng app:
     * Với app có file APK tải về: **`Install`** (*Cài đặt*), **`Do not install`** (*Không cài*), **`Delete`** (*Xóa*).
       * *Phân biệt `Do not install` và `Delete`:*
         * **`Do not install`** (*Không cài*): Bỏ qua, không tải và không cài mới. Nếu máy **đã cài sẵn** app đó trước đây thì **vẫn giữ nguyên trên máy** (chỉ ẩn khỏi Launcher).
         * **`Delete`** (*Xóa*): Chủ động gửi lệnh ép thiết bị **gỡ cài đặt (`uninstall`)** app đó khỏi máy hoàn toàn.
     * Với System App (không có APK): **`Allow`** (*Cho phép*), **`Block`** (*Chặn*).
   * Tùy chọn thêm cho mỗi app: hiển thị icon trên màn hình, **Run at boot** *(tự chạy khi bật máy)*, và nút **Upgrade** khi có phiên bản mới.
4. **`MDM Settings`** *(Cài đặt MDM)*:
   * **Kiosk mode** *(Chế độ Kiosk)* & **Content application** *(chọn app chạy cố định duy nhất trên màn hình)*.
   * **Screen always on** *(Giữ màn hình luôn sáng — rất hữu ích cho Signage)*.
   * **Lock safe settings (WiFi, GPS, etc)**, **Lock the Power button**, **Permissive (unlocked) mode** *(tạm mở khóa chạy app tự do)*.
   * **New server URL**: Dùng để đẩy URL Server mới hàng loạt xuống các thiết bị khi chuyển server.
5. **`Applications Settings`** *(Cài đặt ứng dụng)*:
   * Truyền các biến cấu hình dạng Key-Value (`Name` / `Value` / `Type`) xuống cho các ứng dụng hỗ trợ nhận tham số từ MDM.
6. **`Files`** *(Tệp)*:
   * Gắn các file (video `.mp4`, ảnh, file cấu hình...) đã upload từ tab **Files** để thiết bị tự động tải về và lưu đúng đường dẫn trên bộ nhớ máy.

---

## 8. Hướng dẫn sử dụng tab Files (Kho tệp tin & Đồng bộ xuống thiết bị)

**Đúng, tab `Files` dùng để lưu trữ và truyền các tệp tin (video `.mp4`, hình ảnh, file `.obb`, file cấu hình `.json`/`.txt`...) từ Server xuống bộ nhớ thiết bị Android.**

### A. Các bước truyền File từ Server xuống thiết bị
1. **Bước 1 — Đưa file lên kho Server:**
   * Vào tab **Files** -> bấm nút **Add** *(Thêm)*.
   * Chọn upload file từ máy tính *(hoặc chọn **External** nếu muốn dán link tải trực tiếp từ bên ngoài mà không cần lưu file lên Server)*.
   * Tại ô **Path on device** *(Đường dẫn trên thiết bị)*: Điền đường dẫn lưu trên bộ nhớ máy Android.
     * Mặc định hệ thống tính từ gốc bộ nhớ ngoài (`/storage/emulated/0/`). Ví dụ: nhập `/Movies/quangcao.mp4` -> máy sẽ lưu tại `/storage/emulated/0/Movies/quangcao.mp4`.
     * *(Nâng cao)* Nếu đường dẫn bắt đầu bằng 2 dấu gạch chéo `//` (ví dụ `//data/local/...`), app sẽ ghi từ thư mục gốc `/` *(chỉ dùng khi máy có quyền Root/System)*.
   * Bấm **Save**.
2. **Bước 2 — Gắn file vào Configuration để đẩy xuống thiết bị *(Bắt buộc)*:**
   * Bấm vào biểu tượng **Danh sách (3 gạch ngang)** ngay trên dòng file vừa tạo *(hoặc vào **Configurations -> sửa cấu hình -> tab con `Files`**)* -> chọn cấu hình của thiết bị -> bấm **Save**.
   * Thiết bị đang online sẽ tự động tải file về đúng thư mục đã chỉ định.

### B. Các cột & tính năng hữu ích trong tab Files
* **External** *(Bên ngoài)*: Cho phép trỏ thẳng tới một đường dẫn URL bên ngoài thay vì upload file trực tiếp lên Server.
* **Variable** *(Biến)*: Dành cho các file cấu hình dạng text (`.json`, `.xml`, `.txt`). Khi bật tùy chọn này, app Android sẽ tự động thay thế các từ khóa `DEVICE_NUMBER`, `CUSTOM1`, `CUSTOM2`, `CUSTOM3` bằng thông số thật của từng thiết bị trước khi lưu file.
* **Cột Actions** *(Hành động)*:
  * **Mắt xích:** Copy đường dẫn URL tải trực tiếp của file.
  * **Danh sách:** Gắn nhanh file này vào các **Configurations**.
  * **Bút chì:** Sửa đường dẫn đích hoặc upload file mới đè lên *(khi cập nhật file mới cùng tên, các thiết bị sẽ tự động tải lại bản mới)*.
  * **Thùng rác:** Xóa file khỏi kho Server *(nếu nút này bị mờ thì cần gỡ file khỏi `Configurations` trước)*.
* **Xóa file trên bộ nhớ thiết bị từ xa:** Vào **Configurations -> tab con `Files`**, tick vào ô **Remove** *(Buộc xóa tệp khỏi bộ nhớ thiết bị)* ở dòng file muốn xóa rồi bấm **Save** -> thiết bị sẽ tự xóa file đó khỏi bộ nhớ máy.




