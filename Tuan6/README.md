# Tuần 6: Linux LED Driver và Blink Application

## 1. Mục tiêu

* Tìm hiểu cách viết **Linux Device Driver**
* Hiểu cơ chế **giao tiếp giữa User Space và Kernel Space**
* Điều khiển LED thông qua **device file**
* Tích hợp chương trình vào **Buildroot**
* Boot hệ thống trên **BeagleBone Black**

---

# Bài 1: Viết Linux LED Driver

## 1.1 Mô hình hệ thống

```
User Application
      │
      ▼
Device File (/dev/led_test)
      │
      ▼
Kernel Driver
      │
      ▼
GPIO
      │
      ▼
LED
```

Driver trong Kernel sẽ điều khiển GPIO để bật/tắt LED.

---

## 1.2 Device File

Driver sẽ tạo device:

```
/dev/led_test
```

User Application sẽ ghi dữ liệu vào device này để điều khiển LED.

---

## 1.3 Nguyên lý hoạt động

| Giá trị ghi | Chức năng |
| ----------- | --------- |
| 1           | Bật LED   |
| 0           | Tắt LED   |

---

## 1.4 Load Driver

Sau khi biên dịch driver:

```
insmod led_driver.ko
```

Kiểm tra module đã load:

```
lsmod
```

Kiểm tra device file:

```
ls /dev/led_test
```

---

# Bài 2: Chương trình Blink LED (User Application)

## 2.1 Mục tiêu

* Viết chương trình **User Space**
* Giao tiếp với **Device Driver**
* Điều khiển LED bật tắt liên tục

---

## 2.2 File chương trình

```
blink_app.c
```

---

## 2.3 Code chương trình

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

#define DEVICE "/dev/led_test"

int main()
{
    int fd;

    fd = open(DEVICE, O_WRONLY);
    if(fd < 0)
    {
        printf("Khong mo duoc driver\n");
        return -1;
    }

    printf("Blink LED\n");

    while(1)
    {
        write(fd,"1",1);
        printf("LED ON\n");
        sleep(1);

        write(fd,"0",1);
        printf("LED OFF\n");
        sleep(1);
    }

    close(fd);

    return 0;
}
```

---

## 2.4 Hoạt động của chương trình

Chương trình thực hiện:

1. Mở device `/dev/led_test`
2. Ghi `"1"` để bật LED
3. Ghi `"0"` để tắt LED
4. Lặp lại mỗi 1 giây

---

# Bài 3: Build hệ thống bằng Buildroot và chạy trên BeagleBone Black

## 3.1 Build hệ thống

Tại thư mục Buildroot:

```
make
```

Sau khi build thành công:

```
output/images/sdcard.img
```

---

## 3.2 Flash Image vào SD Card

Kiểm tra thiết bị:

```
lsblk
```

Unmount SD card:

```
sudo umount /dev/sdb1
sudo umount /dev/sdb2
```

Flash image:

```
sudo dd if=output/images/sdcard.img of=/dev/sdb bs=4M status=progress
sync
```

---

## 3.3 Boot hệ thống

Kết nối UART:

```
sudo picocom -b 115200 /dev/ttyUSB0
```

Sau khi boot:

```
Welcome to Buildroot
```

---

## 3.4 Chạy chương trình Blink LED

Load driver:

```
insmod led_driver.ko
```

Chạy ứng dụng:

```
./blink_app
```

Kết quả trên terminal:

```
Blink LED
LED ON
LED OFF
LED ON
LED OFF
```

LED trên board **BeagleBone Black** sẽ nháy liên tục.

---

# Kết quả thực nghiệm

## Boot hệ thống

![boot](boot.jpg)

---

## Log UART

![uart](uart.jpg)

---

## LED hoạt động

![led](led.jpg)

---

# Kết luận

Qua bài thực hành này đã:

* Viết thành công **Linux Device Driver**
* Tạo device file `/dev/led_test`
* Viết **User Application** điều khiển LED
* Tích hợp chương trình vào **Buildroot**
* Boot hệ thống thành công trên **BeagleBone Black**
* Điều khiển LED nháy thông qua driver
