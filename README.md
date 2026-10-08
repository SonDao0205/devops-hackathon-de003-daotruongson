# DevOps Hackathon – Đề 003: Quản lý công việc (Task)

## 1. Thông tin sinh viên

| Họ và tên | Mã sinh viên | Lớp | Tài khoản Linux | GitHub | Cổng Nginx |
|---|---|---|---|---|---|
| Đào Trường Sơn | SV111 | HN_KS24_CNTT2 | `sondao` | [SonDao0205](https://github.com/SonDao0205) | `8081` |

## 2. Môi trường triển khai

- Hệ điều hành : Ubuntu 26.04.1 LTS
- Phiên bản nginx : nginx/1.28.3
- Git : git version 2.53.0
- Nơi chạy : Máy ảo Lima trên Macos

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


- chỉnh src/nginx/sondao.conf để dùng cổng 8081, IP guest, và document root của dự án.

sudo cp /var/www/devops-hackathon-de003-daotruongson/src/nginx/sondao.conf /etc/nginx/sites-available/sondao.conf
sudo ln -sfn /etc/nginx/sites-available/sondao.conf /etc/nginx/sites-enabled/sondao.conf
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx

- kết quả kiểm tra cấu hình:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


## 7. Kiểm tra và minh chứng

- kiểm tra nginx hoạt động

systemctl is-active nginx
curl -I http://127.0.0.1:8081

- kết quả : Nginx trả active và curl trả HTTP/1.1 200 OK


Chèn ảnh vào README sau khi lưu, ví dụ: `![Trạng thái UFW](screenshots/03-firewall.png)`. Ảnh terminal cần thấy prompt user và kết quả đầy đủ.

## 8. Quy trình cập nhật website

- máy cá nhân, sửa index.html rồi commit và push:

git add src/index.html
git commit -m "docs: update website content"
git push

- trên máy ảo:

cd /var/www/devops-hackathon-de003-daotruongson
git pull

Kiểm tra lại website:

curl http://127.0.0.1:8081

## 9. Sự cố và cách khắc phục

- clone repo báo Permission denied

clone khi chưa gán quyền khiến báo chưa có quyền , cách khắc phục :
sudo chown -R sondao:sondao /var/www/devops-hackathon-de003-daotruongson