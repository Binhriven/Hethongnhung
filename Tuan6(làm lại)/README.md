#!/bin/bash

# Thiết lập các biến đường dẫn (Điều chỉnh nếu thư mục buildroot của bạn ở chỗ khác)
BR_DIR="$HOME/buildroot"
SRC_DIR="$HOME/blink_led_src"

echo "=========================================================="
echo "Hệ điều hành Nhúng - Setup Tự động Tuần 6"
echo "=========================================================="

# 1. Tạo thư mục mã nguồn và các file code C
echo "[1/4] Dang tao thu muc ma nguon tai $SRC_DIR..."
mkdir -p $SRC_DIR

cat << 'EOF' > $SRC_DIR/blink.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// Hàm hỗ trợ ghi file sysfs
void write_sysfs(const char *file_path, const char *value) {
    FILE *fp = fopen(file_path, "w");
    if (fp != NULL) {
        fprintf(fp, "%s", value);
        fclose(fp);
    } else {
        printf("Loi: Khong the mo %s\n", file_path);
    }
}

int main() {
    printf("=== CHUONG TRINH BLINK LED (TUAN 6) ===\n");

    // 1. Nap driver leds-gpio
    printf("Dang nap driver leds-gpio...\n");
    system("modprobe leds-gpio");
    usleep(200000); // Đợi 0.2s cho driver khởi động

    // 2. Tắt chế độ nhấp nháy tự động (trigger)
    printf("Dang tat trigger mac dinh...\n");
    write_sysfs("/sys/class/leds/beaglebone:green:usr3/trigger", "none");

    // 3. Vòng lặp Blink LED 10 lần
    printf("Bat dau chop tat LED...\n");
    for (int i = 0; i < 10; i++) {
        printf("Lan %d: LED ON\n", i + 1);
        write_sysfs("/sys/class/leds/beaglebone:green:usr3/brightness", "1");
        sleep(1);

        printf("Lan %d: LED OFF\n", i + 1);
        write_sysfs("/sys/class/leds/beaglebone:green:usr3/brightness", "0");
        sleep(1);
    }

    printf("Hoan thanh nhiem vu!\n");
    return 0;
}
EOF

cat << 'EOF' > $SRC_DIR/Makefile
all: blink

blink: blink.c
	$(CC) $(CFLAGS) -o blink blink.c

clean:
	rm -f blink
EOF

echo "Done: Da tao blink.c va Makefile."

# 2. Tạo Package trong Buildroot
echo "[2/4] Dang tao package blink_led trong Buildroot..."
mkdir -p $BR_DIR/package/blink_led

cat << 'EOF' > $BR_DIR/package/blink_led/Config.in
config BR2_PACKAGE_BLINK_LED
	bool "blink_led"
	help
	  Simple blink led application for BeagleBone Black (Week 6).
EOF

cat << 'EOF' > $BR_DIR/package/blink_led/blink_led.mk
BLINK_LED_VERSION = 1.0
BLINK_LED_SITE = $(HOME)/blink_led_src
BLINK_LED_SITE_METHOD = local

define BLINK_LED_BUILD_CMDS
	$(MAKE) $(TARGET_CONFIGURE_OPTS) -C $(@D) all
endef

define BLINK_LED_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/blink $(TARGET_DIR)/usr/bin/blink
endef

$(eval $(generic-package))
EOF

echo "Done: Da tao Config.in va blink_led.mk."

# 3. Khai báo gói vào menu tổng
echo "[3/4] Dang khai bao package vao menu Buildroot..."
if ! grep -q "blink_led" $BR_DIR/package/Config.in; then
    sed -i '/menu "Hardware handling"/a \	source "package/blink_led/Config.in"' $BR_DIR/package/Config.in
    echo "Done: Da them vao package/Config.in."
else
    echo "Skip: Package da ton tai trong Config.in."
fi

# 4. Hướng dẫn các bước tiếp theo
echo "=========================================================="
echo "KẾT QUẢ: Đã thiết lập xong các file cần thiết."
echo "BƯỚC TIẾP THEO:"
echo "1. Chạy lệnh: cd $BR_DIR && make menuconfig"
echo "2. Tìm 'blink_led' trong Hardware handling và bật nó lên (Dấu *)"
echo "3. Chạy lệnh: make blink_led-rebuild && make"
echo "4. Flash file sdcard.img vào thẻ nhớ."
echo "5. Trên BeagleBone Black, chạy lệnh 'blink' để kiểm tra."
echo "=========================================================="
