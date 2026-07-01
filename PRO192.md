# Presentation Structure — Restaurant Management System

## Member 1 — User Management

### 1. Module Overview (30s)
- Chức năng của User Management
- Actor liên quan
- Input / Output

### 2. UML (45–60s)
#### Class Diagram
- User
- Role
- UserRepository
- UserService

**Giải thích:**
- Quan hệ giữa User và Role
- Cách Service tương tác với Repository

### 3. System Architecture (30–45s)

```text
UI
↓
UserService
↓
UserRepository
↓
Data
```

**Giải thích:**
- Module User nằm ở đâu trong toàn hệ thống
- Luồng xử lý dữ liệu

### 4. Code Analysis (1 phút)

#### Show:
- Login
- Validation
- CRUD User

#### Highlight:
- Interface
- Encapsulation
- Exception Handling

### 5. Testing (45s)

#### Test Cases
- Login success
- Wrong password
- Duplicate account

---

## Member 2 — Inventory

### 1. Module Overview
- Quản lý nguyên liệu
- Quản lý thực đơn

### 2. UML

#### Class Diagram
- InventoryItem
- Category
- MenuItem

**Giải thích:**
- Quan hệ giữa tồn kho và thực đơn

### 3. Architecture

```text
InventoryUI
↓
InventoryService
↓
Repository
```

### 4. Code Analysis

#### Show:
- Update stock
- Validation
- Search / Filter

#### Highlight:
- Repository Pattern
- Reuse

### 5. Testing

#### Test Cases
- Add item
- Update quantity
- Out-of-stock

---

## Member 3 — Order

### 1. Module Overview
- Tạo đơn
- Quản lý bàn
- Tính tiền tạm thời

### 2. UML

#### Class Diagram
- Order
- OrderDetail
- Table

**Quan trọng:**
- Giải thích vì sao tách `Order` và `OrderDetail`

### 3. Architecture

```text
OrderUI
↓
OrderService
↓
Repository
```

### 4. Code Analysis

#### Show:
- Create Order
- Calculate Total
- Update Status

#### Highlight:
- Polymorphism
- Business Logic

### 5. Testing

#### Test Cases
- Empty order
- Multiple items
- Cancel order

---

## Member 4 — Payment

### 1. Module Overview
- Thanh toán
- Hóa đơn
- Báo cáo

### 2. UML

#### Class Diagram
- Invoice
- Payment
- Transaction

### 3. Architecture

```text
PaymentUI
↓
PaymentService
↓
Repository
```

### 4. Code Analysis

#### Show:
- Payment process
- Invoice generation

#### Highlight:
- Data consistency
- Error handling

### 5. Testing

#### Test Cases
- Exact payment
- Invalid payment
- Revenue output

---

# Final Slide — Integration + Demo

## Full System Flow

```text
User
↓
Inventory
↓
Order
↓
Payment
```

## Demo Scenario

1. Login
2. Quản lý kho
3. Tạo Order
4. Thanh toán
5. Xuất kết quả

---

## Expected Outcome

Giảng viên sẽ thấy:

- Mỗi thành viên hiểu rõ module mình phụ trách
- Có đủ:
  - UML
  - Architecture
  - Code Analysis
  - Testing
- Các module kết nối thành một hệ thống hoàn chỉnh
- Không tạo cảm giác 4 bài thuyết trình tách rời
