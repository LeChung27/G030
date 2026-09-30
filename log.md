2: x/16xb buffer_tx
0x20000024 <buffer_tx>: 0x2b    0x43    0x4f    0x4e    0x4e    0x45    0x43    0x54 CONNECTED\0d\0a
0x2000002c <buffer_tx+8>:       0x45    0x44    0x0d    0x0a    0x00    0x00    0x00    0x00


0x20000024 <buffer_tx>: 0x2b    0x43    0x4f    0x4e    0x4e    0x45    0x43    0x54
0x2000002c <buffer_tx+8>:       0x45    0x44    0x0d    0x0a    0xaa    0x55    0x02    0xc8

---
##debug khi gui 400 byte
[ ] da debug xong UART
[ ] Doc data tu chip spi


```c
--------------------------
1: mode_cmd_data = 1 '\001'
2: x/16xb buffer_tx
0x20000024 <buffer_tx>: 0xaa    0x55    0x02    0xc8    0x63    0x64    0x00    0x01
0x2000002c <buffer_tx+8>:       0x02    0x03    0x04    0x05    0x06    0x07    0x08    0x09
3: Count_rxByte = 0 '\000'
4: /x buffer_tx[200] = 0x5d
(gdb) p /x buffer_tx[201]
$1 = 0x5e
(gdb) p /x buffer_tx[202]
$2 = 0x5f
(gdb) p /x buffer_tx[203]
$3 = 0x60
(gdb) p /x buffer_tx[204]
$4 = 0x0
(gdb) p /x buffer_tx[205]
$5 = 0x0
```

---
```c
----------------debug khi gui 423 byte----------

1: mode_cmd_data = 1 '\001'
2: x/16xb buffer_tx
0x20000024 <buffer_tx>: 0xaa    0x55    0x02    0x17    0x61    0x62    0x63    0x64
0x2000002c <buffer_tx+8>:       0x00    0x01    0x02    0x03    0x04    0x05    0x06    0x07
3: Count_rxByte = 0 '\000'
4: /x buffer_tx[200] = 0x5d
(gdb) p /x buffer_tx[201]
$6 = 0x5e
(gdb) p /x buffer_tx[202]
$7 = 0x5f
(gdb) p /x buffer_tx[203]
$8 = 0x60
(gdb) p /x buffer_tx[204]
...
(gdb) p /x buffer_tx[205]
$10 = 0x0
(gdb) p /x buffer_tx[22]
$11 = 0xe
(gdb) p /x buffer_tx[23]
$12 = 0xf
(gdb) p /x buffer_tx[26]
$13 = 0x12
(gdb) p /x buffer_tx[27]
$14 = 0x15
(gdb) q
```

---
##Cac buoc update webapp BLE
1. Đổi layout card + spacing
2. Đổi màu + font + button
3. Status BLE đẹp hơn
4. Toast ACK animation
5. Progress upload file
6. Background ảnh user blur nhẹ
7. Dark mode (optional)

###1. card UI
- **De xuat**
```text
┌────────────────────┐	Mau nen: #FFF7F5 hoac #F8F5FF
│  Ảnh nền mờ nhẹ    │	accent: pastel hồng / tím nhạt
│  [Tên thiết bị]    │	success ACK: xanh mint
│  🟢 Đã kết nối     │	Font: Inter, Nunito(mềm hơn,)
└────────────────────┘	border-radius: 18px; // card bo góc lớn:
┌ Điều khiển LED ────┐
│ [ON]    [OFF]      │
└────────────────────┘
┌ Âm nhạc ───────────┐
│ Chọn file WAV      │
│ Progress upload    │
│ [▶ Play] [⏸ Stop]  │
└────────────────────┘
┌ Pin thiết bị ──────┐	
│ 🔋 82% / 4.0V       │	warning pin yếu: cam nhẹ
└────────────────────┘
┌ Nhật ký ACK ───────┐
│ ✓ Upload thành công │
│ ✓ LED bật          │
└────────────────────┘
```

###2. Animation 
- Nhan nut: scale
```
button:active{
 transform: scale(0.96);
 transition: .1s;
}
```

---
- Khi nhan ACK : Hien toast nho :
✓ LED đã bật
✓ Gửi nhạc thành công
✓ Đồng bộ xong
+ Fade 2–3s rồi biến mất.
- Upload File: Progress bar mềm:
███████░░░ 72%

---
- Trang thai BLE :Có icon + màu.
```
🔴 Chưa kết nối
🟡 Đang kết nối
🟢 Đã kết nối
```

6. Background ảnh user blur nhẹ
- Cho user chọn ảnh nền, nhưng: blur nhẹ + opacity 10–20%
/home/chun/laptrinh/stm32/g0/Project/blink/log.md

---
Các thông tin đã tìm hiểu được:

---
Thứ tự khởi tạo chuẩn trong main()

 [System Clock (RCC)] *Cấu hình HSE/HSI, PLL, HCLK, PCLK*
        ||||
 [GPIO & Bus Clock Enable]  *Bật clock RCC cho GPIO, cấu hình mode, speed, alternate function (AF)*
        ||||
 [DMA & DMAMUX Init]    *Bật clock DMA, cấu hình các kênh (Channel), ưu tiên (Priority)*
        ||||            *hướng truyền dữ liệu khi DMA chưa chạy*
        ||||
 [Ngoại vi độc lập]     *(RTC, ADC, SPI, UART, I2C): Cấu hình tham số, kết nối DMA Request*
        ||||
 [Timer & PWM]          *Cấu hình Prescaler, ARR, Counter Mode, PWM Mode*
        ||||
 [NVIC (Ngắt) & Enable Trigger] *Bật ngắt hệ thống, bật DMA Channel, bật Ngoại vi và bật Timer Counter cuối cùng*


- <p>UART / SPI / I2C: CÓ THỂ bật ngay (LL_USART_Enable, LL_SPI_Enable) ở cuối hàm init của chính nó,
    miễn là cờ ngắt (IT) hoặc DMA Request của ngoại vi đó chưa bật</p>

- <p> PWM (Timer): KHÔNG NÊN bật xuất xung ngay. Chỉ dừng ở mức cấu hình thanh ghi. 
    Khi nào cần phát PWM mới gọi LL_TIM_CC_EnableChannel() và LL_TIM_EnableCounter() để tránh 
    tạo ra xung nhiễu (glitch) làm giật thiết bị ngoại vi lúc vừa bootup</p>

- <p>Nên tách riêng hàm MX_DMA_Init(): Không viết gộp DMA vào từng hàm ngoại vi. 
    Dòng STM32G0 sử dụng bộ ghép kênh DMAMUX chung cho toàn hệ thống, tách riêng giúp quản lý 
    phân bổ kênh dễ dàng và tránh xung đột</p>

- Thứ tự bắt buộc: DMA phải được cấu hình TRƯỚC ngoại vi.

- Quy tắc kích hoạt: Cấu hình DMA --> Cấu hình Ngoại vi --> Bật DMA Enable (LL_DMA_EnableChannel) 
    --> Bật Peripheral/Timer Enable (LL_TIM_EnableCounter)

