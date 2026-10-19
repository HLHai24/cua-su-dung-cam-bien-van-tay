# Khóa cửa vân tay ESP32 + AS608 + Servo

Dự án khóa cửa thông minh dùng **ESP32**, cảm biến vân tay **AS608**, màn hình **OLED SSD1306** và **servo quay liên tục (360°)** để mở/khóa cơ cấu cửa. Có còi cảnh báo khi quét sai nhiều lần và menu quản lý vân tay qua Serial Monitor.

> ⚠️ Dự án phục vụ học tập/thử nghiệm, chưa phải sản phẩm khóa an ninh thương mại.

## Tính năng

- Mở khóa bằng vân tay đã đăng ký (AS608 lưu tối đa 127 ID trong bộ nhớ của module).
- Servo quay mở khóa, giữ cửa mở 5 giây rồi tự quay ngược để khóa lại.
- Hiển thị trạng thái trên OLED 128x64 và in log chi tiết ra Serial (ID, độ tin cậy).
- Quét sai 3 lần liên tiếp: còi kêu 2 giây, sau đó reset bộ đếm.
- Nút nhấn vật lý để bật/tắt chức năng quét vân tay.
- Menu Serial: nạp vân tay, xóa theo ID, xóa tất cả (có xác nhận Y/N), xem số vân tay đã lưu.

## Phần cứng

| Linh kiện | Số lượng | Ghi chú |
|---|---|---|
| ESP32 DevKit | 1 | |
| Cảm biến vân tay AS608 | 1 | UART 57600 baud |
| OLED SSD1306 128x64 I2C | 1 | Địa chỉ `0x3C` |
| Servo quay liên tục (360°) | 1 | Điều khiển bằng độ rộng xung |
| Nút nhấn | 1 | Dùng `INPUT_PULLUP` |
| Còi buzzer | 1 | Còi chủ động, kích mức HIGH |
| Nguồn 5V cho servo | 1 | Nên dùng nguồn riêng |

## Sơ đồ nối dây

| Thiết bị | Chân thiết bị | Chân ESP32 |
|---|---|---|
| AS608 | TX | GPIO 16 (RX2) |
| AS608 | RX | GPIO 17 (TX2) |
| AS608 | VCC / GND | 3.3V / GND |
| OLED | SDA | GPIO 21 |
| OLED | SCL | GPIO 22 |
| OLED | VCC / GND | 3.3V / GND |
| Servo | Tín hiệu | GPIO 18 |
| Servo | VCC / GND | 5V / GND |
| Nút nhấn | Một chân / chân còn lại | GPIO 27 / GND |
| Buzzer | Tín hiệu (+) | GPIO 26 |
| Buzzer | GND (-) | GND |

Lưu ý:
- Chân TX của AS608 nối vào RX của ESP32 và ngược lại.
- Nếu cấp nguồn servo riêng, **phải nối chung GND** với ESP32.
- Servo tiêu thụ dòng lớn khi quay, nên tránh lấy trực tiếp từ chân 5V của ESP32 nếu nguồn USB yếu.
- Điện áp của AS608 theo loại module (thường 3.3V–5V), kiểm tra datasheet module của bạn.

## Thư viện cần cài

Cài qua **Arduino IDE → Library Manager**:

- `Adafruit GFX Library`
- `Adafruit SSD1306`
- `Adafruit Fingerprint Sensor Library`
- `ESP32Servo`

Board: **ESP32** (*esp32 by Espressif Systems*).

## Cấu hình servo

Code điều khiển servo như servo 360° bằng độ rộng xung (µs):

```cpp
#define SERVO_STOP   1500   // Đứng yên
#define OPEN_SPEED   1300   // Quay mở khóa
#define CLOSE_SPEED  1700   // Quay khóa lại

#define OPEN_TIME    3000   // ms quay để mở
#define CLOSE_TIME   3000   // ms quay để khóa
#define DOOR_OPEN_TIME 5000 // ms giữ cửa mở
```

- Nếu servo quay **ngược chiều** mong muốn, hoán đổi `OPEN_SPEED` và `CLOSE_SPEED`.
- Nếu servo **không đứng yên** ở trạng thái nghỉ, tinh chỉnh `SERVO_STOP` quanh 1500 (ví dụ 1480–1520).
- Điều chỉnh `OPEN_TIME` / `CLOSE_TIME` cho phù hợp với cơ cấu khóa thực tế.

## Cách sử dụng

1. Nối dây theo sơ đồ và nạp code cho ESP32.
2. Mở Serial Monitor, baud **115200**, chế độ line ending nên chọn *No line ending* hoặc *Newline*.
3. Hệ thống kiểm tra kết nối AS608 và hiển thị số vân tay đã lưu.
4. Nạp ít nhất một vân tay bằng menu (xem bên dưới).
5. Đặt ngón tay lên cảm biến: vân tay đúng thì servo mở khóa, giữ 5 giây rồi tự khóa lại.

### Menu Serial

| Phím | Chức năng |
|---|---|
| `1` | Nạp vân tay: nhập ID (1–127), đặt ngón tay 2 lần theo hướng dẫn trên OLED |
| `2` | Xóa vân tay theo ID |
| `3` | Xóa toàn bộ vân tay (xác nhận `Y` / hủy `N`) |
| `4` | Xem số vân tay đã lưu |

### Nút nhấn

Mỗi lần nhấn sẽ **bật/tắt chức năng quét vân tay** (có chống dội phím 300 ms). Khi tắt, OLED hiển thị `QUET VAN TAY - DANG TAT` và cửa sẽ không mở bằng vân tay.

### Cảnh báo sai vân tay

| Lần sai | Phản hồi |
|---|---|
| 1, 2 | OLED hiện `VAN TAY SAI! LAN 1/2` |
| 3 | OLED hiện `CANH BAO!`, còi kêu 2 giây, bộ đếm được reset |

## Xử lý sự cố

- **`AS608 KHONG KET NOI!`**: kiểm tra lại TX/RX (thường bị nối ngược), nguồn và GND; module phải dùng baud 57600.
- **Không tìm thấy OLED**: kiểm tra SDA/SCL, nguồn và địa chỉ I2C (`0x3C` hoặc `0x3D`).
- **Servo quay liên tục khi khởi động**: chỉnh lại `SERVO_STOP` cho đúng điểm dừng của servo.
- **Vân tay đúng nhưng báo sai**: nạp lại vân tay, lau sạch kính cảm biến, đặt ngón tay chắc và đủ diện tích.
- **Nạp vân tay thất bại "Khong trung khop"**: hai lần quét không giống nhau, hãy đặt đúng cùng một ngón và cùng vị trí.

## Hạn chế hiện tại

- Servo và các thông báo dùng `delay()` nên hệ thống không phản hồi nút/menu trong lúc đang mở/khóa cửa.
- Màn hình OLED dùng font mặc định, không hiển thị được chữ tiếng Việt có dấu; nên dùng tiếng Việt không dấu cho các dòng hiển thị.
- Chưa có cảm biến xác nhận trạng thái cửa (đóng/mở thực tế).

## Cấu trúc dự án

```
.
├── khoa_van_tay.ino   # Mã nguồn chính
├── LICENSE
└── README.md
```

## Giấy phép

Phát hành theo giấy phép [MIT](LICENSE) (thay đổi nếu bạn dùng giấy phép khác).

## Tác giả

**LeHai**
