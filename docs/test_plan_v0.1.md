
# KỊCH BẢN KIỂM THỬ SƠ BỘ (TEST PLAN V0.1)

* **Dự án:** LoadingSystem – Module 3: Điều khiển Cơ điện tử 
* **Người thực hiện:** Bùi Thanh Quí (QA/Tester Lead)  
* **Mục tiêu giai đoạn v0.1:** Kiểm thử luồng dữ liệu hardcode (nhãn muỗi, tọa độ) từ tầng phát (Python Simulation / Pi Rust Client) xuống tầng chấp hành (ESP32 / Mô phỏng nhận lệnh) đảm bảo tính toàn vẹn dữ liệu 

---

## 1. PHẠM VI KIỂM THỬ (TEST SCOPE)
* **Kiểm thử đơn vị (Unit Test):** Kiểm tra cấu trúc gói tin logic, cơ chế tính toán Checksum XOR và cơ chế băm chuỗi (String Parsing) 
* **Kiểm thử tích hợp (Integration Test):** Kiểm tra khả năng truyền nhận thông suốt giữa tầng tính toán (Pi) và tầng chấp hành (MCU) qua giao thức mạng/giao tiếp.
* **Ràng buộc biên (Boundary & Negative Test):** Kiểm tra cách hệ thống xử lý khi dữ liệu đầu vào bị sai lệch, vượt ngưỡng (Ví dụ: loài muỗi không tồn tại).

---

## 2. KỊCH BẢN KIỂM THỬ CHI TIẾT (TEST CASES)

### 2.1. Nhóm Kiểm Thử Định Dạng Gói Tin & Mã Hóa (Mô phỏng của Nam)

#### **TC-01: Kiểm tra tính đúng đắn của cấu trúc dữ liệu và Checksum XOR**
* **Mục đích:** Đảm bảo gói tin `CmdPacket` sinh ra đủ 8 byte, đúng cấu trúc cấu hình và trường Checksum tính toán chính xác để tránh sai lệch lệnh điều khiển phần cứng.
* **Các bước thực hiện:**
    1. Gọi hàm tạo gói tin `CmdPacket(cmd=CmdType.MOVE_TO, param1=5, param2=0, data=0, seq=1)`.
    2. Xuất gói tin ra mảng bytes bằng hàm `to_bytes()`.
* **Kết quả kỳ vọng:**
    * Mảng bytes có độ dài chính xác bằng `8`.
    * Byte đầu tiên (Header) phải là `0xAA`.
    * Byte cuối cùng (Checksum) phải bằng kết quả phép XOR của 7 byte đầu tiên cộng lại.

#### **TC-02: Kiểm tra dữ liệu phản hồi giả lập (Status Packet)**
* **Mục đích:** Xác định trạng thái phản hồi từ tầng chấp hành (`StatusPacket`) được RPi giải mã chính xác các cờ lỗi (Alarm) hoặc cờ vị trí (`IN_POSITION`).
* **Các bước thực hiện:**
    1. Kích hoạt lệnh di chuyển `CmdType.MOVE_TO` với tham số `param1 = 3` (lọ số 3).
    2. Đọc gói tin phản hồi trả về từ hàm giả lập hệ thống.
* **Kết quả kỳ vọng:**
    * Trạng thái trả về chuyển từ `MOVING` sang `IN_POSITION`.
    * Vị trí lọ hiện tại ghi nhận chính xác là `3`.

---

### 2.2. Nhóm Kiểm Thử Truyền Nhận Không Dây & Giải Mã (Code Rust của Lộc)

#### **TC-03: Kiểm tra giải mã chuỗi định dạng (String Parsing Verification)**
* **Mục đích:** Đảm bảo firmware ESP32 trích xuất đúng chỉ số loài muỗi (0-11) từ chuỗi UDP nhận được theo định dạng `"FLAG:X"`.
* **Các bước thực hiện:**
    1. Gửi gói tin UDP chứa chuỗi ký tự `"FLAG:7"` từ Pi sang IP của ESP32.
    2. Quan sát log Terminal đầu ra của ESP32.
* **Kết quả kỳ vọng:**
    * ESP32 băm chuỗi thành công, nhận diện được chỉ số index là `7`.
    * Hệ thống chuyển đổi màu LED RGB thành công dựa trên cấu hình index tương ứng.

#### **TC-04: Kiểm thử giá trị biên và dữ liệu lỗi (Negative Test)**
* **Mục đích:** Đảm bảo vi điều khiển không bị treo (panic/crash) hoặc ra lệnh sai cho servo khi nhận phải dữ liệu rác từ môi trường truyền thông.
* **Các bước thực hiện:**
    1. Gửi chuỗi không đúng định dạng: `"FLAG:invalid"` hoặc `"HELLO:1"`.
    2. Gửi chỉ số vượt biên quy định của 12 loài muỗi: `"FLAG:15"`.
* **Kết quả kỳ vọng:**
    * Firmware ESP32 phải bắt được lỗi (Error handling), log ra màn hình cảnh báo dữ liệu lỗi.
    * Hệ thống giữ nguyên trạng thái an toàn cũ, tuyệt đối không crash hoặc đứng chương trình nhúng.

---

## 3. TIÊU CHÍ NGHIỆM THU (ACCEPTANCE CRITERIA V0.1)
* **Tỷ lệ pass:** 100% các Test Case cơ bản (TC-01, TC-02, TC-03) phải đạt trạng thái **SUCCESS**.
* **Xử lý ngoại lệ:** TC-04 phải chứng minh hệ thống từ chối gói tin lỗi một cách an toàn mà không làm gián đoạn luồng chạy của chương trình.
