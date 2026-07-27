# Package `com.mycompany.restaurantmanagement.model`

Tài liệu mô tả các class thuộc package **model** trong dự án Restaurant Management, bao gồm: `Invoice`, `InvoiceStatus`, `Payment`, `PaymentMethod`, `PaymentStatus`.

---

## 1. Invoice

Đại diện cho một hóa đơn thanh toán, liên kết với một `Order` và một `Payment`.

### Thuộc tính

| Thuộc tính | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `invoiceId` | `String` | Mã định danh hóa đơn |
| `order` | `Order` | Đơn hàng liên kết với hóa đơn |
| `payment` | `Payment` | Thông tin thanh toán liên kết |
| `totalAmount` | `Double` | Tổng số tiền của hóa đơn |
| `issueDate` | `Date` | Ngày lập hóa đơn |
| `status` | `InvoiceStatus` | Trạng thái hóa đơn (mặc định `UNPAID`) |

### Constructor

- `Invoice()` — constructor mặc định.
- `Invoice(String invoiceId, Order order, Payment payment, Double totalAmount, Date issueDate, InvoiceStatus status)` — khởi tạo đầy đủ thông tin hóa đơn.

### Getter / Setter

Đầy đủ getter/setter cho tất cả 6 thuộc tính ở trên.

### Phương thức nghiệp vụ

| Phương thức | Mô tả |
|---|---|
| `createInvoice()` | Khởi tạo hóa đơn: gán `issueDate` là thời điểm hiện tại, `status = UNPAID`, và `totalAmount` lấy từ `order.getTotalPrice()`. |
| `printInvoice()` | In thông tin chi tiết hóa đơn ra console (mã hóa đơn, mã đơn hàng, ngày lập, trạng thái, tổng tiền, phương thức thanh toán, mã giao dịch nếu có). |
| `updateStatus(InvoiceStatus newStatus)` | Cập nhật trạng thái hóa đơn (ví dụ: từ "Chờ thanh toán" sang "Đã thanh toán"). |
| `getInvoiceById(String id)` | Tìm kiếm hóa đơn theo mã ID; trả về `this` nếu khớp, ngược lại trả về `null`. |
| `toString()` | Trả về chuỗi mô tả hóa đơn gồm `invoiceId`, `orderId`, `totalAmount`, `status`. |

---

## 2. InvoiceStatus (enum)

Định nghĩa các trạng thái của một hóa đơn.

| Giá trị | Ý nghĩa |
|---|---|
| `UNPAID` | Chưa thanh toán |
| `PAID` | Đã thanh toán |
| `CANCELLED` | Đã hủy |

---

## 3. Payment

Đại diện cho một giao dịch thanh toán ứng với một hóa đơn.

### Thuộc tính

| Thuộc tính | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `paymentId` | `String` | Mã định danh giao dịch thanh toán |
| `invoiceId` | `String` | Mã hóa đơn liên quan |
| `amount` | `Double` | Số tiền thanh toán |
| `paymentDate` | `Date` | Ngày thực hiện thanh toán |
| `method` | `PaymentMethod` | Phương thức thanh toán (Tiền mặt / Thẻ / Chuyển khoản) |
| `status` | `PaymentStatus` | Trạng thái thanh toán (mặc định `PENDING`) |

### Constructor

- `Payment()` — constructor mặc định.
- `Payment(String paymentId, String invoiceId, Double amount, Date paymentDate, PaymentMethod method)` — khởi tạo giao dịch, tự động gán `status = PENDING`.

### Getter / Setter

Đầy đủ getter/setter cho tất cả 6 thuộc tính ở trên.

### Phương thức nghiệp vụ

| Phương thức | Mô tả |
|---|---|
| `processPayment()` | Xử lý thanh toán: nếu `amount > 0` thì đặt `status = SUCCESS` và trả về `true`; ngược lại đặt `status = CANCELLED` và trả về `false`. |
| `recordTransaction()` | In thông tin giao dịch ra console (mã giao dịch, mã hóa đơn, số tiền, ngày thanh toán, phương thức, trạng thái). |
| `getPaymentStatus()` | Trả về trạng thái hiện tại của giao dịch (`status`). |
| `cancelPayment()` | Hủy giao dịch: đặt `status = CANCELLED` và `amount = 0.0`. |
| `toString()` | Trả về chuỗi mô tả giao dịch gồm `paymentId`, `invoiceId`, `amount`, `method`, `status`. |

---

## 4. PaymentMethod (enum)

Định nghĩa các phương thức thanh toán được hỗ trợ.

| Giá trị | Ý nghĩa |
|---|---|
| `CASH` | Tiền mặt |
| `CARD` | Thẻ ngân hàng |
| `BANK_TRANSFER` | Chuyển khoản |

---

## 5. PaymentStatus (enum)

Định nghĩa các trạng thái của một giao dịch thanh toán.

| Giá trị | Ý nghĩa |
|---|---|
| `PENDING` | Chờ thanh toán |
| `SUCCESS` | Thanh toán thành công |
| `CANCELLED` | Đã hủy thanh toán |

---

## Sơ đồ quan hệ (tổng quan)

```
Order 1 ---- 1 Invoice 1 ---- 1 Payment
                 |                  |
                 v                  v
           InvoiceStatus       PaymentMethod
           (UNPAID/PAID/        (CASH/CARD/
            CANCELLED)          BANK_TRANSFER)
                                     |
                                     v
                               PaymentStatus
                           (PENDING/SUCCESS/
                              CANCELLED)
```

- Một **Invoice** được tạo ra từ một **Order** và có thể liên kết với một **Payment**.
- Một **Payment** tham chiếu ngược lại `invoiceId` để biết mình thuộc hóa đơn nào.
- Cả hai class `Invoice` và `Payment` đều sử dụng enum riêng để quản lý trạng thái (`InvoiceStatus`, `PaymentStatus`), và `Payment` còn dùng thêm `PaymentMethod` để xác định hình thức thanh toán.
