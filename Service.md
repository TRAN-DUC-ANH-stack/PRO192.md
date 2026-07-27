# Package `com.mycompany.restaurantmanagement.service`

Tài liệu mô tả các class thuộc package **service** trong dự án Restaurant Management, bao gồm: `InvoiceService`, `PaymentService`.

> Lưu ý: theo khai báo `package` trong file gốc, hai class này thuộc package `service` (không phải `repository`). Chúng sử dụng các interface trong package `repository` (`InvoiceRepository`, `PaymentRepository`) để thao tác dữ liệu, và kế thừa từ `BaseService<T, ID>` chung.

---

## 1. InvoiceService

Kế thừa: `BaseService<Invoice, String>`

Xử lý nghiệp vụ liên quan đến hóa đơn (`Invoice`), thao tác dữ liệu thông qua `InvoiceRepository`.

### Thuộc tính

| Thuộc tính | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `invoiceRepository` | `InvoiceRepository` (final) | Repository dùng để lưu trữ/truy xuất `Invoice` |

### Constructor

- `InvoiceService(InvoiceRepository invoiceRepository)` — gọi `super(invoiceRepository)` để khởi tạo `BaseService`, đồng thời gán vào field riêng để dùng cho các truy vấn đặc thù.

### Phương thức nghiệp vụ

| Phương thức | Mô tả |
|---|---|
| `createInvoiceFromOrder(Order order)` | Tạo hóa đơn mới từ một `Order`. Kiểm tra `order != null`, sinh `invoiceId` theo dạng `"INV" + timestamp`, khởi tạo `Invoice` với `payment = null`, `totalAmount = order.getTotalPrice()`, `issueDate` hiện tại, `status = UNPAID`, sau đó lưu qua `add(invoice)` và trả về hóa đơn vừa tạo. |
| `markAsPaid(String invoiceId, Payment payment)` | Đánh dấu hóa đơn đã thanh toán: tìm hóa đơn theo `invoiceId` (báo lỗi nếu không tìm thấy), gán `payment` cho hóa đơn, cập nhật `status = PAID` (qua `invoice.updateStatus`), rồi lưu lại bằng `update(invoice)`. |
| `printInvoice(String invoiceId)` | Tìm hóa đơn theo ID (báo lỗi nếu không tìm thấy) và gọi `invoice.printInvoice()` để in thông tin ra console. |
| `getByOrderId(String orderId)` | Truy vấn trực tiếp qua `invoiceRepository.findByOrderId(orderId)` để lấy hóa đơn theo mã đơn hàng. |

1. ("Phương thức này dùng để tạo hóa đơn từ một đơn hàng.

phương thức createInvoiceFromOrder nằm trong lớp Service, đóng vai trò chuyển đổi một Đơn hàng (Order) thành một Hóa đơn (Invoice) chuẩn bị cho quá trình thanh toán."

Đầu tiên, nó kiểm tra đơn hàng đầu vào — nếu order là null, thì đơn hàng không hợp lệ, và ném ra ngoại lệ IllegalArgumentException ngay, với thông báo 'Order không hợp lệ" 

Nếu hợp lệ, nó sinh ra một mã hóa đơn tự động  thời gian hiện tại, ví dụ INV1623456789012, để đảm bảo mã không bị trùng.

Sau đó, tạo hóa đơn mới: mã vừa sinh, đơn hàng gốc, Payment để null vì chưa thanh toán, tổng tiền lấy trực tiếp từ đơn hàng, ngày phát hành là thời điểm hiện tại, và trạng thái mặc định là UNPAID.

Cuối cùng, gọi add(invoice) — kế thừa từ BaseService — để lưu hóa đơn vào kho dữ liệu, rồi trả về 

2. "Phương thức này dùng để đánh dấu một hóa đơn đã được thanh toán.

markAsPaid là một phương thức để nhận vào ID hóa đơn (invoiceId) và thông tin thanh toán (payment)

Đầu tiên, nó tìm hóa đơn trong kho dữ liệu dựa trên invoiceId. Nếu không tìm thấy — tức invoice trả về null — thì ném ngoại lệ IllegalArgumentException ngay, báo "Không tìm thấy hóa đơn".

Nếu tìm thấy, nó gán thông tin giao dịch thanh toán vào hóa đơn, rồi cập nhật trạng thái hóa đơn sang PAID 

Cuối cùng, gọi update(invoice) — kế thừa từ BaseService — để cập nhật hóa đơn vào kho lưu trữ"

printInvoice(String invoiceId)

3. "Phương thức này dùng để in thông tin hóa đơn

Đầu tiên, nó tìm hóa đơn trong kho dữ liệu dựa trên invoiceId. Nếu không tìm thấy — tức invoice trả về null — thì ném ngoại lệ IllegalArgumentException ngay, báo "Không tìm thấy hóa đơn".

Nếu tìm thấy,nó sẽ in thông tin hóa đơn ra màn hình console. 

'→ Điểm đáng nói khi trình bày: đây là ví dụ cho Rich Domain Model — logic in ấn nằm ngay trong Invoice (Model), còn InvoiceService chỉ đóng vai trò tìm đối tượng rồi gọi lại, không tự viết logic in. Nếu GV hỏi "tại sao không in trực tiếp trong Service?", đây chính là câu trả lời: giữ trách nhiệm hiển thị thông tin của chính đối tượng nằm trong đối tượng đó (tự nó biết cách mô tả chính nó), Service chỉ điều phối.'

4. getByOrderId(String orderId)

"Phương thức này dùng để tìm hóa đơn dựa trên mã đơn hàng
Nó chỉ đơn giản gọi thẳng xuống invoiceRepository.findByOrderId(orderId) để lấy đúng hóa đơn và trả về kết quả 

→ Điểm đáng lưu ý: đây là method "pass-through" (ủy quyền thẳng xuống Repository), khác với các method khác trong class đều có validate/xử lý logic. Nếu bị hỏi "vì sao method này không throw exception khi không tìm thấy như 2 method kia?" — bạn cần kiểm tra lại: hiện tại nếu không tìm thấy, nó sẽ trả về null (theo mô tả findByOrderId trong InvoiceRepository), khác với printInvoice/markAsPaid là ném exception ngay. Đây là điểm không nhất quán trong xử lý lỗi giữa các method — có thể chủ động nêu ra như một hạn chế bạn nhận thấy (rất hợp với chiến lược "chủ động nói điểm chưa hoàn hảo" đã bàn trước đó).

### Xử lý ngoại lệ

- Ném `IllegalArgumentException` khi:
  - `order == null` trong `createInvoiceFromOrder`.
  - Không tìm thấy hóa đơn theo ID trong `markAsPaid` và `printInvoice`.

---

## 2. PaymentService

Kế thừa: `BaseService<Payment, String>`

Xử lý nghiệp vụ liên quan đến giao dịch thanh toán (`Payment`), thao tác dữ liệu thông qua `PaymentRepository`, đồng thời phối hợp với `InvoiceService`.

### Thuộc tính

| Thuộc tính | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `paymentRepository` | `PaymentRepository` (final) | Repository dùng để lưu trữ/truy xuất `Payment` |
| `invoiceService` | `InvoiceService` (final) | Service hóa đơn, dùng để phối hợp cập nhật trạng thái hóa đơn khi thanh toán |

### Constructor

- `PaymentService(PaymentRepository paymentRepository, InvoiceService invoiceService)` — gọi `super(paymentRepository)` để khởi tạo `BaseService`, đồng thời gán cả hai field riêng.

### Phương thức nghiệp vụ

| Phương thức | Mô tả |
|---|---|
| `createPayment(String invoiceId, double amount, PaymentMethod method)` | Tạo giao dịch thanh toán mới. Kiểm tra `invoiceId` không rỗng và `amount > 0` (ngược lại ném `IllegalArgumentException`). Sinh `paymentId` theo dạng `"PAY" + timestamp`, khởi tạo `Payment` với ngày hiện tại và `status = PENDING`, lưu qua `add(payment)`, trả về giao dịch vừa tạo. |
| `confirmPayment(String paymentId)` | Xác nhận thanh toán: tìm giao dịch theo ID (báo lỗi nếu không tìm thấy), gọi `payment.processPayment()`; nếu thành công thì đặt `status = SUCCESS` và lưu lại qua `update(payment)`. Trả về kết quả xử lý (`true`/`false`). |
| `cancelPayment(String paymentId)` | Hủy giao dịch: tìm giao dịch theo ID (báo lỗi nếu không tìm thấy), gọi `payment.cancelPayment()`, đặt `status = CANCELLED`, rồi lưu lại qua `update(payment)`. |

createPayment(String invoiceId, double amount, PaymentMethod method)

"Tạo một giao dịch thanh toán mới. Kiểm tra 2 điều kiện đầu vào: invoiceId không được rỗng/null, và amount phải lớn hơn 0 — sai điều kiện nào thì ném lỗi ngay. Nếu hợp lệ, sinh mã giao dịch thanh toán tự động theo thời gian hiện tại, tạo đối tượng Payment mới, gán trạng thái PENDING, sau đó dùng hàm add(payment) để lưu vào kho dữ liệu và trả về."

confirmPayment(String paymentId)

"Xác nhận một giao dịch thanh toán. Tìm kím thông tin thanh toán trong kho lưu trữ theo paymentId — không thấy thì ném lỗi. Thấy rồi thì gọi payment.processPayment() — đây là nơi ủy quyền để xử lý logic (kiểm tra amount > 0 để quyết định thành công hay không). Nếu thành công, cập nhật trạng thái sang SUCCESS và lưu lại. Trả về true/false tùy kết quả."

cancelPayment(String paymentId)

"Hủy một giao dịch thanh toán. Tìm  thông tin thanh toán trong kho lưu trữ theo paymentId — không thấy thì ném lỗi. Thấy rồi thì gọi payment.cancelPayment() để hủy giao dịch (đặt amount = 0), cập nhật trạng thái sang CANCELLED, rồi lưu lại."

### Xử lý ngoại lệ

- Ném `IllegalArgumentException` khi:
  - `invoiceId` rỗng/null hoặc `amount <= 0` trong `createPayment`.
  - Không tìm thấy giao dịch theo ID trong `confirmPayment` và `cancelPayment`.

---

## Luồng nghiệp vụ tổng quan

```
Order
  │
  ▼
InvoiceService.createInvoiceFromOrder(order)
  │  → Invoice (status = UNPAID, payment = null)
  ▼
PaymentService.createPayment(invoiceId, amount, method)
  │  → Payment (status = PENDING)
  ▼
PaymentService.confirmPayment(paymentId)
  │  → Payment.processPayment() → status = SUCCESS
  ▼
InvoiceService.markAsPaid(invoiceId, payment)
  │  → Invoice.setPayment(payment), status = PAID
  ▼
InvoiceService.printInvoice(invoiceId)
```

- `InvoiceService` và `PaymentService` đều kế thừa `BaseService<T, ID>` (cung cấp các thao tác CRUD chung: `add`, `update`, `getById`, ...).
- `InvoiceService` giao tiếp với `InvoiceRepository`; `PaymentService` giao tiếp với `PaymentRepository`.
- `PaymentService` phụ thuộc vào `InvoiceService` để có thể liên kết giao dịch thanh toán với hóa đơn tương ứng (dependency injection qua constructor).
