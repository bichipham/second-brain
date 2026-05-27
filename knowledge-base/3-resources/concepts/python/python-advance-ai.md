---
title: Python Nâng Cao cho AI (Level 71–80C)
type: concept
tags: [python, pandas, numpy, matplotlib, machine-learning, ai, data-science]
updated: 2026-05-27
sources: [Python4AI-Ultimate-docx, conversation-2026-05-27]
---

# Python Nâng Cao cho AI — Level 71 → 80C

> Tổng hợp 12 chủ đề Python ứng dụng AI, kèm ví dụ thực tế dễ hiểu.

---

## Level 71 — CSV Files

CSV (Comma-Separated Values) là định dạng lưu dữ liệu dạng bảng, phổ biến nhất trong data science.

```python
import csv

# Đọc file CSV thủ công
with open('hoc_sinh.csv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# Ghi file CSV
with open('ket_qua.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['Ten', 'Diem'])
    writer.writerow(['An', 85])
    writer.writerow(['Binh', 92])
```

**Ví dụ thực tế:** Xuất báo cáo điểm thi từ hệ thống ra file CSV để gửi email.

---

## Level 72 — Đọc CSV bằng Pandas

Pandas giúp đọc và xử lý CSV nhanh hơn nhiều so với module `csv` thuần.

```python
import pandas as pd

# Đọc file
df = pd.read_csv('hoc_sinh.csv')

# Xem nhanh dữ liệu
print(df.head())       # 5 dòng đầu
print(df.shape)        # (số hàng, số cột)
print(df.info())       # kiểu dữ liệu từng cột
print(df.describe())   # thống kê cơ bản

# Lọc & tính toán
df[df['diem'] >= 80]                    # học sinh đạt loại giỏi
df.groupby('lop')['diem'].mean()        # điểm trung bình từng lớp
df.sort_values('diem', ascending=False) # xếp hạng

# Lưu kết quả ra file mới
df.to_csv('ket_qua_xu_ly.csv', index=False)
```

**Ví dụ thực tế:** Phân tích doanh thu bán hàng tháng từ file CSV xuất từ phần mềm kế toán.

---

## Level 73 — Trích xuất dữ liệu bằng iloc

`iloc` (integer location) dùng để chọn dữ liệu theo **vị trí số** (hàng, cột).

```python
import pandas as pd

df = pd.DataFrame({
    'ten':   ['An', 'Binh', 'Chi', 'Dung'],
    'toan':  [8.5, 9.0, 7.5, 8.0],
    'van':   [7.0, 8.5, 9.0, 6.5],
    'anh':   [9.0, 7.5, 8.0, 9.5]
})

df.iloc[0]          # hàng đầu tiên
df.iloc[1, 2]       # hàng 2, cột 3 → điểm Văn của Bình = 8.5
df.iloc[0:2]        # 2 hàng đầu
df.iloc[:, 1:3]     # tất cả hàng, cột 2 và 3
df.iloc[[0, 2], :]  # hàng 1 và 3

# So sánh: loc dùng nhãn, iloc dùng số
df.loc[df['ten'] == 'An']   # loc → lọc theo điều kiện/nhãn
df.iloc[0]                  # iloc → lọc theo vị trí số
```

**Ví dụ thực tế:** Lấy 100 dòng dữ liệu đầu tiên và 3 cột đặc trưng để train model thử.

```python
X = df.iloc[:100, 1:4]  # 100 dòng, cột 2-4 làm features
y = df.iloc[:100, 0]    # cột đầu làm label
```

---

## Level 74 — reshape và ravel

Dùng để thay đổi hình dạng mảng NumPy — bắt buộc khi chuẩn bị dữ liệu cho ML.

```python
import numpy as np

a = np.array([1, 2, 3, 4, 5, 6])

# reshape: đổi shape, tổng phần tử không đổi
a.reshape(2, 3)     # [[1,2,3],[4,5,6]]
a.reshape(3, 2)     # [[1,2],[3,4],[5,6]]
a.reshape(-1, 1)    # [[1],[2],[3],[4],[5],[6]] → dùng cho sklearn

# ravel: làm phẳng về 1D
b = np.array([[1, 2, 3], [4, 5, 6]])
b.ravel()           # [1, 2, 3, 4, 5, 6]
```

**Ví dụ thực tế:** Chuẩn bị ảnh 28×28 pixel cho neural network.

```python
from PIL import Image
import numpy as np

img = Image.open('chu_so.png').resize((28, 28)).convert('L')
arr = np.array(img)      # shape (28, 28)
vector = arr.ravel()     # shape (784,) → input cho model
```

**Khi nào dùng:**
- `reshape(-1, 1)` → khi sklearn báo lỗi "1D array"
- `ravel()` → khi cần flatten ảnh/matrix về vector

---

## Level 75 — Vẽ Line Chart bằng Matplotlib

Line chart phù hợp để thể hiện xu hướng theo thời gian.

```python
import matplotlib.pyplot as plt

thang = ['T1', 'T2', 'T3', 'T4', 'T5', 'T6']
doanh_thu = [120, 135, 98, 160, 175, 190]

plt.figure(figsize=(10, 5))
plt.plot(thang, doanh_thu, marker='o', color='steelblue', linewidth=2)
plt.title('Doanh Thu 6 Tháng Đầu Năm')
plt.xlabel('Tháng')
plt.ylabel('Triệu VNĐ')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('doanh_thu.png')
plt.show()
```

**Ví dụ thực tế:** Biểu đồ theo dõi nhiệt độ mỗi ngày, giá cổ phiếu theo tuần.

---

## Level 76 — Vẽ Scatter Chart bằng Matplotlib

Scatter chart dùng để tìm **mối quan hệ** giữa 2 biến số.

```python
import matplotlib.pyplot as plt
import numpy as np

# Dữ liệu: diện tích nhà vs giá
dien_tich = [30, 45, 60, 75, 90, 120, 150]
gia        = [500, 700, 950, 1100, 1400, 1900, 2500]

plt.figure(figsize=(8, 5))
plt.scatter(dien_tich, gia, color='tomato', s=100, alpha=0.7)
plt.title('Diện Tích vs Giá Nhà')
plt.xlabel('Diện tích (m²)')
plt.ylabel('Giá (triệu VNĐ)')
plt.grid(True, alpha=0.3)
plt.show()
```

**Ví dụ thực tế:** Phát hiện ra "ô nhiễm dữ liệu" — điểm nằm xa cụm (outlier) trong tập dữ liệu.

---

## Level 77 — Vẽ Bar Chart bằng Matplotlib

Bar chart so sánh các nhóm rời rạc.

```python
import matplotlib.pyplot as plt

mon_hoc = ['Toán', 'Văn', 'Anh', 'Lý', 'Hóa']
diem_tb = [8.2, 7.5, 8.8, 7.9, 6.5]
mau = ['steelblue', 'salmon', 'mediumseagreen', 'mediumpurple', 'goldenrod']

plt.figure(figsize=(8, 5))
bars = plt.bar(mon_hoc, diem_tb, color=mau, edgecolor='white')

# Ghi số lên mỗi cột
for bar, diem in zip(bars, diem_tb):
    plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.1,
             str(diem), ha='center', fontweight='bold')

plt.title('Điểm Trung Bình Các Môn')
plt.ylabel('Điểm')
plt.ylim(0, 10)
plt.tight_layout()
plt.show()
```

**Ví dụ thực tế:** So sánh doanh thu giữa các chi nhánh cửa hàng trong quý.

---

## Level 78 — Linear Regression (Hồi quy tuyến tính)

Dự đoán **giá trị liên tục** (số) dựa trên mối quan hệ tuyến tính.

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

# Dữ liệu: số giờ học → điểm thi
gio_hoc = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]).reshape(-1, 1)
diem    = np.array([40, 50, 55, 60, 65, 70, 75, 82, 88, 95])

# Train model
X_train, X_test, y_train, y_test = train_test_split(gio_hoc, diem, test_size=0.2)
model = LinearRegression()
model.fit(X_train, y_train)

# Dự đoán
print(f'Học 7.5 giờ → dự đoán điểm: {model.predict([[7.5]])[0]:.1f}')
print(f'R² score: {model.score(X_test, y_test):.2f}')

# Vẽ đường hồi quy
plt.scatter(gio_hoc, diem, color='tomato')
plt.plot(gio_hoc, model.predict(gio_hoc), color='steelblue', linewidth=2)
plt.title('Số Giờ Học vs Điểm Thi')
plt.xlabel('Giờ học')
plt.ylabel('Điểm')
plt.show()
```

**Ví dụ thực tế:** Dự đoán giá điện dựa trên lượng tiêu thụ kWh trong tháng.

---

## Level 79 — Logistic Regression (Hồi quy logistic)

Phân loại **nhị phân** (đúng/sai, đậu/rớt, spam/không spam).

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

# Dữ liệu: [số giờ học, điểm giữa kỳ] → đậu(1) hay rớt(0)
X = np.array([
    [2, 5], [4, 6], [6, 7], [8, 8], [3, 4],
    [5, 7], [7, 8], [9, 9], [1, 3], [6, 6]
])
y = np.array([0, 0, 1, 1, 0, 1, 1, 1, 0, 1])

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)

model = LogisticRegression()
model.fit(X_train, y_train)

# Dự đoán: học 5h, giữa kỳ 6 điểm → đậu không?
pred = model.predict([[5, 6]])
prob = model.predict_proba([[5, 6]])
print(f'Kết quả: {"Đậu" if pred[0]==1 else "Rớt"}')
print(f'Xác suất đậu: {prob[0][1]:.0%}')
print(classification_report(y_test, model.predict(X_test)))
```

**Khác biệt vs Linear Regression:**
- Linear → dự đoán **số** (điểm, giá, nhiệt độ)
- Logistic → dự đoán **nhóm** (đậu/rớt, ung thư/bình thường, spam/không)

---

## Level 80A — Python Dictionary

Dictionary lưu dữ liệu dạng **key-value**, tra cứu nhanh O(1).

```python
# Tạo dictionary
sinh_vien = {
    'ten': 'Nguyễn An',
    'tuoi': 20,
    'diem': {'toan': 8.5, 'van': 7.0, 'anh': 9.0}
}

# Truy cập
print(sinh_vien['ten'])           # Nguyễn An
print(sinh_vien['diem']['toan'])  # 8.5

# Thêm / sửa / xóa
sinh_vien['truong'] = 'HCMUT'
sinh_vien['tuoi'] = 21
del sinh_vien['van']

# Duyệt
for key, value in sinh_vien.items():
    print(f'{key}: {value}')

# Dict comprehension
binh_phuong = {x: x**2 for x in range(1, 6)}
# {1:1, 2:4, 3:9, 4:16, 5:25}
```

**Ví dụ thực tế:** Lưu cấu hình app, đếm tần suất từ trong văn bản.

```python
# Đếm từ
van_ban = "python là ngôn ngữ python rất hay python"
dem_tu = {}
for tu in van_ban.split():
    dem_tu[tu] = dem_tu.get(tu, 0) + 1
# {'python': 3, 'là': 1, 'ngôn': 1, ...}
```

---

## Level 80B — Pandas DataFrame

DataFrame là "bảng tính thông minh" — cấu trúc dữ liệu trung tâm của data science.

```python
import pandas as pd

# Tạo DataFrame
df = pd.DataFrame({
    'ten':    ['An', 'Binh', 'Chi', 'Dung'],
    'tuoi':   [20, 21, 19, 22],
    'diem':   [8.5, 9.0, 7.5, 8.0],
    'lop':    ['A', 'B', 'A', 'B']
})

# Thao tác cơ bản
df['diem_xep_loai'] = df['diem'].apply(lambda x: 'Giỏi' if x >= 8.5 else 'Khá')
df.drop(columns=['tuoi'], inplace=True)
df.rename(columns={'ten': 'ho_ten'}, inplace=True)

# Gộp 2 DataFrame
df_diem_moi = pd.DataFrame({'ho_ten': ['An', 'Binh'], 'diem_moi': [9.0, 8.5]})
df_merged = pd.merge(df, df_diem_moi, on='ho_ten', how='left')

# Pivot table
pivot = df.pivot_table(values='diem', index='lop', aggfunc='mean')

# Xử lý giá trị thiếu
df.fillna(0, inplace=True)
df.dropna(inplace=True)
```

**Ví dụ thực tế:** Hợp nhất bảng đơn hàng + bảng khách hàng, tính tổng doanh thu theo vùng.

---

## Level 80C — K-Means Clustering

Phân nhóm dữ liệu **không cần nhãn** (unsupervised learning) — tìm các cụm tự nhiên.

```python
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# Dữ liệu khách hàng: [thu nhập (triệu), chi tiêu (triệu)]
X = np.array([
    [3, 2], [4, 3], [3, 4],          # nhóm thu nhập thấp
    [8, 7], [9, 8], [7, 9],          # nhóm thu nhập trung
    [15, 14], [16, 13], [14, 15]     # nhóm thu nhập cao
])

# Chuẩn hóa dữ liệu
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Tìm số cụm tối ưu bằng Elbow Method
inertias = []
for k in range(1, 8):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    km.fit(X_scaled)
    inertias.append(km.inertia_)

plt.plot(range(1, 8), inertias, marker='o')
plt.title('Elbow Method — Chọn số cụm K')
plt.xlabel('K')
plt.ylabel('Inertia')
plt.show()

# Train với K=3
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
labels = kmeans.fit_predict(X_scaled)

# Vẽ kết quả
colors = ['tomato', 'steelblue', 'mediumseagreen']
for i, (point, label) in enumerate(zip(X, labels)):
    plt.scatter(point[0], point[1], color=colors[label], s=100)
plt.title('Phân nhóm khách hàng')
plt.xlabel('Thu nhập (triệu)')
plt.ylabel('Chi tiêu (triệu)')
plt.show()

print('Nhóm của từng khách hàng:', labels)
```

**Ví dụ thực tế:**
- **Marketing:** Chia khách hàng thành VIP / thường / tiềm năng
- **Y tế:** Phân loại bệnh nhân theo nhóm triệu chứng
- **Thương mại điện tử:** Gợi ý sản phẩm theo nhóm hành vi mua hàng

---

## Tóm tắt nhanh

| Level | Chủ đề | Dùng để |
|-------|--------|---------|
| 71 | CSV files | Đọc/ghi dữ liệu dạng bảng |
| 72 | Pandas read_csv | Phân tích file CSV nhanh |
| 73 | iloc | Chọn dữ liệu theo vị trí |
| 74 | reshape/ravel | Chuẩn bị data cho ML |
| 75 | Line chart | Xu hướng theo thời gian |
| 76 | Scatter chart | Tương quan 2 biến |
| 77 | Bar chart | So sánh các nhóm |
| 78 | Linear Regression | Dự đoán số |
| 79 | Logistic Regression | Phân loại nhị phân |
| 80A | Dictionary | Lưu key-value, tra nhanh |
| 80B | DataFrame | Xử lý bảng dữ liệu |
| 80C | K-Means | Phân nhóm không nhãn |

## Thứ tự học đề nghị

```
CSV (71) → Pandas (72, 80B) → iloc (73) → reshape/ravel (74)
→ Matplotlib (75→77) → Linear (78) → Logistic (79) → K-Means (80C)
Dictionary (80A) có thể học bất cứ lúc nào
```
