---
title: Python Căn Bản
type: concept
tags: [python, programming, basics, beginner]
updated: 2026-05-27
sources: [conversation-2026-05-27]
---

# Python Căn Bản

## 1. Môi Trường

- **Google Colab** — code Python online, không cần cài đặt, phù hợp cho người mới học. Truy cập tại [colab.research.google.com](https://colab.research.google.com).

---

## 2. Biến (Variables)

- Python **phân biệt chữ hoa và chữ thường** — `name`, `Name`, `NAME` là 3 biến khác nhau.
- Không cần khai báo kiểu dữ liệu — Python tự nhận diện.
- Dùng **`snake_case`** theo convention (vd: `my_name`, không dùng `myName`).
- **Không dùng từ khóa đặt tên biến**: tránh `list`, `print`, `type`, `input`... vì sẽ ghi đè hàm built-in.
- Tên biến hợp lệ: chỉ dùng chữ cái, số, dấu `_`; không bắt đầu bằng số.

```python
name = "Bichi"    # ✅
Name = "Bichi"    # ✅ nhưng là biến khác
list = [1, 2, 3]  # ⚠️ tránh — ghi đè built-in list
```

---

## 3. Kiểu Dữ Liệu & Chuyển Kiểu

### True / False

Python dùng **`True` / `False`** (viết hoa chữ cái đầu), khác JS dùng `true / false`.

```python
print(1 == 1)   # → True
print(1 == 2)   # → False
```

### None (tương đương null trong JS)

```python
x = None

if x is None:       # ✅ cách kiểm tra chuẩn
    print("rỗng")
```

> Python **không có `undefined`** — dùng biến chưa khai báo sẽ báo lỗi `NameError` ngay.

### Hàm chuyển kiểu

```python
int('10')           # → 10        chuỗi → số nguyên
float('3.14')       # → 3.14      chuỗi → số thực
str(100)            # → '100'     số → chuỗi
bool(1)             # → True
bool(0)             # → False
bool('')            # → False     (chuỗi rỗng = False)
bool(None)          # → False

# Lưu ý
int('3.14')         # ❌ Lỗi — phải qua float trước
int(float('3.14'))  # ✅ → 3
```

---

## 4. Chuỗi (String)

### Nối chuỗi

```python
# ❌ Không thể cộng chuỗi với số trực tiếp
print('a' + 'b' + i)       # TypeError nếu i là số

# ✅ Cách 1 — str()
print('a' + 'b' + str(i))

# ✅ Cách 2 — f-string (khuyến khích, phổ biến nhất)
print(f'ab{i}')

# ✅ Cách 3 — dùng dấu phẩy (tự thêm dấu cách)
print('a', 'b', i)
```

### split()

```python
text = "xin chào bạn"
text.split(" ")    # → ['xin', 'chào', 'bạn']
```

---

## 5. List (Mảng)

```python
fruits = ['a', 'b', 'c', 'd', 'e']
print(fruits)   # → ['a', 'b', 'c', 'd', 'e']
```

### Slice — cắt list

```python
fruits[1:3]    # → ['b', 'c']        (từ index 1, không lấy index 3)
fruits[:3]     # → ['a', 'b', 'c']   (từ đầu đến index 2)
fruits[2:]     # → ['c', 'd', 'e']   (từ index 2 đến cuối)
fruits[-1]     # → 'e'               (phần tử cuối)
fruits[::-1]   # → ['e','d','c','b','a']  (đảo ngược)
```

### Các hàm list phổ biến

| Python | JavaScript | Ý nghĩa |
|---|---|---|
| `arr.append(x)` | `arr.push(x)` | Thêm vào cuối |
| `arr.pop()` | `arr.pop()` | Xóa phần tử cuối |
| `len(arr)` | `arr.length` | Độ dài |
| `"-".join(arr)` | `arr.join("-")` | Nối thành chuỗi |
| `arr[1:3]` | `arr.slice(1, 3)` | Cắt list |

> ⚠️ `join` trong Python **ngược JS**: Python viết `"-".join(list)` thay vì `list.join("-")`.

---

## 6. Vòng Lặp For

```python
# Cú pháp
for biến in đối_tượng:
    # code (indent bắt buộc)

# Lặp qua list
for fruit in ['apple', 'banana']:
    print(fruit)

# Lặp với range()
for i in range(10):       # 0 → 9   (không bao gồm 10)
    print(i)

for i in range(1, 6):     # 1 → 5
    print(i)

for i in range(0, 10, 2): # 0, 2, 4, 6, 8  (bước nhảy 2)
    print(i)

# Lặp qua chuỗi
for char in "Python":
    print(char)   # P, y, t, h, o, n
```

### Lưu ý

- **Indent bắt buộc** — code trong `for` phải thụt vào 4 spaces, không dùng `{}` như JS.
- **`range()` không bao gồm số cuối** — `range(1, 5)` chỉ ra `1, 2, 3, 4`.
- `break` — thoát vòng lặp ngay.
- `continue` — bỏ qua lần lặp hiện tại.
- `else` trong `for` — chạy khi vòng lặp kết thúc bình thường (không bị `break`).

---

## 7. Điều Kiện If

```python
x = 10
if x > 5:
    print("lớn hơn 5")
elif x == 5:
    print("bằng 5")
else:
    print("nhỏ hơn 5")
```

---

## 8. Input từ Người Dùng

```python
bichi = input('input your name: ')
print(bichi)
```

> `input()` **luôn trả về string** dù người dùng gõ số.

```python
# ❌ Lỗi nếu muốn tính toán
age = input('Tuổi: ')
print(age + 1)          # TypeError

# ✅ Chuyển sang int trước
age = int(input('Tuổi: '))
print(age + 1)          # ✅
```

---

## 9. So Sánh Python vs JavaScript

| | Python | JavaScript |
|---|---|---|
| Khai báo biến | `x = 5` | `let x = 5` |
| In ra màn hình | `print(x)` | `console.log(x)` |
| Block code | indent (tab/space) | `{}` |
| Comment | `# comment` | `// comment` |
| Boolean | `True / False` | `true / false` |
| Null | `None` | `null` |
| Undefined | ❌ không có | `undefined` |
| Nhận input | `input('...')` | `prompt('...')` |
| Chuyển sang số | `int()`, `float()` | `parseInt()`, `parseFloat()` |
| Slice | `arr[1:3]` | `arr.slice(1, 3)` |
