# **Hướng dẫn thiết lập Cronjob trên Linux**

Tài liệu này hướng dẫn cách thiết lập các cronjob và worker cần thiết cho backend chạy môi trường production.

---

## **1. Các job cần chạy**

### **`check_publication_expiry`**

* **Command:**

  ```bash
  python manage.py check_publication_expiry --batch-size 1000
  ```
* **Lịch chạy:** `0 0 * * *` (00:00 mỗi ngày)
* **Mục đích:**
  Quét lại toàn bộ user để đồng bộ (reconcile) trạng thái hết hạn của publication và gửi notification tương ứng.

---

### **`release_expired_entitlement_sessions`**

* **Command:**

  ```bash
  python manage.py release_expired_entitlement_sessions --limit 500 --batch-size 100
  ```
* **Lịch chạy:** `* * * * *` (mỗi phút)
* **Mục đích:**
  Giải phóng (release) các token đã được giữ (reserve) khi entitlement session hết hạn.

---

## **2. Vì sao `release_expired_entitlement_sessions` nên chạy mỗi phút**

* Trong file `src/service/services/entitlement.py` có cấu hình:

  ```python
  SESSION_TTL_MINUTES = 17
  ```
* Nghiệp vụ cho phép tối đa **17 phút** từ lúc reserve token đến khi hoàn tất thanh toán.
* Do đó, job release nên chạy **mỗi phút** để:

  * Token được trả lại sớm nhất ngay khi session hết hạn.
* Nếu chỉ chạy mỗi 5 phút:

  * Token có thể bị giữ thêm tối đa gần **5 phút** sau khi đã hết hạn → gây lãng phí tài nguyên.

---

## **3. Yêu cầu bắt buộc: chạy worker `django-q`**

Job `check_publication_expiry` **không xử lý toàn bộ logic trực tiếp**, mà sẽ enqueue các `AsyncTask`.

👉 Vì vậy, production **bắt buộc phải có process `qcluster` chạy liên tục**.

### **Ví dụ file systemd**

File: `/etc/systemd/system/khambenhvn-qcluster.service`

```ini
[Unit]
Description=Khambenhvn Django Q Cluster
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/srv/khambenhvn/BE
ExecStart=/srv/venv/bin/python manage.py qcluster
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### **Các lệnh kích hoạt**

```bash
sudo systemctl daemon-reload
sudo systemctl enable khambenhvn-qcluster
sudo systemctl start khambenhvn-qcluster
sudo systemctl status khambenhvn-qcluster
```

---

## **4. Thiết lập cron cho user chạy ứng dụng**

Mở crontab:

```bash
crontab -e
```

Thêm các dòng sau:

```cron
# Reconcile publication expiry lúc 00:00 mỗi ngày
0 0 * * * cd /srv/khambenhvn/BE && /usr/bin/flock -n /tmp/check_publication_expiry.lock /srv/venv/bin/python manage.py check_publication_expiry --batch-size 1000 >> /var/log/khambenhvn/check_publication_expiry.log 2>&1

# Release entitlement session hết hạn mỗi phút
* * * * * cd /srv/khambenhvn/BE && /usr/bin/flock -n /tmp/release_expired_entitlement_sessions.lock /srv/venv/bin/python manage.py release_expired_entitlement_sessions --limit 500 --batch-size 100 >> /var/log/khambenhvn/release_expired_entitlement_sessions.log 2>&1
```

### **Giải thích thêm**

* `flock`: tránh việc job bị chạy trùng (concurrent).
* `>> log 2>&1`: ghi cả stdout và stderr vào file log.

---

## **5. Tạo thư mục log (nếu chưa có)**

```bash
sudo mkdir -p /var/log/khambenhvn
sudo chown www-data:www-data /var/log/khambenhvn
```

> Nếu ứng dụng chạy bằng user khác (không phải `www-data`), cần chỉnh lại quyền cho phù hợp.

---

## **6. Kiểm tra sau khi setup**

### **Kiểm tra crontab**

```bash
crontab -l
```

---

### **Chạy thử command thủ công**

```bash
cd /srv/khambenhvn/BE

/srv/venv/bin/python manage.py check_publication_expiry --batch-size 1000
/srv/venv/bin/python manage.py release_expired_entitlement_sessions --limit 500 --batch-size 100
```

---

### **Kiểm tra log**

```bash
tail -f /var/log/khambenhvn/check_publication_expiry.log
tail -f /var/log/khambenhvn/release_expired_entitlement_sessions.log
```

---

### **Kiểm tra worker**

```bash
sudo systemctl status khambenhvn-qcluster
```

---

## **7. Lưu ý về timezone**

* Trong `src/settings.py`:

  ```python
  TIME_ZONE = "UTC"
  USE_TZ = True
  ```
* Cronjob sẽ chạy theo **timezone của server Linux**, không phải timezone Django.

👉 Nếu muốn job `check_publication_expiry` chạy đúng **00:00 giờ Việt Nam**, cần:

* Cấu hình timezone server/crontab là:

  ```
  Asia/Ho_Chi_Minh
  ```

  **hoặc**
* Điều chỉnh lại lịch cron tương ứng với UTC.
