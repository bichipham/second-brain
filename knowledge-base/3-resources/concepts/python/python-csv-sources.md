---
title: File CSV — Nguồn gốc và cách thu thập dữ liệu
type: concept
tags: [python, csv, data-collection, web-scraping, api, database, iot]
updated: 2026-05-27
sources: [conversation-2026-05-27]
---

# File CSV — Từ đâu ra và thu thập như thế nào?

CSV (Comma-Separated Values) là file text thuần — mỗi dòng là một bản ghi, các cột ngăn cách bằng dấu phẩy. Dữ liệu có thể đến từ nhiều nguồn khác nhau.

---

## 1. Xuất từ phần mềm có sẵn

Hầu hết mọi phần mềm đều có nút "Export CSV":

- **Excel / Google Sheets** → File → Save As → CSV
- **Phần mềm kế toán** (MISA, Fast...) → xuất báo cáo → CSV
- **Hệ thống quản lý** (CRM, ERP) → xuất danh sách khách hàng, đơn hàng
- **Google Analytics, Facebook Ads** → xuất báo cáo quảng cáo

---

## 2. Thu thập từ Internet (Web Scraping)

Code tự động vào website, đọc dữ liệu rồi lưu vào CSV:

```python
import requests
from bs4 import BeautifulSoup
import csv

url = 'https://giavang.org'
res = requests.get(url)
soup = BeautifulSoup(res.text, 'html.parser')

rows = soup.select('table tr')

with open('gia_vang.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['Loại', 'Mua vào', 'Bán ra'])
    for row in rows[1:]:
        cols = [td.text.strip() for td in row.select('td')]
        if cols:
            writer.writerow(cols)
```

---

## 3. Gọi API → lưu CSV

Nhiều dịch vụ cung cấp API trả về JSON, convert sang CSV:

```python
import requests
import pandas as pd

res = requests.get('https://api.exchangerate-api.com/v4/latest/USD')
data = res.json()

df = pd.DataFrame(data['rates'].items(), columns=['Tiền tệ', 'Tỷ giá'])
df.to_csv('ty_gia.csv', index=False)
```

---

## 4. Từ cơ sở dữ liệu (Database)

Query SQL rồi xuất ra CSV:

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('cua_hang.db')
df = pd.read_sql_query("SELECT * FROM don_hang WHERE thang = 5", conn)
df.to_csv('don_hang_thang5.csv', index=False)
conn.close()
```

MySQL, PostgreSQL, SQL Server đều làm được tương tự.

---

## 5. Từ thiết bị / cảm biến (IoT)

Máy móc, cảm biến ghi dữ liệu theo thời gian thực:

```python
import csv, time, random

with open('nhiet_do.csv', 'a', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['thoi_gian', 'nhiet_do', 'do_am'])
    for _ in range(10):
        writer.writerow([
            time.strftime('%Y-%m-%d %H:%M:%S'),
            round(random.uniform(25, 35), 1),
            round(random.uniform(60, 90), 1)
        ])
        time.sleep(1)
```

**Thực tế:** nhà máy ghi nhiệt độ lò, trạm thời tiết ghi lượng mưa, xe tự lái ghi dữ liệu cảm biến.

---

## 6. Người dùng tự nhập (Form / Survey)

Google Forms, Typeform, SurveyMonkey → tự động lưu mỗi phản hồi thành 1 dòng CSV.

---

## Vòng đời của một file CSV

```
Nguồn dữ liệu
    ↓
Thu thập (scraping / API / sensor / form / DB)
    ↓
Lưu thành file .csv
    ↓
Đọc bằng Pandas → Làm sạch → Phân tích → Train model
    ↓
Xuất kết quả ra CSV mới / Dashboard / Báo cáo
```

---

## Xem thêm

- [./python-advance-ai.md](./python-advance-ai.md) — Level 71–80C: CSV, Pandas, ML
- [./python-advance-file.md](./python-advance-file.md) — Cơ chế ghi file Python
