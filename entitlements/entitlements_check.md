
# 1. Mục đích API

**POST /api/v1/service/entitlements/check**

Dùng để:

* Kiểm tra **khả năng sử dụng gói (entitlements)** khi user tạo tin đăng
* Tính toán:

  * Token còn lại (post, image, video)
  * Phần **thiếu (over-usage)**
  * Các **chi phí phát sinh (extra pricing)**

---

# 2. Khái niệm chính

### 2.1 Entitlements (quyền sử dụng gói)

* Mỗi user có thể mua **nhiều gói**
* Mỗi gói chứa:

  * `post_token`
  * `image_token`
  * `video_token`
  * `expired_at`

👉 Một loại gói có thể được mua **nhiều lần (multiple subscriptions)**

---

### 2.2 Nguyên tắc sử dụng gói

1. Lấy danh sách gói theo **type phù hợp**
2. Sort theo:

   ```
   expired_at ASC (gần hết hạn dùng trước)
   ```
3. Trừ token theo thứ tự:

   ```
   gói gần hết hạn → gói xa hơn
   ```
4. Nếu thiếu:

   * Sinh **extra usage**
   * Map sang **pricing rule tương ứng**

---

# 3. Flow tổng quát (FE ↔ BE)

### Bước 1: FE gửi request

Sau khi user:

* Nhập thông tin tin đăng
* Upload media (image/video)

FE gọi:

```
POST /api/v1/service/entitlements/check
```

Payload ví dụ:

```json
{
  "type": "CHO_DUOC_PHAM",
  "post": 1,
  "image": 4,
  "video": 2
}
```

---

### Bước 2: BE xử lý

#### 2.1 Xác định nhóm gói (Service Group)

### Case A — Mua bán & cộng đồng

Áp dụng cho các type:

```
HOP_TAC
CHIA_SE_NHA
KET_BAN
VIEC_LAM
CHO_DUOC_PHAM
CHO_Y_TE
```

→ Service group:

```
MUABAN_CONGDONG
```

Các gói hợp lệ:

* Standard
* Professional
* VIP

---

### 2.2 Thuật toán trừ token

Ví dụ:

User đăng:

```
CHO_DUOC_PHAM
post: 1
image: 4
video: 2
```

Danh sách gói:

| Gói          | post | image | video | expire |
| ------------ | ---- | ----- | ----- | ------ |
| Standard     | 0    | 0     | 0     | 1/1    |
| Professional | 0    | 2     | 1     | 3/1    |
| VIP          | 0    | 0     | 1     | 4/1    |

---

### Bước trừ:

#### Gói 1: Standard

* Không có token → skip

#### Gói 2: Professional

* image: dùng 2 → còn thiếu 2
* video: dùng 1 → còn thiếu 1
* post: thiếu 1

#### Gói 3: VIP

* video: dùng 1 → hết thiếu video

---

### Kết quả còn thiếu:

```
post: 1
image: 2
video: 0
```

---

### 2.3 Mapping sang pricing rule

BE map phần thiếu sang các pricing:

| Loại thiếu | Pricing                           |
| ---------- | --------------------------------- |
| post       | POSTING_FEE_30_DAYS_CHO_DUOC_PHAM |
| image      | EXTRA_IMAGE                       |
| video      | EXTRA_VIDEO                       |

---

### Bước 3: BE response

```json
{
  "remaining_required": {
    "post": 1,
    "image": 2,
    "video": 0
  },
  "pricing": [
    {
      "code": "POSTING_FEE_30_DAYS_CHO_DUOC_PHAM",
      "quantity": 1
    },
    {
      "code": "EXTRA_IMAGE",
      "quantity": 2
    }
  ]
}
```

---

### Bước 4: FE xử lý

* Hiển thị chi phí phát sinh
* Chuyển sang **checkout**
* Tổng tiền = sum(pricing rule)

---

# 4. Flow cho gói đặc thù (Special Packages)

## 4.1 Các loại tin đặc thù

Bao gồm:

* NVYT (Nhân viên y tế)
* Phòng khám
* Xét nghiệm
* Chẩn đoán
* Nha khoa
* YHCT
* Nhà thuốc
* Xe cứu thương

---

## 4.2 Logic xử lý

### Ví dụ: NVYT

FE gửi:

```json
{
  "type": "NVYT",
  "target": "BAC_SY",
  "post": 1,
  "image": 10,
  "video": 3
}
```

---

### BE xử lý

#### Bước 1: Lấy gói tương ứng

```
GOI_TRON_GOI_NVYT
```

---

#### Bước 2: Trừ token

Gói hiện có:

```
post: 0
image: 6
video: 2
```

---

#### Kết quả:

```
image: thiếu 4
video: thiếu 1
post: thiếu 1
```

---

### Bước 3: Mapping pricing

| Loại  | Pricing                   |
| ----- | ------------------------- |
| post  | POSTING_FEE_30_DAYS_BACSY |
| image | EXTRA_IMAGE               |
| video | EXTRA_VIDEO               |

---

### Bước 4: Response

```json
{
  "remaining_required": {
    "post": 1,
    "image": 4,
    "video": 1
  },
  "pricing": [
    {
      "code": "POSTING_FEE_30_DAYS_BACSY",
      "quantity": 1
    },
    {
      "code": "EXTRA_IMAGE",
      "quantity": 4
    },
    {
      "code": "EXTRA_VIDEO",
      "quantity": 1
    }
  ]
}
```

---

# 5. Quy tắc quan trọng

### 5.1 Ưu tiên sử dụng gói

* Gói gần hết hạn → dùng trước

---

### 5.2 Không trộn nhóm gói

* Mua bán & cộng đồng → chỉ dùng gói MUABAN_CONGDONG
* NVYT → chỉ dùng GOI_TRON_GOI_NVYT
* Không cross-group

---

### 5.3 Fallback sang pricing

* Khi **không đủ token**
* Luôn trả về:

  * số lượng thiếu
  * pricing tương ứng

---

### 5.4 Pricing rule là source of truth

* Không hardcode giá
* BE chỉ trả:

  ```
  code + quantity
  ```
* FE hoặc checkout service tính tiền

---

# 6. Tóm tắt flow

```
FE nhập tin → gọi check entitlement
        ↓
BE:
  - xác định group
  - lấy gói user
  - sort theo expire
  - trừ token
  - tính phần thiếu
  - map pricing
        ↓
FE:
  - hiển thị phí phát sinh
  - chuyển checkout
```

---
