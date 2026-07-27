# CHECKLIST CHUẨN BỊ BẢO VỆ ĐỒ ÁN — PRO192
**Module 4: Payment & Invoice Management | Trần Đức Anh — SE203405**

> Mục tiêu: đi từ "hiểu code mình viết" → "hiểu cả dự án nhóm" → "trình bày & vấn đáp trôi chảy trong 30 phút", đáp ứng đúng cấu trúc Quick Card GV chấm.

---

## PHẦN A — HIỂU TOÀN CẢNH DỰ ÁN NHÓM

### A1. Vẽ lại bức tranh tổng ("bird's-eye view")
- [ ] Vẽ tay (hoặc mô tả miệng) sơ đồ 4 tầng kiến trúc chung của cả nhóm: **UI → Service → Repository → Model/Enum**
- [ ] Liệt kê các **module** trong dự án và ai phụ trách module nào (VD: Module 3 = Order, Module 4 = Payment & Invoice — của bạn)
- [ ] Xác định rõ: dự án lưu dữ liệu bằng **file text** (không dùng database) — hiểu vì sao (đây là điểm PRO192 hay hỏi: "sao không dùng DB?")

### A2. Hiểu luồng nghiệp vụ chính từ đầu đến cuối
- [ ] Vẽ luồng: `Order được tạo (Module 3)` → `Invoice được tạo từ Order (Module 4)` → `Payment được tạo & confirm (Module 4)` → `Invoice/Order cập nhật trạng thái đã thanh toán`
- [ ] Trả lời được: **nếu bỏ Module 4 đi, hệ thống thiếu gì?** (không thanh toán được, Order không bao giờ đóng)
- [ ] Hiểu vai trò `Main.java` (hoặc entry point của nhóm): nơi khởi tạo và inject tất cả repository/service/UI vào nhau

### A3. Việc cần làm ngay
- [ ] Xin lại (hoặc tự tổng hợp) **sơ đồ UML tổng của cả dự án**, không chỉ Module_4.png, để biết Module 4 nằm ở đâu trong bức tranh lớn
- [ ] Hỏi các thành viên khác 1 câu duy nhất mỗi người: *"Module của bạn expose ra ngoài (public method) những gì mà Module 4 có thể cần dùng?"*

---

## PHẦN B — HIỂU CODE PHẦN MÌNH LÀM (Module 4)

Học theo đúng 4 tầng, từ dưới lên (Model trước, vì mọi thứ đều dựa vào Model):

### B1. Tầng Model & Enum (nền tảng)
- [ ] `Invoice`: thuộc tính, 2 constructor, và **5 method nghiệp vụ** — giải thích được từng method dùng khi nào trong luồng thật (không chỉ đọc định nghĩa)
- [ ] `Payment`: đặc biệt hiểu kỹ `processPayment()` (logic if/else amount > 0) và `cancelPayment()` (set amount = 0.0) — đây là 2 method hay bị hỏi xoáy
- [ ] `InvoiceStatus` (UNPAID/PAID/CANCELLED) vs `PaymentStatus` (PENDING/SUCCESS/CANCELLED) — giải thích được **vì sao KHÔNG gộp chung 1 enum** (đã có sẵn lý luận trong log Entry #6)
- [ ] `PaymentMethod` (CASH/CARD/BANK_TRANSFER) — chỉ dùng ở đâu

**Tự kiểm tra:** Che tài liệu lại, tự vẽ được class diagram của 5 class Model trên giấy trắng.

### B2. Tầng Repository (đọc/ghi file)
- [ ] Hiểu cơ chế chung: `BaseRepository` giữ `data` (List trong RAM) + `filePath`, tự load khi khởi tạo, `saveToFile()` khi cần lưu
- [ ] Học thuộc **định dạng dòng dữ liệu** của từng file:
  - Invoice: `invoiceId|orderId|totalAmount|issueDate|status`
  - Payment: `paymentId|invoiceId|amount|paymentDate(timestamp)|status|method`
- [ ] Giải thích được vì sao `PaymentRepository.save()` phải **override** (xóa bản ghi cũ trước khi thêm mới) — tránh trùng lặp file
- [ ] Hiểu `parseLine()`/`toLine()` là cặp method **abstract** — mỗi Repository con tự quyết định cách parse riêng (Template Method pattern)

**Tự kiểm tra:** Tự viết tay 1 dòng text giả lập cho Invoice và Payment, rồi tự "parse" bằng miệng ra từng field.

### B3. Tầng Service (business logic)
- [ ] `InvoiceService.createInvoiceFromOrder()` — thuộc lòng 6 bước (validate → check trùng → sinh ID → tạo Invoice → save → return)
- [ ] `PaymentService.createPayment()` → `confirmPayment()` → `cancelPayment()` — vẽ được sơ đồ trạng thái `PENDING → SUCCESS/CANCELLED`
- [ ] **Trọng tâm:** giải thích được guard clause `if (status != PENDING) return false` trong `confirmPayment()` — đây là lỗi AI bạn tự phát hiện, GV rất thích hỏi lại
- [ ] Vì sao `PaymentService` phải **inject `InvoiceService`** (không phải `InvoiceRepository`) — nguyên tắc không cross-call repository của module khác

### B4. Tầng UI (điều phối)
- [ ] Học thuộc menu 4 lựa chọn và luồng `processCheckout()` — 6 bước từ nhập invoiceId đến in "Payment successful."
- [ ] **QUAN TRỌNG — cần xác minh trước:** constructor `PaymentUI` thật sự nhận mấy service? (2 hay 3 — xem lại phần đối chiếu vênh dữ liệu bên dưới)
- [ ] Hiểu try/catch: vì sao UI luôn bọc try/catch quanh các lời gọi service (tránh crash chương trình khi lỗi nghiệp vụ)

---

## PHẦN C — HIỂU LIÊN KẾT VỚI MODULE CỦA THÀNH VIÊN KHÁC

### C1. Điểm nối trực tiếp với Module 3 (Order)
- [ ] Xác định chính xác **1 trong 2** phương án đang chạy thật trong code (bắt buộc kiểm tra file `.java` gốc):
  - **Phương án A:** `PaymentUI` được inject `OrderService`, tự gọi `orderService.checkoutOrder()` sau khi thanh toán thành công
  - **Phương án B:** `InvoiceService.markAsPaid()` tự cập nhật `invoice.getOrder().setPaid(true)` nội bộ, Module 4 không biết đến `OrderService`
  
  → **Đây là điểm mâu thuẫn giữa tài liệu UI và Audit Log Entry #14 đã phát hiện ở lượt trò chuyện trước — bắt buộc xác minh code thật và chốt 1 phương án duy nhất trước khi bảo vệ.**

- [ ] Sau khi chốt, tập giải thích 30 giây: *"Module 4 liên kết với Module 3 qua đâu, và vì sao chọn cách đó chứ không phải cách còn lại (tránh cross-dependency)"*

### C2. Điểm nối gián tiếp (dùng chung)
- [ ] `IService<T,ID>` / `BaseService<T,ID>` / `IRepository<T,ID>` / `BaseRepository<T,ID>` — đây là các class **dùng chung toàn nhóm**, không phải riêng Module 4. Hỏi ai là người thiết kế gốc 4 class này, để không bị hỏi ngược mà không biết trả lời
- [ ] Nếu nhóm có class `AppConfig` chứa đường dẫn file chung — hiểu vì sao tập trung config 1 chỗ thay vì hardcode ở từng Repository

### C3. Việc cần làm
- [ ] Vẽ 1 sơ đồ đơn giản: Module 4 (ô ở giữa) — mũi tên vào từ đâu, mũi tên ra đi đâu, sang module nào
- [ ] Chuẩn bị câu trả lời cho câu hỏi kinh điển: *"Nếu Module 3 đổi cấu trúc Order, Module 4 của bạn có bị ảnh hưởng không? Ảnh hưởng ở đâu?"*

---

## PHẦN D — HIỂU CHỨC NĂNG & CHUẨN BỊ DEMO

### D1. Kịch bản demo cần chạy thử trước (không để lỗi khi trình bày thật)
- [ ] Demo 1 — Happy path: tạo Order → tạo Invoice → chọn phương thức thanh toán → confirm → xem status chuyển UNPAID → PAID
- [ ] Demo 2 — Trường hợp lỗi: nhập `invoiceId` không tồn tại → phải in đúng thông báo lỗi, **không crash**
- [ ] Demo 3 — Trường hợp biên: confirm 1 payment đã SUCCESS lần 2 → phải bị chặn (guard clause) — chuẩn bị sẵn để demo trực tiếp nếu GV yêu cầu "Live Modification"
- [ ] Demo 4 — Xem danh sách tất cả hóa đơn (`viewAllInvoices`) khi danh sách rỗng và khi có dữ liệu

### D2. Chuẩn bị cho "Live Modification" (GV yêu cầu sửa/giải thích trực tiếp)
- [ ] Tập sửa nhanh 1 đoạn code đơn giản ngay tại chỗ, ví dụ: đổi điều kiện `amount > 0` thành `amount >= 1000` và giải thích ảnh hưởng
- [ ] Tập giải thích **từng dòng** trong `confirmPayment()` mà không cần nhìn màn hình

### D3. Đối chiếu Slide/Report với sản phẩm thật
- [ ] Kiểm tra slide có đúng số lượng class, đúng tên method như code thật không (GV sẽ đối chiếu trực tiếp)
- [ ] Đảm bảo screenshot/demo trong slide là bản build **mới nhất**, không phải bản cũ

---

## PHẦN E — ĐỐI CHIẾU LẠI VỚI QUICK CARD (đảm bảo đủ tiêu chí GV chấm)

### E1. Trước buổi chấm — hồ sơ
- [ ] Report/Slide/App đã nộp đúng trên LMS
- [ ] AI Audit Log đã nộp — **và đã sửa xong 2 điểm vênh (Entry #3, #14) nêu ở Phần C1**
- [ ] Link Drive/GitHub hoạt động, phân quyền GV xem được

### E2. Timeline 30 phút — tập dượt đúng nhịp
| Phút | Nội dung | Bạn cần chuẩn bị gì |
|---|---|---|
| 0–1' | GV thông báo & sinh viên ký E360 | Không cần chuẩn bị, chỉ cần lắng nghe kỹ |
| 2–6' | Demo & trình bày | Kịch bản Demo 1 ở Phần D1, nói rõ ràng, khớp slide |
| 5–20' | Vấn đáp DTC | Ôn 4 nhóm câu hỏi ở Phần B (Model/Repo/Service/UI) theo đúng Decomposition/Pattern/Abstraction/Algorithms |
| 20–28' | AI Audit Log | Ôn kỹ 6 entry trọng điểm: #2, #3, #5, #7, #9, #11, #14 |
| 28–30' | Chốt điểm | Lắng nghe, xác nhận đúng nội dung trước khi ký E360 |

### E3. Checklist AI Audit Log (theo đúng tiêu chí Quick Card)
- [x] Log có cấu trúc chuỗi core prompt — **đã đạt (14 entries)**
- [x] Prompt gắn với project cụ thể, không hỏi chung chung — **đã đạt**
- [x] Self-assessment checklist đầy đủ — **đã đạt**
- [x] Chỉ ra ≥1 lỗi AI (thực tế bạn có 5, vượt yêu cầu ≥3) — **đã đạt**
- [ ] SV giải thích được đã sửa lỗi thế nào — **cần luyện nói to, không chỉ đọc log**
- [ ] Human Delta: SV tự làm gì ngoài AI — **cần chuẩn bị ví dụ cụ thể ngoài log để trả lời tự nhiên, không như học thuộc**

### E4. Tránh rơi vào "dấu hiệu nghi vấn"
- [ ] Đảm bảo mọi entry trong log **khớp với code cuối cùng nộp** (không phải bản đã bị thay đổi mà log chưa cập nhật) — xem lại Phần C1
- [ ] Có thể giải thích **bằng lời riêng**, không đọc lại nguyên văn log
- [ ] Nếu có prompt tiếng Anh trong log, phải hiểu và diễn giải lại được bằng tiếng Việt trôi chảy

---

## THỨ TỰ ÔN TẬP ĐỀ XUẤT (nếu chỉ còn ít thời gian)

1. **Ưu tiên 1 (bắt buộc):** Xác minh & sửa vênh dữ liệu Entry #3, #14 với code thật (Phần C1)
2. **Ưu tiên 2:** Luyện nói 6 entry trọng điểm trong Audit Log không nhìn giấy
3. **Ưu tiên 3:** Chạy thử đủ 4 kịch bản demo (Phần D1), đảm bảo không lỗi
4. **Ưu tiên 4:** Ôn 4 nhóm câu hỏi DTC theo bảng ở Phần B
5. **Ưu tiên 5:** Vẽ tay sơ đồ liên kết Module 4 ↔ Module 3 (Phần C3) để trả lời tự nhiên khi bị hỏi bất ngờ
