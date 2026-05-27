---
title: Python Advanced File Writing
type: concept
tags: [python, file-io, programming]
updated: 2026-05-27
sources: [conversation-2026-05-27]
---

# Cơ chế ghi file trong Python

## File Write Mode `'w'`

Khi mở file với mode `'w'`, Python sẽ **xóa toàn bộ nội dung cũ** (truncate về 0 byte) và ghi lại từ đầu mỗi lần mở.

### Ví dụ

```python
file2 = open('myfile.txt', 'w')
for i in range(10):
    file2.write(str(i) + "a" + "\n")
file2.close()
```

Chạy đoạn code trên 3 lần, kết quả **luôn giống nhau** (file chỉ chứa 10 dòng):

```
0a
1a
2a
3a
4a
5a
6a
7a
8a
9a
```

Lý do: mỗi lần `open(..., 'w')` sẽ reset file về rỗng trước khi vòng lặp ghi.

---

## So sánh các File Mode phổ biến

| Mode | Hành vi |
|------|---------|
| `'w'` | Tạo mới hoặc **xóa trắng** rồi ghi |
| `'a'` | Tạo mới hoặc **nối thêm** vào cuối (append) |
| `'r'` | Chỉ đọc, lỗi nếu file không tồn tại |
| `'x'` | Tạo mới, lỗi nếu file đã tồn tại |
| `'rb'`, `'wb'` | Binary mode (ảnh, PDF, v.v.) |

### Mode `'a'` — Append

Nếu thay `'w'` bằng `'a'`, chạy 3 lần sẽ có **30 dòng** (mỗi lần nối thêm 10 dòng vào cuối).

---

## Best Practice: dùng `with` statement

Nên dùng context manager để đảm bảo file luôn được đóng, kể cả khi có lỗi xảy ra:

```python
with open('myfile.txt', 'w') as file2:
    for i in range(10):
        file2.write(str(i) + "a" + "\n")
# file tự đóng khi ra khỏi block 'with'
```

### Lợi ích
- File luôn được đóng đúng cách (không bị lock hay mất dữ liệu).
- Code ngắn gọn, không cần gọi `.close()` thủ công.
- An toàn hơn khi có exception.
