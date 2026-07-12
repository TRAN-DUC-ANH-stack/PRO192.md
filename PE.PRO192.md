# Cheat Sheet Xử Lý Chuỗi (String) - PE PRO192

Tổng hợp nhanh: đề ghi gì → nghĩ ngay code gì.

## 1. Truy xuất ký tự (Character Access)

| Đề ghi | Code |
|---|---|
| First character / first letter | `charAt(0)` hoặc `substring(0,1)` |
| Second character | `charAt(1)` |
| Third character | `charAt(2)` |
| Last character | `charAt(length-1)` |
| First n characters | `substring(0, n)` |
| Last n characters | `substring(length-n)` |

## 2. Xóa / Cắt chuỗi (Remove / Substring)

| Đề ghi | Code |
|---|---|
| Remove first (character/letter) | `substring(1)` |
| Remove last (character/letter) | `substring(0, length-1)` |
| Remove even characters (vị trí chẵn) | `for (i=0; i<length; i+=2)` — bỏ qua hoặc giữ tùy đề |
| Remove odd characters (vị trí lẻ) | `for (i=1; i<length; i+=2)` |
| Remove non-alphabet (chỉ giữ chữ) | `replaceAll("[^a-zA-Z]", "")` |
| Only digit (chỉ giữ số) | `replaceAll("[^0-9]", "")` |
| Only alphabet | `replaceAll("[^A-Za-z]", "")` |

## 3. Thêm / Nối chuỗi (Insert / Append)

| Đề ghi | Code |
|---|---|
| Insert vào đầu (before) | `"CF" + name` hoặc `number + string` |
| Insert vào cuối (after) | `name + "CF"` hoặc `string + number` |
| Append | `+` |
| Insert to beginning | `newPart + old` |
| Insert to end | `old + newPart` |
| toString (nối nhiều thông tin) | Nối chuỗi bằng `+` |

## 4. Đảo ngược (Reverse)

| Đề ghi | Code |
|---|---|
| Reverse | `new StringBuilder(str).reverse().toString()` hoặc vòng `for` chạy ngược |

## 5. Chữ hoa / Chữ thường (Case)

| Đề ghi | Code |
|---|---|
| Uppercase (toàn chuỗi) | `str.toUpperCase()` |
| Lowercase (toàn chuỗi) | `str.toLowerCase()` |
| Uppercase 1 ký tự | `Character.toUpperCase(ch)` |
| Lowercase 1 ký tự | `Character.toLowerCase(ch)` |

## 6. Kiểm tra điều kiện (Check / Validate)

| Đề ghi | Code |
|---|---|
| Contains digit | `Character.isDigit(ch)` |
| Contains letter | `Character.isLetter(ch)` |
| Check condition (điều kiện bất kỳ) | `if...else` |
| Replace | `str.replace(oldChar, newChar)` |

## 7. Định dạng số (Format)

| Đề ghi | Code |
|---|---|
| Format 2 decimals | `String.format("%.2f", value)` |
| Power (lũy thừa/tính toán) | `a * b` hoặc `Math.pow(a, b)` |

## 8. Thuộc tính & Cập nhật (Class-related)

| Đề ghi | Code |
|---|---|
| Length | `str.length()` |
| Update (cập nhật field) | `this.x = ...` |

## 9. Tìm kiếm trong chuỗi (Search)

| Đề ghi | Code |
|---|---|
| Contains (chứa chuỗi con) | `str.contains("abc")` |
| Index of (vị trí xuất hiện đầu) | `str.indexOf("abc")` |
| Last index of (vị trí xuất hiện cuối) | `str.lastIndexOf("abc")` |
| Starts with | `str.startsWith("abc")` |
| Ends with | `str.endsWith("abc")` |
| Equals (so sánh chính xác) | `str.equals(other)` |
| Equals ignore case | `str.equalsIgnoreCase(other)` |
| So sánh thứ tự từ điển | `str.compareTo(other)` |
| Đếm số lần xuất hiện ký tự | vòng `for` + `if (ch == target) count++` |

## 10. Cắt / Tách / Xử lý khoảng trắng

| Đề ghi | Code |
|---|---|
| Trim (xóa khoảng trắng đầu-cuối) | `str.trim()` hoặc `str.strip()` |
| Split (tách chuỗi thành mảng) | `str.split(" ")` hoặc `str.split(",")` |
| Join (nối mảng thành chuỗi) | `String.join(", ", array)` |
| Replace all | `str.replaceAll("regex", "replacement")` |
| Replace first | `str.replaceFirst("regex", "replacement")` |
| Convert to char array | `str.toCharArray()` |

## 11. StringBuilder (khi cần thay đổi chuỗi nhiều lần)

| Đề ghi | Code |
|---|---|
| Khởi tạo | `StringBuilder sb = new StringBuilder();` |
| Thêm vào cuối | `sb.append("abc")` |
| Chèn tại vị trí | `sb.insert(index, "abc")` |
| Xóa ký tự | `sb.deleteCharAt(index)` |
| Xóa đoạn | `sb.delete(start, end)` |
| Đảo ngược | `sb.reverse()` |
| Thay thế ký tự | `sb.setCharAt(index, ch)` |
| Chuyển về String | `sb.toString()` |

## 12. Regex thường dùng trong PE

| Mục đích | Regex |
|---|---|
| Chỉ chữ cái | `"[a-zA-Z]+"` |
| Chỉ chữ số | `"[0-9]+"` |
| Chữ và số (không ký tự đặc biệt) | `"[a-zA-Z0-9]+"` |
| Email cơ bản | `"^[\\w.-]+@[\\w.-]+\\.\\w+$"` |
| Số điện thoại VN (10 số, bắt đầu 0) | `"^0\\d{9}$"` |
| Không có ký tự đặc biệt | `"^[a-zA-Z0-9\\s]+$"` |
| Kiểm tra chuỗi khớp toàn bộ | `str.matches(regex)` |

## 13. Các bài toán xử lý chuỗi thường gặp (pattern code sẵn)

**Đếm nguyên âm (vowels):**
```java
int count = 0;
for (char c : str.toLowerCase().toCharArray()) {
    if ("aeiou".indexOf(c) != -1) count++;
}
```

**Kiểm tra chuỗi đối xứng (palindrome):**
```java
String reversed = new StringBuilder(str).reverse().toString();
boolean isPalindrome = str.equalsIgnoreCase(reversed);
```

**Viết hoa chữ cái đầu mỗi từ (capitalize each word):**
```java
String[] words = str.split(" ");
StringBuilder result = new StringBuilder();
for (String w : words) {
    result.append(Character.toUpperCase(w.charAt(0)))
          .append(w.substring(1))
          .append(" ");
}
```

**Đếm số lần xuất hiện của 1 ký tự:**
```java
int count = 0;
for (char c : str.toCharArray()) {
    if (c == 'a') count++;
}
```

**Kiểm tra chuỗi chỉ chứa số (để validate input):**
```java
boolean isNumeric = str.matches("[0-9]+");
```

---

## Lưu ý khi làm bài

1. **Đọc kỹ output mẫu trước khi code** — soi từng dấu cách, dấu `-`, số thập phân.
2. **Chạy thử và so sánh output từng ký tự** với đề bài, đừng chỉ nhìn lướt.
3. Khi đề nói "ký tự chẵn/lẻ" cần xác định rõ: tính theo **vị trí index (0,1,2...)** hay **thứ tự thường (1,2,3...)** vì lệch nhau 1 sẽ sai kết quả.
4. `substring(a, b)`: lấy từ index `a` đến **trước** index `b` (không bao gồm `b`).
5. Luôn kiểm tra chuỗi rỗng / `null` / độ dài trước khi gọi `charAt()` hay `substring()` để tránh lỗi `StringIndexOutOfBoundsException`.
