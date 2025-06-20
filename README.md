# Smart Parking System - README

## 🔍 1. Mục tiêu

Thiết kế hệ thống đỗ xe thông minh tự động hóa quy trình:

* Phát hiện xe đến/đi.
* Xác minh thẻ xe.
* Mở/đóng barrier.
* Cập nhật số chỗ trống.
* Tính phí đỗ xe.
* Hiển thị thông tin và trạng thái.
* Cảnh báo khi có sự cố hoặc khi hết chỗ.

---

## 📋 2. Bảng cổng I/O

| Port               | Direction | Bus size | Mô tả chức năng               |
| ------------------ | --------- | -------- | ----------------------------- |
| `Clk`              | In        | 1        | Clock hệ thống (50 MHz)       |
| `Reset`            | In        | 1        | Reset đồng bộ, active-high    |
| `Raw_entry_sensor` | In        | 1        | Cảm biến phát hiện xe vào     |
| `Raw_exit_sensor`  | In        | 1        | Cảm biến phát hiện xe ra      |
| `Card_id`          | In        | 4        | Thẻ xe để xác thực            |
| `Emergency`        | In        | 1        | Tín hiệu khẩn cấp             |
| `Entry_barrier`    | Out       | 1        | Điều khiển barrier cổng vào   |
| `Exit_barrier`     | Out       | 1        | Điều khiển barrier cổng ra    |
| `Available_spaces` | Out       | 6        | Hiển thị số chỗ trống (0–100) |
| `Segment_display`  | Out       | 8        | Giao tiếp LED 7-segment       |
| `Digit_select`     | Out       | 4        | Chọn digit hiển thị           |
| `Parking_full`     | Out       | 1        | Báo khi bãi xe đầy            |
| `Alarm`            | Out       | 1        | Báo động khi có khẩn cấp      |
| `Led_indicators`   | Out       | 4        | LED báo trạng thái hệ thống   |
| `Fee_amount`       | Out       | 8        | Hiển thị phí cần thanh toán   |

---

## 📀 3. Sơ đồ hoạt động bãi đỗ xe

* 2 cảm biến vị trí trước/sau barrier.
* Quy trình: Xe tới -> Xác thẻ -> Mở barrier -> Xe vào -> Cập nhật số chỗ.

---

## 📘 4. Thông số kỹ thuật (Specification)

| Yêu cầu                | Chi tiết                         |
| ---------------------- | -------------------------------- |
| Clock                  | 50 MHz                           |
| Max vehicles           | 100 xe                           |
| Barrier open time      | 3 giây                           |
| Card verification time | 2 giây                           |
| Fee rate               | 5đ/giờ, 50đ/ngày, 200đ/tuần      |
| Sensor debounce        | 100ms                            |
| Timeout detect vehicle | 10s                              |
| Emergency response     | < 500ms                          |
| Display refresh rate   | 1kHz                             |
| Data storage           | 1000 bản ghi thời gian xe vào/ra |

---

## 🌟 5. Yêu cầu chức năng

* Phát hiện xe đến
* Xác thẻ khi xe vào
* Mở barrier sau khi xác minh hợp lệ
* Ghi nhận xe đi qua
* Cập nhật số chỗ trống
* Báo đầy bãi xe
* Tính phí đỗ xe khi xe ra
* Hiển thị LED: số chỗ, phí, trạng thái
* Kích hoạt Alarm khi Emergency

---

## 🛠️ 6. Các module chính cần thiết kế

✅ 1. Sensor Interface Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk	Đồng hồ hệ thống
reset	Reset đồng bộ, khởi động lại module
Raw_entry_sensor	Tín hiệu cảm biến đầu vào xe (có thể nhiễu)
Raw_exit_sensor	Tín hiệu cảm biến đầu ra xe (có thể nhiễu)

📤 Đầu ra:
Tín hiệu	Vai trò
entry_detected	Tín hiệu đã phát hiện xe vào sau debounce
exit_detected	Tín hiệu đã phát hiện xe ra sau debounce

Ghi chú: Cần thêm logic debounce 100ms để lọc nhiễu từ cảm biến.

✅ 2. Authentication Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk	Đồng hồ
reset	Reset module
Card_id[3:0]	Dữ liệu ID của thẻ xe từ đầu đọc thẻ

📤 Đầu ra:
Tín hiệu	Vai trò
auth_valid	1 nếu thẻ hợp lệ
auth_done	1 khi kiểm tra thẻ hoàn tất (sau 2 giây)

Ghi chú: Có thể dùng bộ đếm thời gian 2 giây để tạo auth_done.

✅ 3. Barrier Control Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Hệ thống
entry_open_cmd	Lệnh mở barrier đầu vào
exit_open_cmd	Lệnh mở barrier đầu ra

📤 Đầu ra:
Tín hiệu	Vai trò
Entry_barrier	1: mở, 0: đóng
Exit_barrier	1: mở, 0: đóng

Ghi chú: Mỗi barrier mở trong 3 giây, cần timer.

✅ 4. Parking Management Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Điều khiển cơ bản
entry_confirmed	Xe đã đi vào (qua barrier + sensor)
exit_confirmed	Xe đã ra ngoài

📤 Đầu ra:
Tín hiệu	Vai trò
Available_spaces[5:0]	Số lượng chỗ trống (0–100)
Parking_full	Bật khi hết chỗ đậu

Ghi chú: Cần bộ đếm lên/xuống, giới hạn 0–100.

✅ 5. Fee Calculation Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Hệ thống
entry_time	Dữ liệu thời gian xe vào
exit_time	Dữ liệu thời gian xe ra
start_fee_calc	Kích hoạt quá trình tính phí

📤 Đầu ra:
Tín hiệu	Vai trò
Fee_amount[7:0]	Số tiền phí tính được
fee_calc_done	Cho biết quá trình tính phí đã xong

Ghi chú: Áp dụng biểu phí: Giờ (5đ), Ngày (50đ), Tuần (200đ).

✅ 6. Display Control Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Hệ thống
Digit_select[3:0]	Chọn digit trên 7-segment
display_data[7:0]	Dữ liệu cần hiển thị

📤 Đầu ra:
Tín hiệu	Vai trò
Segment_display[7:0]	Điều khiển các thanh led

Ghi chú: Có thể chạy kiểu multiplexing 1kHz để hiển thị từng digit.

✅ 7. LED Indicator Module
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Hệ thống
system_state[2:0]	Trạng thái hệ thống từ FSM

📤 Đầu ra:
Tín hiệu	Vai trò
Led_indicators[3:0]	Báo trạng thái: Normal, Full, Error, Processing

✅ 8. Alarm Module
📥 Đầu vào:
Tín hiệu	Vai trò
Emergency	Cảnh báo khẩn cấp từ người dùng

📤 Đầu ra:
Tín hiệu	Vai trò
Alarm	1 nếu có báo động

Ghi chú: Ưu tiên cao, phản hồi <500ms.

✅ 9. FSM Controller (Main Control)
📥 Đầu vào:
Tín hiệu	Vai trò
clk, reset	Điều khiển chính
entry_detected, exit_detected	Từ Sensor
auth_valid, auth_done	Từ Authentication
fee_calc_done	Từ Fee Module
Parking_full, Emergency	Tình trạng hệ thống

📤 Đầu ra:
Tín hiệu	Vai trò
entry_open_cmd, exit_open_cmd	Điều khiển barrier
entry_confirmed, exit_confirmed	Tín hiệu cập nhật số chỗ
start_fee_calc	Kích hoạt tính phí
display_data[7:0]	Dữ liệu gửi tới Display
system_state[2:0]	Gửi đến LED

---

## 🔖 Kết luận

> Thiết kế hệ thống Parking thông minh bao gồm nhiều module đồng bộ, yêu cầu FSM kiểm soát, xử lý timer, giao tiếp LED, và phản hồi nhanh.

**Tiếp theo:** Thiết kế sơ đồ block hoặc viết Verilog module theo từng phần.
