# 📘 Tuần 6 : Bài tập HDH Nhúng - Ứng dụng tổng hợp

---

## 🧩 Bài tập 1 : Giao tiếp với Device Driver từ ứng dụng

1. Bật BBB đã build hệ điều hành ở bài 5 và bật minicom  

2. "Đánh thức” chân GPIO (Lệnh Export):

```bash
echo 60 > /sys/class/gpio/export
```

3. Thiết lập chân GPIO đó ở chế độ output  

```bash
echo out > /sys/class/gpio/gpio60/direction
```

4. Bật sáng bóng LED (ON)

```bash
echo 1 > /sys/class/gpio/gpio60/value
```

5. Đối với bóng LED trên bo mạch  

👉 Ảnh:  
![Bai1](https://raw.githubusercontent.com/Binhriven/Hethongnhung/main/Tuan6_lam_lai/1.jpg)

---

## 🧩 Bài tập 2 : Viết chương trình C/C++ và đóng gói vào Buildroot

### 🔹 Bước 1: Viết mã nguồn C và Makefile

#### Code C:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    system("echo 60 > /sys/class/gpio/export");
    system("echo out > /sys/class/gpio/gpio60/direction");

    while(1) {
        system("echo 1 > /sys/class/gpio/gpio60/value");
        sleep(1);
        system("echo 0 > /sys/class/gpio/gpio60/value");
        sleep(1);
    }

    return 0;
}
```

#### Makefile:
```makefile
all:
	$(CC) blink_led.c -o blink_led

clean:
	rm -f blink_led
```

---

### 🔹 Bước 2: Tạo Gói (Package) trong Buildroot

```bash
nano package/blink_led/blink_led.mk
```

#### Nội dung:
```makefile
BLINK_LED_VERSION = 1.0
BLINK_LED_SITE = $(TOPDIR)/package/blink_led
BLINK_LED_SITE_METHOD = local

define BLINK_LED_BUILD_CMDS
	$(MAKE) CC="$(TARGET_CC)" -C $(@D)
endef

define BLINK_LED_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/blink_led $(TARGET_DIR)/usr/bin/blink_led
endef

$(eval $(generic-package))
```

---

### 🔹 Khai báo gói vào menu tổng

👉 Ảnh:  
![Bai4](https://raw.githubusercontent.com/Binhriven/Hethongnhung/main/Tuan6_lam_lai/4.jpg)

---

### 🔹 Kích hoạt trong menuconfig

```bash
make menuconfig
```

👉 Ảnh:  
![Bai5](https://raw.githubusercontent.com/Binhriven/Hethongnhung/main/Tuan6_lam_lai/5.jpg)

---

### 🔹 Kết quả build

👉 Ảnh:  
![Bai6](https://raw.githubusercontent.com/Binhriven/Hethongnhung/main/Tuan6_lam_lai/6.jpg)

---

## 🧩 Bài tập 3 : Tự khởi chạy

### 🔹 Tạo script

```bash
cat > /etc/init.d/S99blink << 'EOF'
#!/bin/sh
/usr/bin/blink_led &
EOF
```

---

### 🔹 Cấp quyền

```bash
chmod +x /etc/init.d/S99blink
```

---

### 🔹 Kết quả

👉 Ảnh:  
![Bai11](https://raw.githubusercontent.com/Binhriven/Hethongnhung/main/Tuan6_lam_lai/11.jpg)
