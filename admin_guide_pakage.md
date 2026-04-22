# Gói dịch vụ (Service): Entitlement mode, Purchase policy, Stackable, Credit pool group

Tài liệu này dành cho **Admin vận hành** để cấu hình gói dịch vụ trên giao diện quản trị.  
Mục tiêu là giúp Admin hiểu:

- Mỗi trường có ý nghĩa gì trong nghiệp vụ.
- Hệ thống sẽ vận hành như thế nào sau khi cấu hình.
- Nên chọn cấu hình nào cho từng tình huống thực tế.

Tài liệu dùng ngôn ngữ vận hành, **không cần đọc code**.

---

## 1) Cấu hình ở đâu trên Admin UI

Khi mở màn hình **Service package**, Admin sẽ thấy nhóm trường chính:

- **Thông tin cơ bản**
  - `name`, `type`, `target_type`, `price`, `duration`, `billing_unit`, `is_active`
- **Chính sách quyền lợi**
  - `entitlement_mode`
  - `purchase_policy`
  - `is_stackable`
  - `credit_pool_group`
- **Service package tokens** (danh sách token của gói)
  - `type` (post/image/video/posting_service)
  - `quantity`
  - `reset_policy`
  - `expires_with_entitlement`
  - `is_active`

---

## 2) `entitlement_mode` (Cách cấp quota cho user)

### Vai trò

Quyết định cách hệ thống cấp số lượt sử dụng (quota) cho user sau khi gói được thanh toán.

### Các lựa chọn trên UI

| Giá trị | Ý nghĩa cho Admin | Hành vi vận hành |
| --- | --- | --- |
| `credit_pool` | Dùng kho lượt cộng dồn trong toàn thời hạn gói. | Quota cấp theo token của gói và số lượng mua. Không nhân thêm theo thời lượng trong đơn. |
| `aggregate` | Mua nhiều kỳ thì quota tăng theo số kỳ. | Quota được nhân thêm theo thời lượng mua (ví dụ mua 3 kỳ thì bộ quota được nhân 3). |
| `monthly_bucket` | Quota theo kỳ 30 ngày. | Quota được nạp lại theo từng kỳ 30 ngày tính từ ngày bắt đầu entitlement. |

### Gợi ý chọn nhanh

- Gói đăng tin phổ thông / VIP: chọn **`credit_pool`**.
- Gói bán theo số kỳ mà muốn quota tăng theo số kỳ: chọn **`aggregate`**.
- Gói muốn reset đều theo chu kỳ vận hành: chọn **`monthly_bucket`**.

### Lưu ý quan trọng cho Admin

- `monthly_bucket` dùng chu kỳ **30 ngày cố định** theo ngày bắt đầu entitlement, không phải tháng dương lịch.
- Nếu nghiệp vụ cần reset theo lịch tháng (đầu/tháng) hoặc prorata, cần quy định riêng ở mức sản phẩm.

---

## 3) `purchase_policy` (Chính sách mua thêm)

### Vai trò

Xác định user có được mua thêm gói khi đang còn entitlement hay không.

### Các lựa chọn trên UI

| Giá trị | Ý nghĩa cho Admin |
| --- | --- |
| `allow_parallel` | Cho phép mua thêm khi còn gói cũ, tùy ràng buộc stackable. |
| `allow_when_exhausted` | Chỉ cho mua khi quyền lợi hiện tại đã dùng hết theo chính sách. |
| `allow_cancel_then_buy` | Phải hủy gói đang hiệu lực trước rồi mới được mua mới. |

### Gợi ý vận hành

- Dùng **`allow_parallel`** cho nhóm gói cần bán linh hoạt.
- Dùng **`allow_when_exhausted`** để tránh user mua dồn khi quota cũ còn nhiều.
- Dùng **`allow_cancel_then_buy`** cho gói độc quyền, tránh chồng entitlement.

---

## 4) `is_stackable` (Có cho phép chồng gói hay không)

### Vai trò

Quy định một user có thể giữ nhiều entitlement cùng loại gói trong cùng thời gian hay không.

| Giá trị | Ý nghĩa cho Admin |
| --- | --- |
| `True` | Cho phép chồng gói (mua nhiều lần, nhiều entitlement song song hoặc nối kỳ theo cấu hình). |
| `False` | Không cho chồng gói; user phải hết/hủy entitlement hiện tại trước khi mua mới. |

### Lưu ý vận hành

- Trường này nên được xem cùng với `purchase_policy`.
- Nếu muốn kiểm soát chặt mua mới khi còn gói cũ, cấu hình `is_stackable=False` là lựa chọn an toàn.

---

## 5) Token của gói (Service package tokens)

Token là quyền lợi cụ thể user được dùng: số tin, số ảnh, số video, dịch vụ trong tin.

## 5.1 `reset_policy`

| Giá trị | Ý nghĩa cho Admin | Hành vi thực tế |
| --- | --- | --- |
| `carry_over` | Quota cộng dồn trong kỳ entitlement. | Không tự reset theo kỳ chỉ vì trường này. |
| `reset_monthly` | Quota token reset theo chu kỳ 30 ngày. | Quota token được nạp lại theo kỳ 30 ngày từ start_date entitlement. |

### Ghi nhớ

- Nếu Service đang dùng `monthly_bucket`, hệ thống vận hành theo mô hình reset kỳ cho quyền lợi dạng bucket.
- Khi cần hành vi đơn giản cộng dồn, ưu tiên `carry_over`.

## 5.2 `expires_with_entitlement`

| Giá trị | Ý nghĩa cho Admin | Hành vi thực tế |
| --- | --- | --- |
| `True` (mặc định) | Quota hết hiệu lực cùng entitlement. | Khi entitlement hết hạn, quota liên quan không còn dùng được. |
| `False` | Quota có thể rollover sau khi entitlement hết hạn. | Quota còn dư có thể tiếp tục dùng theo điều kiện nghiệp vụ, trừ các trạng thái bị hủy. |

### Gợi ý

- Dùng `True` cho gói muốn chốt doanh thu theo kỳ rõ ràng.
- Dùng `False` khi muốn trải nghiệm “mua trước dùng dần”.

---

## 6) `credit_pool_group` (Nhóm kho lượt được phép trừ)

### Vai trò

Khi user thực hiện hành động cần trừ lượt (ví dụ đăng tin), hệ thống sẽ chọn quota theo **nhóm pool** và theo loại target phù hợp.

| Giá trị | Dùng cho tình huống nào | Cách hiểu vận hành |
| --- | --- | --- |
| `COMMUNITY` | Tin cộng đồng/mua bán phổ thông. | Nhiều gói cùng nhóm này có thể dùng chung kho lượt cộng đồng. |
| `SPECIAL` | Tin ngành dọc (clinic, lab, dental, ...). | Gói cùng nhóm và cùng target_type ngành sẽ dùng chung pool theo ngành đó. |
| `HEALTHCARE_STAFF` | Luồng nhân sự y tế (bác sĩ, điều dưỡng...). | Pool dành riêng nhóm nhân sự y tế, không lẫn với community. |
| `GENERAL` | Trường hợp mặc định còn lại. | Áp dụng theo target_type cấu hình của gói. |

### Quy tắc cấu hình để tránh sai pool

1. Muốn nhiều gói dùng chung kho lượt: đặt cùng `credit_pool_group` và target_type phù hợp.
2. Muốn tách theo ngành: dùng `SPECIAL` và đảm bảo target_type theo đúng ngành.
3. Không trộn `COMMUNITY` với `SPECIAL` nếu nghiệp vụ yêu cầu tách quota.

---

## 7) Luồng vận hành entitlement (phiên bản cho Admin)

### 7.1 Sau khi thanh toán thành công

1. Hệ thống tạo entitlement cho user theo gói đã mua.
2. Hệ thống cấp quota token theo cấu hình `entitlement_mode` + token setup.
3. Quota được gắn hiệu lực theo `expires_with_entitlement`.

### 7.2 Khi user dùng tính năng cần quota (đăng tin, dùng token)

1. Hệ thống xác định nhóm pool theo `credit_pool_group` và ngữ cảnh target.
2. Hệ thống kiểm tra quota còn lại.
3. Nếu đủ quota: giữ chỗ/ghi nhận sử dụng và trừ lượt theo chính sách.
4. Nếu không đủ quota: trả về yêu cầu mua thêm/nâng cấp theo chính sách mua.

### 7.3 Cơ chế theo kỳ

- Với gói có reset kỳ (bucket/tháng), quota được đồng bộ theo chu kỳ 30 ngày.
- Các phiên giữ chỗ/quota tạm thời được dọn dẹp tự động theo lịch tác vụ nền.

---

## 8) Playbook cấu hình nhanh cho Admin

## 8.1 Gói cộng đồng phổ biến (dễ vận hành)

- `entitlement_mode`: `credit_pool`
- `purchase_policy`: `allow_parallel`
- `is_stackable`: `True`
- `credit_pool_group`: `COMMUNITY`
- Token:
  - `reset_policy`: `carry_over`
  - `expires_with_entitlement`: `True`

## 8.2 Gói ngành dọc (clinic/lab/dental) tách pool riêng

- `credit_pool_group`: `SPECIAL`
- `target_type`: chọn đúng ngành (ví dụ `CLINIC`)
- Nếu muốn nhiều gói cùng ngành dùng chung lượt: giữ cùng group + cùng target_type.

## 8.3 Gói cần kiểm soát chặt mua mới

- `is_stackable`: `False`
- `purchase_policy`: `allow_cancel_then_buy` hoặc `allow_when_exhausted`

## 8.4 Gói theo chu kỳ tháng vận hành

- `entitlement_mode`: `monthly_bucket`
- Token chính: `reset_policy=reset_monthly`
- Truyền thông rõ với user: reset theo block 30 ngày, không theo lịch tháng dương lịch.

---

## 9) Checklist QA trước khi bật `is_active`

1. Đúng `target_type` theo đối tượng gói.
2. Đúng `credit_pool_group` theo nghiệp vụ dùng chung/tách riêng.
3. `entitlement_mode` phù hợp chính sách bán hàng.
4. Token `quantity`, `reset_policy`, `expires_with_entitlement` đúng kỳ vọng.
5. Test 3 tình huống: mua mới, dùng quota, hết quota/mua lại.
6. Test thêm trường hợp gia hạn hoặc mua song song theo `purchase_policy`.

---

## 10) Tóm tắt điều hành

- `entitlement_mode` quyết định **cách cấp và làm mới quota**.
- `purchase_policy` + `is_stackable` quyết định **luật mua thêm**.
- `credit_pool_group` + `target_type` quyết định **quota nào được phép trừ**.
- Token (`reset_policy`, `expires_with_entitlement`) quyết định **quota reset và hết hạn ra sao**.

Nếu cấu hình đúng 4 cụm trên, hành vi mua gói và sử dụng quota sẽ ổn định, minh bạch cho cả user và đội vận hành.
