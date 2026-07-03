# Loading System - Module 3: Mechatronics Control 🦟🥜

## 📌 Tổng Quan Dự Án
Repository này chứa toàn bộ mã nguồn, tài liệu kỹ thuật và kịch bản kiểm thử của **Module 3: Điều khiển Cơ điện tử (Mechatronics Control)** thuộc hệ thống **LoadingSystem**. Dự án được phát triển bởi đội ngũ Thực tập sinh Khoa Kỹ thuật Máy tính - Đại học Quốc gia TP.HCM (VNU-HCM) phối hợp cùng Phòng Giải pháp Cấu trúc AI & Tự động hóa (MKS Company) 

**LoadingSystem** là một hệ thống tích hợp công nghệ cao kết hợp giữa Máy học xử lý thị giác (Computer Vision), Không gian số thời gian thực (Digital Twin) và Cơ điện tử chính xác nhằm tự động hóa quy trình phân loại vật thể nhẹ (muỗi/côn trùng) và nông sản (hạt điều) trên băng chuyền công nghiệp.

### 🎯 Vai Trò Của Module 3
*   Thu nhận tín hiệu định vị, tọa độ và nhãn phân loại từ tầng thị giác AI (Jetson Nano / Raspberry Pi) 
*   Tính toán tốc độ băng tải, số xung cần phát và vị trí điều phối của cơ cấu chấp hành.
*   Điều khiển chính xác cụm cơ khí mâm xoay (hỗ trợ phân loại từ 10 đến 20 loài muỗi vào đúng lọ thủy tinh) với độ chính xác đạt **≥ 98%** (Đo hao hụt < 2%) 
*   Phân luồng hạt điều dựa trên trạng thái vỏ lụa để đưa vào phân hệ xử lý chuyên sâu.

---

## 🏗️ Kiến Trúc Hệ Thống (Module 3 Architecture)

Hệ thống điều khiển được thiết kế theo kiến trúc **Embedded nhúng thuần (Không dùng PLC)** để tối ưu chi phí và tăng tốc độ xử lý thời gian thực:

[ TẦNG 1: THỊ GIÁC (AI VISION) ]

└── Camera Công Nghiệp 4K (HIGH CLOUD) -> Video Stream (UVC 1080p@30fps)

└── Jetson Nano / Raspberry Pi 5 -> Xử lý mô hình AI (ONNX/TensorRT)

│

▼ Giao tiếp: UART / USB Serial / UDP Wifi

[ TẦNG 2: ĐIỀU KHIỂN TRUNG TÂM (EMBEDDED CONTROL) ]

└── Vi điều khiển (STM32 / ESP32) lập trình bằng ngôn ngữ RUST (ưu tiên) hoặc C

└── Chức năng: Nhận tọa độ/nhãn, tính toán toán học, phát xung điều khiển 

│

▼ Tín hiệu: Pulse/Direction (Max 4 Mpps Line Driver) & Digital I/O

[ TẦNG 3: CƠ CẤU CHẤP HÀNH (ACTUATORS) ]

└── Servo Driver CSD7_02BX1 (RS Automation, 200W)

└── Servo Motor + Đĩa xoay chứa lọ + Băng tải phân luồng


---

## 📁 Cấu Trúc Thư Mục Repository

```text
LoadingSystem_Module3/
├── .gitignore               # Cấu hình bỏ qua cache Python, build target của Rust 
├── README.md                # Tài liệu tổng quan dự án 
├── docs/                    # Tài liệu đặc tả kỹ thuật và QA
│   ├── requirement_v1.1.md  # Tài liệu yêu cầu kỹ thuật cập nhật sang hệ Embedded
│   └── test_plan_v0.1.md    # Kịch bản kiểm thử (Unit, Integration, E2E) của QA
├── src_pi_ai/               # Mô phỏng pipeline AI Vision & Giao thức đóng gói packet (Python)
│   └── loading_system_demo.py
├── src_pi_rust/             # Module Pi đóng vai trò Client gửi dữ liệu tính toán (Rust) 
│   ├── Cargo.toml
│   └── src/
├── src_esp32/               # Module vi điều khiển ESP32 thử nghiệm truyền nhận không dây (Rust) 
│   ├── Cargo.toml
│   └── src/
└── src_stm32/               # Mã nguồn firmware điều khiển servo driver (Chờ tích hợp v1.0)
```

---

## 🛠️ Quy Định Cộng Tác Lập Trình (Coding & Git Convention)

Để đảm bảo tính nghiêm ngặt của hệ thống chạy công nghiệp liên tục, toàn bộ thành viên trong nhóm phải tuân thủ các quy tắc sau:

1. **Coding Convention:**
* Mã nguồn Python tuân thủ tiêu chuẩn mã hóa `PEP 8` (Quét qua `pylint`).


* Mã nguồn Rust tuân thủ nghiêm ngặt `Formatting` mặc định và kiểm tra lỗi logic bằng `clippy` trước khi commit.


* Kiến trúc mã nguồn tách tầng độc lập giữa tầng ứng dụng (Application logic) và tầng giao tiếp driver phần cứng (Low-level Driver).




2. **Sử dụng AI hỗ trợ:** Khuyến khích sử dụng AI trợ giúp viết code nhưng khi Commit bắt buộc phải ghi rõ prompt đã dùng trong Message.


* *Ví dụ Commit Message:* `feat(calc): thêm hàm tính góc quay servo (code AI sinh bởi prompt: viết hàm tính góc chia đĩa xoay bằng Python)`.




3. **Quy trình nhánh (Branching Strategy):**
* `main`: Nhánh môi trường chạy thật ổn định, tuyệt đối không commit trực tiếp.
* `develop`: Nhánh tích hợp chung của cả nhóm.
* `feature/<tên_tính_năng>`: Nhánh phát triển độc lập của từng sub-team trước khi tạo Pull Request merge vào `develop`.



---

## ⏱️ Trạng Thái Phát Triển (Milestones - Giai đoạn Hiện tại)

Hiện tại hệ thống đang ở **Version 0.1 (Thử nghiệm v0.1)** với các chức năng đã hiện thực bao gồm:

* **Phía Pi (Python Simulation):** Hoàn thành giả lập kịch bản kết nối camera, xử lý mô hình AI, định nghĩa cấu trúc gói tin Binary SPI 8-byte có mã hóa Checksum XOR để gửi lệnh điều khiển mâm xoay (`MOVE_TO`, `SERVO_ON`, `HOME`).


* **Mạng Không Dây (Rust Pipeline):** Hoàn thành luồng truyền dữ liệu Socket UDP gửi mã Flag muỗi (0-11) từ Pi sang vi điều khiển và xử lý hiển thị dải màu LED RGB (WS2812) bằng RMT driver phần cứng để kiểm tra tính toàn vẹn dữ liệu.


---

## 📝 Bản Quyền & Người Đóng Góp

* **Tổ chức chịu trách nhiệm:** Mekong Solution Company (MKS).


* Đội ngũ phát triển (Nhóm Thực tập sinh CE - VNU-HCM):


* **Lộc, Nam, Đạt, Khánh:** Chịu trách nhiệm module tính toán dữ liệu & xử lý AI thị giác trên Pi/Jetson.


* **Khôi, Lâm, Duy:** Chịu trách nhiệm firmware nhúng (Rust/C) điều khiển chấp hành servo và đọc cảm biến.


* **Quí (QA/Tester & DevOps Lead):** Quản lý Git, lập kịch bản kiểm thử, tổ chức daily standup và review code.
 
