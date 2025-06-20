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

### 1. Sensor Interface Module

* Xử lý Raw sensor, debounce 100ms
* Output: `entry_detected`, `exit_detected`

### 2. Authentication Module

* Nhận `Card_id`, xử lý 2s xác minh
* Output: `auth_valid`, `auth_done`

### 3. Barrier Control Module

* Nhận lệnh từ FSM, mở/đóng trong 3s
* Output: `Entry_barrier`, `Exit_barrier`

### 4. Parking Management

* Đếm xe vào/ra, số lượng chỗ trống (0–100)
* Output: `Available_spaces`, `Parking_full`

### 5. Fee Calculation

* Nhận `entry_time`, `exit_time`, tính phí
* Output: `Fee_amount`, `fee_calc_done`

### 6. Display Control

* Điều khiển LED 7-segment theo `display_data`
* Output: `Segment_display`, `Digit_select`

### 7. LED Indicator

* Hiển thị trạng thái hệ thống qua LED
* Output: `Led_indicators`

### 8. Alarm Module

* Kích hoạt khi Emergency
* Output: `Alarm`

### 9. FSM Controller

* Quản lý tuần tự hoạt động hệ thống
* Điều khiển các lệnh: mở barrier, bắt đầu tính phí, ...
* Output: `entry_open_cmd`, `exit_open_cmd`, `display_data`, `system_state`

---

## 🔖 Kết luận

> Thiết kế hệ thống Parking thông minh bao gồm nhiều module đồng bộ, yêu cầu FSM kiểm soát, xử lý timer, giao tiếp LED, và phản hồi nhanh.

**Tiếp theo:** Thiết kế sơ đồ block hoặc viết Verilog module theo từng phần.
