📌 BT05 – Biên dịch Ứng dụng và Thư viện với Buildroot
📖 Giới thiệu

Bài tập này hướng dẫn cách sử dụng Buildroot để:

Biên dịch ứng dụng với thư viện có sẵn (cJSON)

Tự tạo thư viện bằng C (static & dynamic)

Đóng gói (package) thư viện và ứng dụng vào hệ điều hành

Hệ thống sử dụng: BeagleBone Black (BBB)

🎯 Mục tiêu

Hiểu cơ chế cross-compile

Làm việc với thư viện tĩnh (.a) và động (.so)

Sử dụng sysroot trong Buildroot

Tạo package tùy chỉnh

Tích hợp ứng dụng vào hệ thống Linux nhúng

🧪 Bài 1: Biên dịch ứng dụng với thư viện cJSON
🔧 Bước 1: Bật cJSON trong Buildroot
make menuconfig

Đi theo đường dẫn:

Target packages ---> Libraries ---> JSON/XML ---> [*] cJSON

Sau đó build lại:

make

📌 Sau khi build:

Thư viện .so:

output/target/usr/lib

File header .h:

output/host/arm-buildroot-linux-gnueabihf/sysroot/usr/include/cjson
📝 Bước 2: Viết chương trình HelloJSON.c
#include <stdio.h>
#include <cjson/cJSON.h>

int main() {
    cJSON *json = cJSON_CreateObject();
    cJSON_AddStringToObject(json, "name", "Buildroot");

    printf("%s\n", cJSON_Print(json));
    cJSON_Delete(json);
    return 0;
}
⚙️ Bước 3: Cross-compile
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc HelloJSON.c -o HelloJSON \
-I./output/host/arm-buildroot-linux-gnueabihf/sysroot/usr/include/cjson \
-L./output/host/arm-buildroot-linux-gnueabihf/sysroot/usr/lib \
-lcjson

📌 Giải thích:

-I: đường dẫn header

-L: đường dẫn thư viện

-lcjson: link với cJSON

🚀 Bước 4: Copy sang BBB
scp HelloJSON root@<IP_BBB>:/root/

Nếu thiếu thư viện:

scp output/target/usr/lib/libcjson.so* root@<IP_BBB>:/usr/lib/
▶️ Bước 5: Chạy chương trình
./HelloJSON
🧪 Bài 2: Tạo thư viện cá nhân
📌 Bước 1: Tạo thư viện

Ví dụ:

libptit.h

libptit.c

⚙️ Bước 2: Biên dịch thư viện
🔹 Thư viện tĩnh (.a)
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc -c libptit.c -o libptit.o
./output/host/bin/arm-buildroot-linux-gnueabihf-ar rcs libptit.a libptit.o
🔹 Thư viện động (.so)
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc -fPIC -c libptit.c -o libptit.o
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc -shared -o libptit.so libptit.o

📌 -fPIC: cho phép thư viện chạy ở nhiều vị trí bộ nhớ

📂 Bước 3: Đưa vào sysroot
SYSROOT=$(pwd)/output/host/arm-buildroot-linux-gnueabihf/sysroot  

cp libptit.h $SYSROOT/usr/include/
cp libptit.a libptit.so $SYSROOT/usr/lib/
🧪 Bước 4: Viết app sử dụng thư viện
🔹 Dùng static
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc app_test.c -o app_static -lptit -static
🔹 Dùng dynamic
./output/host/bin/arm-buildroot-linux-gnueabihf-gcc app_test.c -o app_dynamic -lptit
🚀 Bước 5: Chạy trên BBB
scp app_static app_dynamic root@<IP_BBB>:/root/
scp libptit.so root@<IP_BBB>:/usr/lib/

Chạy:

./app_static
./app_dynamic
🔍 Bước 6: So sánh
ls -lh

Kiểm tra dependency:

./output/host/bin/arm-buildroot-linux-gnueabihf-readelf -d app_dynamic | grep NEEDED

📌 Kết luận:

Static: file lớn, chạy độc lập

Dynamic: nhỏ hơn, cần .so

🧪 Bài 3: Tích hợp vào Buildroot
📦 Bước 1: Tạo package libkmt
mkdir -p package/libkmt/src
📄 Config.in
config BR2_PACKAGE_LIBKMT
    bool "libkmt"
📄 libkmt.mk
LIBKMT_VERSION = 1.0
LIBKMT_SITE = $(TOPDIR)/package/libkmt/src
LIBKMT_SITE_METHOD = local
LIBKMT_INSTALL_STAGING = YES

define LIBKMT_BUILD_CMDS
    $(TARGET_CC) $(TARGET_CFLAGS) -fPIC -c $(@D)/kmt.c -o $(@D)/kmt.o
    $(TARGET_AR) rcs $(@D)/libkmt.a $(@D)/kmt.o
    $(TARGET_CC) -shared $(TARGET_CFLAGS) -o $(@D)/libkmt.so $(@D)/kmt.o
endef

define LIBKMT_INSTALL_STAGING_CMDS
    $(INSTALL) -D -m 0644 $(@D)/kmt.h $(STAGING_DIR)/usr/include/kmt.h
    $(INSTALL) -D -m 0755 $(@D)/libkmt.a $(STAGING_DIR)/usr/lib/libkmt.a
    $(INSTALL) -D -m 0755 $(@D)/libkmt.so $(STAGING_DIR)/usr/lib/libkmt.so
endef

define LIBKMT_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/libkmt.so $(TARGET_DIR)/usr/lib/libkmt.so
endef

$(eval $(generic-package))
🧩 Bước 2: Tạo ứng dụng combined_app
mkdir -p package/combined_app/src
📄 combined_app.c
#include <stdio.h>
#include <cjson/cJSON.h>
#include <kmt.h>

int main() {
    const char *data = "{\"val1\": 15, \"val2\": 25}";
    cJSON *json = cJSON_Parse(data);

    int v1 = cJSON_GetObjectItem(json, "val1")->valueint;
    int v2 = cJSON_GetObjectItem(json, "val2")->valueint;

    printf("val1=%d, val2=%d\n", v1, v2);
    printf("Tong: %d\n", tinh_tong(v1, v2));

    cJSON_Delete(json);
    return 0;
}
📦 Bước 3: Package cho app
📄 Config.in
config BR2_PACKAGE_COMBINED_APP
    bool "combined_app"
    select BR2_PACKAGE_CJSON
    select BR2_PACKAGE_LIBKMT
📄 combined_app.mk
COMBINED_APP_VERSION = 1.0
COMBINED_APP_SITE = $(TOPDIR)/package/combined_app/src
COMBINED_APP_SITE_METHOD = local
COMBINED_APP_DEPENDENCIES = cjson libkmt

define COMBINED_APP_BUILD_CMDS
    $(TARGET_CC) $(TARGET_CFLAGS) $(@D)/combined_app.c -o $(@D)/combined_app \
    $(TARGET_LDFLAGS) -lcjson -lkmt
endef

define COMBINED_APP_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/combined_app $(TARGET_DIR)/usr/bin/combined_app
endef

$(eval $(generic-package))
⚙️ Bước 4: Kích hoạt package

Sửa file:

package/Config.in

Thêm:

menu "Custom Apps"
    source "package/libkmt/Config.in"
    source "package/combined_app/Config.in"
endmenu

Chạy:

make menuconfig

Chọn:

Custom Apps ---> [*] combined_app
🏗️ Build lại hệ thống
make
💾 Nạp vào thẻ nhớ
sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M status=progress conv=fsync
▶️ Chạy trên BBB
combined_app

📌 Kết quả:

Không cần copy .so

Không cần cấu hình thêm

App chạy trực tiếp

🎓 Tổng kết

Qua bài này bạn đã:

Cross compile ứng dụng

Tạo thư viện .a và .so

Hiểu sysroot Buildroot

Tạo package riêng

Tích hợp app vào hệ điều hành

