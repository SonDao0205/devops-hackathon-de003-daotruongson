# DevOps Hackathon – Đề 003: Quản lý công việc (Task)

## 1. Thông tin sinh viên

| Họ và tên | Mã sinh viên | Lớp | Tài khoản Linux | GitHub | Cổng Nginx |
|---|---|---|---|---|---|
| Đào Trường Sơn | SV111 | HN_KS24_CNTT2 | `sondao` | [SonDao0205](https://github.com/SonDao0205) | `8081` |

## 2. Môi trường triển khai

Hệ điều hành : Ubuntu 26.04.1 LTS
Phiên bản nginx : nginx/1.28.3
Git : git version 2.53.0
Nơi chạy : Máy ảo Lima trên Macos

## 3. Cấu trúc dự án

devops-hackathon-de003-daotruongson/
├── src/
│   ├── index.html
│   └── nginx/
│       └── sondao.conf
├── screenshot/
├── .gitignore
└── README.md

## 4. Cấu hình Nginx

| Tham số trong template | Giá trị đã điền | Giải thích |
|---|---|---|
| <PORT> | 8081 | Cổng IPv4 riêng, không dùng cổng 80 |
| [::]:<PORT> | [::]:8081 | Lắng nghe IPv6 cùng cổng |
| <SERVER_NAME> | 192.168.5.15 | IP guest hiện tại; cần cập nhật nếu IP guest thay đổi |
| <WEB_ROOT> | /var/www/devops-hackathon-de003-daotruongson/src | Thư mục web root |
| <INDEX_FILE> | index.html | Trang mặc định |
| access log | /var/log/nginx/sondao.access.log | Log truy cập riêng |
| error log | /var/log/nginx/sondao.error.log | Log lỗi riêng |
| <ALLOW_DIRECTIVE> | allow all; | Cho phép request tới website |
| try_files | $uri $uri/ =404 | Trả 404 khi không tìm thấy tài nguyên |

## 5. Tường lửa UFW

Đã cho phép SSH trước khi bật firewall, sau đó mở cổng website:

sudo ufw allow 22/tcp
sudo ufw allow 8081/tcp
sudo ufw enable
sudo ufw status verbose

Kết quả

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
8081/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
8081/tcp (v6)              ALLOW IN    Anywhere (v6)

## 6. Các bước triển khai

Các lệnh đã thực hiện trên Ubuntu:

- tạo thư mục và gán quyền sở hữu cho user : sondao

sudo chown -R sondao:sondao /var/www/devops-hackathon-de003-daotruongson
find /var/www/devops-hackathon-de003-daotruongson -type d -exec chmod 755 {} +
find /var/www/devops-hackathon-de003-daotruongson -type f -exec chmod 644 {} +


cd /var/www/devops-hackathon-de003-daotruongson
mkdir -p src/nginx
cp nginx/sondao.conf src/nginx/sondao.conf


Đã chỉnh `src/nginx/sondao.conf` để dùng cổng `8081`, IP guest, và document root của dự án.

```bash
sudo cp /var/www/devops-hackathon-de003-daotruongson/src/nginx/sondao.conf /etc/nginx/sites-available/sondao.conf
sudo ln -sfn /etc/nginx/sites-available/sondao.conf /etc/nginx/sites-enabled/sondao.conf
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
```

Kết quả kiểm tra cấu hình:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## 7. Kiểm tra và minh chứng

Kiểm tra trong Ubuntu:

```bash
systemctl is-active nginx
curl -I http://127.0.0.1:8081
```

Kết quả thực tế: Nginx trả `active`; curl trả `HTTP/1.1 200 OK`.

Thêm ảnh chụp vào `screenshots/` và giữ tên theo thứ tự yêu cầu:

| File ảnh | Nội dung |
|---|---|
| `01-user-id.png` | Kết quả `id` của user `sondao` |
| `02-services.png` | Nginx active và enabled |
| `03-firewall.png` | UFW active và rule cổng `8081` IPv4/IPv6 |
| `04-website.png` | Website mở trên trình duyệt, thấy URL IP:cổng |
| `05-github.png` | Repository public trên GitHub |
| `06-commits.png` | Lịch sử có ít nhất 4 commit |
| `07-update.png` | Website sau lần cập nhật nội dung |

Chèn ảnh vào README sau khi lưu, ví dụ: `![Trạng thái UFW](screenshots/03-firewall.png)`. Ảnh terminal cần thấy prompt user và kết quả đầy đủ.

## 8. Quy trình cập nhật website

Trên máy phát triển, sửa `src/index.html`, rồi commit và push:

```bash
git add src/index.html
git commit -m "docs: update website content"
git push
```

Trên máy ảo:

```bash
cd /var/www/devops-hackathon-de003-daotruongson
git pull
```

Kiểm tra lại website:

```bash
curl http://127.0.0.1:8081
```

Với nội dung HTML tĩnh, không cần reload Nginx sau khi cập nhật file. Chụp ảnh trang đã cập nhật thành `07-update.png`.

## 9. Sự cố và cách khắc phục

### Clone repo báo Permission denied

Không chạy `git clone` khi đang ở bên trong thư mục repo. Clone từ `/var/www`. Nếu clone bằng `sudo`, đổi quyền sở hữu bằng lệnh sau (không gõ dấu ngoặc nhọn quanh username):

```bash
sudo chown -R sondao:sondao /var/www/devops-hackathon-de003-daotruongson
```

### Nginx không truy cập được

`nginx -t` thành công, Nginx `active`, UFW mở cổng `8081`, và curl trong guest trả `200 OK`. Tuy nhiên `192.168.5.15` là IP user-mode mặc định của Lima, không truy cập trực tiếp được từ host theo thiết kế. Dùng `http://localhost:8081` trên Mac nếu Lima tự chuyển tiếp cổng. Nếu yêu cầu phải dùng IP guest trực tiếp, chuyển VM sang cấu hình mạng VMNet phù hợp, kiểm tra IP mới, rồi cập nhật `server_name`, README và ảnh minh chứng.

## Nộp bài

Trước khi nộp, điền các thông tin cá nhân còn thiếu, xác nhận repository public, kiểm tra có ít nhất 4 commit và đẩy README cùng server block/ảnh minh chứng lên GitHub. Nộp repository:

<https://github.com/SonDao0205/devops-hackathon-de003-daotruongson>
