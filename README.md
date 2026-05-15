# Báo cáo số liệu Bộ Công Thương (MOIT)

## Tổng quan

App `src.gov_reporting` cung cấp 2 API báo cáo số liệu cho Bộ Công Thương (MOIT):

1. **Statistics API**: Báo cáo chi tiết các chỉ tiêu theo khoảng thời gian tùy chỉnh
2. **Summary API**: Báo cáo tóm tắt so sánh số liệu cho dashboard

Cả hai API đều hỗ trợ xác thực bằng JWT hoặc UserName/PassWord theo mẫu BCT.

## Endpoints

| API | URL | Method | Mô tả |
|-----|-----|--------|-------|
| Statistics | `POST /api/v1/gov-reporting/moit/statistics` | POST | Báo cáo chi tiết số liệu giao dịch |
| Summary | `POST /api/v1/gov-reporting/moit/summary` | POST | Báo cáo tóm tắt so sánh |

## Xác thực (Authentication)

### Cách 1: JWT Token (cho trang báo cáo nội bộ)

1. Lấy JWT token từ user `type=REPORT`:
   ```http
   POST /api/v1/token/
   Content-Type: application/json

   {
     "phone_number": "+84901234567",
     "password": "password_here"
   }
   ```

2. Sử dụng token trong header:
   ```http
   Authorization: Bearer <access_token>
   ```

### Cách 2: UserName/PassWord trong body (theo mẫu BCT)

```json
{
  "UserName": "BaocaoTMDT",
  "PassWord": "<mật_khẩu_user_REPORT>"
}
```

**Lưu ý:**
- `UserName` phải khớp với `MOIT_REPORT_BODY_USERNAME` (env, mặc định `BaocaoTMDT`)
- `PassWord` phải khớp mật khẩu của user `type=REPORT` đang active
- User không phải `REPORT` → 403 Forbidden
- Sai credentials → 401 Unauthorized

## API Details

### 1. Statistics API

**URL:** `POST /api/v1/gov-reporting/moit/statistics`

**Mô tả:** Báo cáo chi tiết các chỉ tiêu giao dịch theo khoảng thời gian tùy chỉnh.

**Caching:** Không cache - luôn query database mỗi request.

**Request Body:**
```json
{
  "UserName": "BaocaoTMDT",           // Optional nếu dùng JWT
  "PassWord": "password",              // Optional nếu dùng JWT
  "start_date": "2026-01-01",          // Optional: YYYY-MM-DD
  "end_date": "2026-05-15"             // Optional: YYYY-MM-DD
}
```

**Rules:**
- `start_date` và `end_date` phải có cả hai hoặc không có
- `end_date >= start_date`
- Nếu không gửi → mặc định: 01/01/năm hiện tại → hôm nay

**Response:**
```json
{
  "soLuongTruyCap": 0,
  "soNguoiBan": 150,
  "soNguoiBanMoi": 0,
  "tongSoSanPham": 0,
  "soSanPhamMoi": 0,
  "soLuongGiaoDich": 1250,
  "tongSoDonHangThanhCong": 1200,
  "tongSoDonHangKhongThanhCong": 50,
  "tongGiaTriGiaoDich": 250000000
}
```

### 2. Summary API

**URL:** `POST /api/v1/gov-reporting/moit/summary`

**Mô tả:** Báo cáo tóm tắt so sánh số liệu cho dashboard MOIT.

**Caching:** Có cache Django với TTL đến cuối ngày hiện tại.

**Khoảng thời gian cố định:**
- Current: 2020-01-01 → hôm qua
- Previous: 2020-01-01 → (hôm qua - 7 ngày)

**Request Body:**
```json
{
  "UserName": "BaocaoTMDT",           // Optional nếu dùng JWT
  "PassWord": "password"               // Optional nếu dùng JWT
}
```

**Response:**
```json
{
  "userCount": {
    "gia_tri": 1250,
    "don_vi": "",
    "ti_le_tang_truong": 5.26
  },
  "postingMembers": {
    "gia_tri": 150,
    "don_vi": "",
    "ti_le_tang_truong": 2.04
  },
  "transactionValue": {
    "gia_tri": 250000000,
    "don_vi": "VND",
    "ti_le_tang_truong": 8.75
  },
  "transactionCount": {
    "gia_tri": 1250,
    "don_vi": "",
    "ti_le_tang_truong": 6.38
  }
}
```

**Giải thích response:**
- `gia_tri`: Giá trị số liệu hiện tại
- `don_vi`: Đơn vị (VND cho tiền, rỗng cho count)
- `ti_le_tang_truong`: Tỷ lệ tăng trưởng % so với tuần trước

## Cấu hình

- **`MOIT_REPORT_BODY_USERNAME`**: Giá trị bắt buộc cho `UserName` (mặc định: `BaocaoTMDT`)
- **User REPORT**: Tạo ít nhất 1 user với `type=REPORT` trong admin/DB
- **INSTALLED_APPS**: Thêm `src.gov_reporting`
- **Whitelist**: Endpoint được whitelist trong `InternalAPIKeyMiddleware` (không cần `X-API-KEY`)

## Examples

### Statistics với JWT
```bash
curl -X POST http://localhost:8000/api/v1/gov-reporting/moit/statistics \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..." \
  -d '{
    "start_date": "2026-01-01",
    "end_date": "2026-05-15"
  }'
```

### Statistics với UserName/PassWord
```bash
curl -X POST http://localhost:8000/api/v1/gov-reporting/moit/statistics \
  -H "Content-Type: application/json" \
  -d '{
    "UserName": "BaocaoTMDT",
    "PassWord": "secure_password",
    "start_date": "2026-01-01",
    "end_date": "2026-05-15"
  }'
```

### Summary API
```bash
curl -X POST http://localhost:8000/api/v1/gov-reporting/moit/summary \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..." \
  -d '{}'
```

## Implementation Notes

### Caching Strategy
- **Statistics API**: `compute_details()` - luôn query fresh, không cache
- **Summary API**: `compute_all()` - cache Django với TTL đến 23:59:59 hôm nay
- Cache key: `moit_metrics_{start_date}_{end_date}`

### Metrics Calculation
- Sử dụng `MetricsBatch` class để batch compute và tránh N+1 queries
- Query tối ưu: aggregate trong 1-2 queries thay vì multiple queries
- Date range: timezone-aware datetimes

### Error Handling
- 400: Validation errors (invalid dates, missing fields)
- 401: Authentication failed
- 403: User không phải REPORT type
- 500: Server errors

### Performance
- Statistics: ~2-3 DB queries per request
- Summary: 1 DB query (cached) + 1 cache lookup per request
- Cache giúp giảm load DB cho Summary API trong ngày
