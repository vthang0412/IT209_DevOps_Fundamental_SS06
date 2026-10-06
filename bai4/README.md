# Bài 4: Quản lý tiến trình nền với `nohup` và tín hiệu `kill`

## 1. Mục tiêu

-   Tạo shell script chạy liên tục dưới nền.
-   Sử dụng `nohup` kết hợp với `&` để tiến trình tiếp tục hoạt động khi
    ngắt kết nối SSH.
-   Kiểm tra tiến trình bằng `pgrep` và `ps`.
-   Kiểm tra file log được cập nhật định kỳ.
-   Dừng tiến trình bằng `SIGTERM (15)`.
-   Chỉ sử dụng `SIGKILL (9)` khi `SIGTERM` không thể dừng tiến trình.

------------------------------------------------------------------------

## 2. Môi trường thực hiện

-   Máy chủ: VPS Cloud AZVPS
-   User: `devops`
-   Host: `azvps-tm8ef3mbd`
-   Hệ điều hành: Ubuntu
-   Thư mục thực hiện: `/home/devops`
-   File script: `loop-monitor.sh`
-   File log: `/tmp/monitor.log`

------------------------------------------------------------------------

## 3. Tạo shell script

Tạo file `loop-monitor.sh`:

``` bash
cat <<'EOF' > loop-monitor.sh
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
EOF
```

Kiểm tra nội dung:

``` bash
cat loop-monitor.sh
```

Kết quả:

``` text
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

------------------------------------------------------------------------

## 4. Cấp quyền thực thi

``` bash
chmod +x loop-monitor.sh
```

Kiểm tra:

``` bash
ls -l loop-monitor.sh
```

Kết quả thực tế:

``` text
-rwxrwxr-x 1 devops devops 97 Oct 6 20:13 loop-monitor.sh
```

File đã có quyền thực thi.

------------------------------------------------------------------------

## 5. Khởi chạy bằng `nohup`

Xóa log cũ trước khi chạy:

``` bash
rm -f /tmp/monitor.log
```

Khởi chạy script ở chế độ nền:

``` bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Kết quả thực tế:

``` text
[1] 48413
```

Trong đó:

``` text
PID = 48413
```

Giải thích:

-   `nohup`: giúp tiến trình không bị kết thúc khi phiên SSH/Terminal bị
    ngắt.
-   `&`: đưa tiến trình chạy ở chế độ background.
-   `> /dev/null`: bỏ qua output tiêu chuẩn.
-   `2>&1`: chuyển cả lỗi chuẩn vào cùng nơi với output chuẩn.

------------------------------------------------------------------------

## 6. Kiểm tra PID của tiến trình

Sử dụng:

``` bash
pgrep -f loop-monitor.sh
```

Kết quả thực tế:

``` text
48413
```

Kiểm tra chi tiết bằng:

``` bash
ps aux | grep loop-monitor.sh
```

Kết quả thực tế:

``` text
devops     48413  0.0  0.1  7340  3712 pts/0    S    20:14   0:00 /bin/bash ./loop-monitor.sh
devops     48424  0.0  0.1  6680  2304 pts/0    S+   20:14   0:00 grep --color=auto loop-monitor.sh
```

Dòng có PID `48413` là tiến trình `loop-monitor.sh`.

Dòng `grep --color=auto loop-monitor.sh` chỉ là lệnh `grep` dùng để tìm
kiếm và không phải tiến trình cần quản lý.

------------------------------------------------------------------------

## 7. Kiểm tra file log

Kiểm tra 10 dòng cuối:

``` bash
tail -n 10 /tmp/monitor.log
```

Kết quả thực tế:

``` text
System time: Tue Oct 6 08:15:53 PM +07 2026
System time: Tue Oct 6 08:15:58 PM +07 2026
System time: Tue Oct 6 08:16:03 PM +07 2026
System time: Tue Oct 6 08:16:08 PM +07 2026
System time: Tue Oct 6 08:16:13 PM +07 2026
System time: Tue Oct 6 08:16:18 PM +07 2026
System time: Tue Oct 6 08:16:23 PM +07 2026
System time: Tue Oct 6 08:16:28 PM +07 2026
System time: Tue Oct 6 08:16:33 PM +07 2026
System time: Tue Oct 6 08:16:38 PM +07 2026
```

Các dòng log cách nhau 5 giây, chứng minh script đang thực hiện đúng
vòng lặp:

``` bash
while true
```

và:

``` bash
sleep 5
```

------------------------------------------------------------------------

## 8. Theo dõi log liên tục

Sử dụng:

``` bash
tail -f /tmp/monitor.log
```

Kết quả thực tế ghi nhận:

``` text
System time: Tue Oct 6 08:14:08 PM +07 2026
System time: Tue Oct 6 08:14:13 PM +07 2026
System time: Tue Oct 6 08:14:18 PM +07 2026
System time: Tue Oct 6 08:14:23 PM +07 2026
System time: Tue Oct 6 08:14:28 PM +07 2026
System time: Tue Oct 6 08:14:33 PM +07 2026
System time: Tue Oct 6 08:14:38 PM +07 2026
System time: Tue Oct 6 08:14:43 PM +07 2026
System time: Tue Oct 6 08:14:48 PM +07 2026
System time: Tue Oct 6 08:14:53 PM +07 2026
System time: Tue Oct 6 08:14:58 PM +07 2026
System time: Tue Oct 6 08:15:03 PM +07 2026
System time: Tue Oct 6 08:15:08 PM +07 2026
```

Sau khi kiểm tra, nhấn:

``` text
Ctrl + C
```

để thoát `tail -f`. Việc này không dừng `loop-monitor.sh`.

------------------------------------------------------------------------

## 9. Kiểm tra tiến trình sau khi đăng nhập lại

Sau khi ngắt phiên SSH và đăng nhập lại VPS, kiểm tra:

``` bash
pgrep -f loop-monitor.sh
```

Kết quả:

``` text
48413
```

Điều này chứng minh tiến trình vẫn tồn tại sau khi phiên Terminal/SSH
trước đó bị ngắt.

Kiểm tra log sau khi đăng nhập lại:

``` bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

``` text
System time: Tue Oct 6 08:15:53 PM +07 2026
System time: Tue Oct 6 08:15:58 PM +07 2026
System time: Tue Oct 6 08:16:03 PM +07 2026
System time: Tue Oct 6 08:16:08 PM +07 2026
System time: Tue Oct 6 08:16:13 PM +07 2026
System time: Tue Oct 6 08:16:18 PM +07 2026
System time: Tue Oct 6 08:16:23 PM +07 2026
System time: Tue Oct 6 08:16:28 PM +07 2026
System time: Tue Oct 6 08:16:33 PM +07 2026
System time: Tue Oct 6 08:16:38 PM +07 2026
```

Kết quả cho thấy script vẫn ghi log đều đặn với chu kỳ 5 giây.

------------------------------------------------------------------------

## 10. Dừng tiến trình bằng `SIGTERM`

PID của tiến trình:

``` text
48413
```

Gửi tín hiệu `SIGTERM (15)`:

``` bash
kill -15 48413
```

Sau đó kiểm tra:

``` bash
pgrep -f loop-monitor.sh
```

Kết quả:

``` text
```

Không còn PID được trả về, chứng minh tiến trình đã được dừng thành
công.

------------------------------------------------------------------------

## 11. Kiểm tra bằng `ps`

Thực hiện:

``` bash
ps aux | grep loop-monitor.sh
```

Kết quả thực tế:

``` text
devops     48601  0.0  0.1   6680  2304 pts/0    S+   20:17   0:00 grep --color=auto loop-monitor.sh
```

Chỉ còn tiến trình `grep`, không còn:

``` text
/bin/bash ./loop-monitor.sh
```

Do đó `loop-monitor.sh` đã được tắt thành công.

------------------------------------------------------------------------

## 12. Không cần sử dụng `SIGKILL`

Trong bài thực hành này:

``` bash
kill -15 48413
```

đã dừng tiến trình thành công.

Vì vậy không cần sử dụng:

``` bash
kill -9 48413
```

`SIGKILL (9)` chỉ được sử dụng khi tiến trình không phản hồi hoặc không
thể dừng bằng `SIGTERM`.

------------------------------------------------------------------------

## 13. Kết quả đạt được

  Nội dung                                    Kết quả
  ------------------------------------------- ---------
  Tạo `loop-monitor.sh`                       Đạt
  Cấp quyền thực thi                          Đạt
  Chạy bằng `nohup`                           Đạt
  Chạy background bằng `&`                    Đạt
  PID                                         `48413`
  Ghi `/tmp/monitor.log`                      Đạt
  Ghi log mỗi 5 giây                          Đạt
  Tiến trình vẫn chạy sau khi đăng nhập lại   Đạt
  Dừng bằng `kill -15`                        Đạt
  Kiểm tra process sau khi kill               Đạt
  Cần `kill -9`                               Không


