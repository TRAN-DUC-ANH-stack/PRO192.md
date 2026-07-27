# Package `com.mycompany.restaurantmanagement.repository`

Tài liệu mô tả các class thuộc package **repository** trong dự án Restaurant Management, bao gồm: `InvoiceRepository`, `PaymentRepository`.

Cả hai class đều kế thừa `BaseRepository<T, ID>` (cung cấp cơ chế đọc/ghi dữ liệu chung từ file: `data`, `saveToFile()`, `save()`, ...) và chịu trách nhiệm chuyển đổi qua lại giữa **object** (`Invoice`, `Payment`) và **dòng dữ liệu dạng text** được phân tách bởi dấu `|` để lưu trữ vào file.

---

## 1. InvoiceRepository

Kế thừa: `BaseRepository<Invoice, String>`

Quản lý việc đọc/ghi dữ liệu hóa đơn (`Invoice`) từ/vào file được cấu hình tại `AppConfig.INVOICE_FILE_PATH`.

### Constructor

- `InvoiceRepository()` — gọi `super(AppConfig.INVOICE_FILE_PATH)` để khởi tạo `BaseRepository` với đường dẫn file lưu trữ hóa đơn.

### Phương thức

| Phương thức | Mô tả |
|---|---|
| `findById(String id)` (override) | Duyệt qua danh sách `data`, trả về `Invoice` có `invoiceId` khớp với `id`; nếu không tìm thấy trả về `null`. |
| `findByOrderId(String orderId)` | Duyệt qua danh sách `data`, trả về `Invoice` mà `order` khác `null` và `order.getOrderId()` khớp với `orderId`; nếu không tìm thấy trả về `null`. |
| `parseLine(String line)` (override, protected) | Chuyển một dòng text (phân tách bởi `\|`) thành đối tượng `Invoice`: <br>• `d[0]` → `invoiceId` <br>• `d[1]` → tạo `Order` mới (chỉ có `orderId`, danh sách món ăn = `null`) <br>• `d[2]` → `totalAmount` (parse `double`) <br>• `d[3]` → `issueDate` (parse theo định dạng `yyyy-MM-dd`; nếu lỗi thì dùng ngày hiện tại) <br>• `d[4]` → `status` (parse `InvoiceStatus`) |
| `toLine(Invoice invoice)` (override, protected) | Chuyển đối tượng `Invoice` thành dòng text để lưu file, định dạng: `invoiceId\|orderId\|totalAmount\|issueDate(yyyy-MM-dd)\|status` (nếu `order == null` thì ghi `"null"` thay cho `orderId`). |

### Định dạng dữ liệu lưu file

```
invoiceId | orderId | totalAmount | issueDate(yyyy-MM-dd) | status
```

Ví dụ: `INV1700000000000|ORD001|150000.0|2026-07-26|UNPAID`

---

## 2. PaymentRepository

Kế thừa: `BaseRepository<Payment, String>`

Quản lý việc đọc/ghi dữ liệu giao dịch thanh toán (`Payment`) từ/vào file được cấu hình tại `AppConfig.PAYMENT_FILE_PATH`.

### Constructor

- `PaymentRepository()` — gọi `super(AppConfig.PAYMENT_FILE_PATH)` để khởi tạo `BaseRepository` với đường dẫn file lưu trữ giao dịch thanh toán.

### Phương thức

| Phương thức | Mô tả |
|---|---|
| `findById(String id)` (override) | Trả về `null` nếu `id == null`; ngược lại duyệt `data` để tìm `Payment` có `paymentId` khớp với `id`, trả về `null` nếu không tìm thấy. |
| `save(Payment entity)` (override) | Ghi đè hành vi lưu mặc định: tìm giao dịch cũ theo `paymentId` (`findById`), nếu tồn tại thì xóa khỏi `data`, sau đó thêm `entity` mới vào `data` và gọi `saveToFile()` để ghi xuống file (đảm bảo không bị trùng lặp bản ghi). |
| `findByInvoiceId(String invoiceId)` | Trả về `null` nếu `invoiceId == null`; ngược lại duyệt `data` để tìm `Payment` có `invoiceId` khớp, trả về `null` nếu không tìm thấy. |
| `parseLine(String line)` (override, protected) | Chuyển một dòng text (phân tách bởi `\|`) thành đối tượng `Payment`. Nếu số phần tử sau khi tách < 6 thì trả về `null`. Thứ tự các trường: <br>• `d[0]` → `paymentId` <br>• `d[1]` → `invoiceId` <br>• `d[2]` → `amount` (parse `double`) <br>• `d[3]` → `paymentDate` (parse từ timestamp `long`) <br>• `d[4]` → `status` (gán sau khi khởi tạo, parse `PaymentStatus`) <br>• `d[5]` → `method` (parse `PaymentMethod`, truyền vào constructor) <br>Nếu có lỗi trong quá trình parse (ví dụ sai định dạng số), bắt `Exception` và trả về `null`. |
| `toLine(Payment payment)` (override, protected) | Chuyển đối tượng `Payment` thành dòng text để lưu file, định dạng: `paymentId\|invoiceId\|amount\|paymentDate(timestamp)\|status\|method`. |

### Định dạng dữ liệu lưu file

```
paymentId | invoiceId | amount | paymentDate(timestamp) | status | method
```

Ví dụ: `PAY1700000000123|INV1700000000000|150000.0|1700000000123|SUCCESS|CASH`

> Lưu ý thứ tự trường khi ghi (`toLine`) là `status` trước `method`, khớp với thứ tự đọc lại trong `parseLine` (`d[4]` = status, `d[5]` = method).

---

## Vai trò trong kiến trúc dự án

```
Service Layer (InvoiceService / PaymentService)
                │
                ▼
Repository Layer (InvoiceRepository / PaymentRepository)
                │
                ▼
      BaseRepository<T, ID>  ──►  File lưu trữ (theo AppConfig)
   (data, saveToFile, save...)     (INVOICE_FILE_PATH / PAYMENT_FILE_PATH)
```

- **`InvoiceRepository`** và **`PaymentRepository`** là tầng truy cập dữ liệu (Data Access Layer), được các class ở tầng **service** (`InvoiceService`, `PaymentService`) sử dụng để thao tác với dữ liệu hóa đơn/thanh toán.
- Cả hai đều kế thừa `BaseRepository<T, ID>` để tái sử dụng logic đọc/ghi file chung, chỉ cần override `parseLine` (đọc) và `toLine` (ghi) để quy định định dạng riêng cho từng loại dữ liệu.
- `PaymentRepository` override thêm `save()` để xử lý việc thay thế bản ghi cũ trước khi thêm bản ghi mới, tránh trùng lặp dữ liệu trong file.
