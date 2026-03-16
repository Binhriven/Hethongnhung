# Tuần 6: Linux LED Driver và Blink Application

## 1. Mục tiêu

* Tìm hiểu cách viết **Linux Device Driver**
* Giao tiếp giữa **User Space và Kernel Space**
* Điều khiển LED thông qua **device file**
* Tích hợp chương trình vào **Buildroot**
* Boot hệ thống trên **BeagleBone Black**

---

# 2. Mô hình hệ thống

User Application
↓
Device File `/dev/led_test`
↓
Kernel Driver
↓
GPIO
↓
LED

---

# 3. Driver LED

Driver tạo device:

```
/dev/led_test
```

Nguyên lý hoạt động:

| Giá trị ghi | Chức năng |
| ----------- | --------- |
| "1"         | Bật LED   |
| "0"         | Tắt LED   |

Load driver:

```
insmod led_driver.ko
```

Kiểm tra module:

```
lsmod
```

---

# 4. Chương trình Blink LED

File:

```
blink_app.c
```

Code chương trình:

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

# 5. Build hệ thống bằng Buildroot

Build project:

```
make
```

Sau khi build xong:

```
output/images/sdcard.img
```

---

# 6. Flash image vào SD Card

Kiểm tra thiết bị:

```
lsblk
```

Unmount SD:

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

# 7. Boot hệ thống

Kết nối UART:

```
sudo picocom -b 115200 /dev/ttyUSB0
```

Sau khi boot:

```
Welcome to Buildroot
```

---

# 8. Kết quả

LED nháy liên tục:

```
LED ON
LED OFF
LED ON
LED OFF
```

---

# 9. Hình ảnh kết quả

## Boot hệ thống

![boot](1.jpg)

## Log UART

![uart](2.jpg)

## LED hoạt động

![led](3.jpg)

---

# 10. Kết luận

Qua bài thực hành này đã:

* Viết thành công **Linux Device Driver**
* Tạo device file `/dev/led_test`
* Giao tiếp giữa **User Space và Kernel Space**
* Tích hợp ứng dụng vào **Buildroot**
* Chạy thành công trên **BeagleBone Black**

