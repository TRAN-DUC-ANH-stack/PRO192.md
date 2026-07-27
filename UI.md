# Package `com.mycompany.restaurantmanagement.ui`

Tài liệu mô tả class thuộc package **ui** trong dự án Restaurant Management: `PaymentUI`.

Class này đóng vai trò giao diện console (menu tương tác qua `Scanner`), điều phối giữa người dùng và các tầng **service** (`InvoiceService`, `PaymentService`, `OrderService`) để thực hiện nghiệp vụ thanh toán hóa đơn.

---

## 1. PaymentUI

### Thuộc tính

| Thuộc tính | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `invoiceService` | `InvoiceService` (final) | Service xử lý nghiệp vụ hóa đơn |
| `paymentService` | `PaymentService` (final) | Service xử lý nghiệp vụ thanh toán |
| `orderService` | `OrderService` (final) | Service xử lý nghiệp vụ đơn hàng |
| `scanner` | `Scanner` (final) | Đọc dữ liệu nhập từ người dùng qua console |

### Constructor

- `PaymentUI(InvoiceService invoiceService, PaymentService paymentService, OrderService orderService)` — nhận vào 3 service qua dependency injection, khởi tạo `Scanner` gắn với `System.in`.

### Phương thức chính

| Phương thức | Mô tả |
|---|---|
| `run()` | Vòng lặp menu chính hiển thị 4 lựa chọn: `1` Thanh toán, `2` Tìm hóa đơn, `3` Xem tất cả hóa đơn, `0` Thoát. Đọc lựa chọn từ người dùng, gọi phương thức tương ứng; bắt lỗi nếu nhập không hợp lệ (không phải số) và in `"Invalid input."`. |
| `viewInvoice()` | Yêu cầu nhập `invoiceId`, gọi `invoiceService.printInvoice(invoiceId)` để in thông tin hóa đơn; bắt và in ra thông báo lỗi nếu có (`e.getMessage()`). |
| `viewAllInvoices()` | Lấy toàn bộ danh sách hóa đơn qua `invoiceService.getAll()`. Nếu danh sách rỗng thì in `"No invoices found."`; ngược lại in ra từng hóa đơn (mã hóa đơn, tổng tiền, trạng thái) theo dạng danh sách. |
| `processCheckout()` | Luồng xử lý thanh toán chính: <br>1. Nhập `invoiceId`, tìm hóa đơn qua `invoiceService.getById(invoiceId)` (nếu không tìm thấy → in `"Invoice not found."` và dừng). <br>2. In chi tiết hóa đơn (`invoice.printInvoice()`). <br>3. Gọi `choosePaymentMethod()` để chọn phương thức thanh toán (nếu `null` thì dừng). <br>4. Tạo giao dịch thanh toán qua `paymentService.createPayment(...)` với số tiền lấy từ `invoice.getTotalAmount()`. <br>5. Xác nhận thanh toán qua `paymentService.confirmPayment(...)`. <br>6. Nếu thành công: cập nhật hóa đơn sang trạng thái đã thanh toán (`invoiceService.markAsPaid(...)`), thực hiện checkout đơn hàng liên quan (`orderService.checkoutOrder(...)`), in `"Payment successful."`. Nếu thất bại: in `"Payment failed."`. |

### Phương thức hỗ trợ (private)

| Phương thức | Mô tả |
|---|---|
| `choosePaymentMethod()` | Hiển thị menu chọn phương thức thanh toán (`1` CASH, `2` CARD, `3` BANK TRANSFER), đọc lựa chọn và trả về `PaymentMethod` tương ứng; nếu lựa chọn không hợp lệ thì in `"Invalid selection."` và trả về `null`. |

### Xử lý ngoại lệ

- Tất cả các thao tác chính (`viewInvoice`, `viewAllInvoices`, `processCheckout`) được bọc trong `try/catch` để bắt lỗi phát sinh từ tầng service (ví dụ: không tìm thấy hóa đơn/giao dịch) và hiển thị `e.getMessage()` cho người dùng thay vì làm crash chương trình.
- `run()` bắt lỗi riêng khi người dùng nhập lựa chọn không phải số nguyên hợp lệ.

---

## Luồng tương tác tổng quan

```
Người dùng
   │
   ▼
PaymentUI.run()  ──► Menu: 1) Thanh toán  2) Tìm hóa đơn  3) Xem tất cả  0) Thoát
   │
   ├── processCheckout()
   │       │
   │       ├─► invoiceService.getById(invoiceId)
   │       ├─► choosePaymentMethod()
   │       ├─► paymentService.createPayment(...)
   │       ├─► paymentService.confirmPayment(...)
   │       ├─► invoiceService.markAsPaid(...)     (nếu thành công)
   │       └─► orderService.checkoutOrder(...)     (nếu thành công)
   │
   ├── viewInvoice()        ──► invoiceService.printInvoice(invoiceId)
   │
   └── viewAllInvoices()    ──► invoiceService.getAll()
```

- **`PaymentUI`** thuộc tầng giao diện (Presentation/UI Layer), không thao tác trực tiếp với dữ liệu mà luôn gọi qua các **service** tương ứng (`InvoiceService`, `PaymentService`, `OrderService`), tuân theo kiến trúc phân lớp (UI → Service → Repository → Model) của dự án.
- Đây là điểm điều phối (orchestration point) cho toàn bộ quy trình thanh toán: từ chọn hóa đơn, chọn phương thức, tạo giao dịch, xác nhận, đến cập nhật trạng thái hóa đơn và đơn hàng liên quan.
