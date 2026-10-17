# CĐ-08 · SQL cho phân tích dữ liệu — Cheatsheet

> 30 câu (A: Nhận biết 1–8 · B: Thông hiểu 9–18 · C: Vận dụng 19–27 · D: Vận dụng cao 28–30). Dữ liệu mẫu dùng xuyên suốt: `customers(id,name,city)` 5 khách (Ha có city NULL), `orders` 7 đơn 101–107 (106 amount NULL/pending, 103 cancelled).

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | Thứ tự logic: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY (→ LIMIT) | A02, A06, C06 |
| 2 | Điều kiện trên hàm gộp đặt ở `HAVING`, không ở `WHERE`; alias SELECT không dùng được trong WHERE | A06, C05 |
| 3 | NULL: so sánh `=`/`<>` với NULL cho UNKNOWN → dòng bị loại; dùng `IS NULL` | A01, B02, B10 |
| 4 | `COUNT(*)` đếm dòng; `COUNT(col)` bỏ NULL; SUM/AVG bỏ NULL | A05, B06, B05, D02 |
| 5 | Gộp trên tập rỗng (không GROUP BY): 1 dòng, COUNT=0, SUM=NULL | B05 |
| 6 | Các loại JOIN: LEFT giữ hết bảng trái, CROSS = tích $m\times n$, self-join, non-equi join | A03, A04, C02, C03 |
| 7 | Anti-join: `LEFT JOIN … WHERE b.pk IS NULL`; bẫy `NOT IN` gặp NULL | B04, B10 |
| 8 | Lọc bảng phải của LEFT JOIN trong WHERE biến nó thành INNER → đưa vào `ON` | B03 |
| 9 | JOIN 1-n nhân bản dòng (fan-out) làm SUM bị thổi phồng | C01, C09 |
| 10 | Window function không gộp dòng; ROW_NUMBER/RANK/DENSE_RANK; LAG; SUM lũy kế; frame mặc định RANGE gộp dòng hoà | A07, B08, B09, C07, D01, D03 |
| 11 | Không lọc window function trong WHERE → bọc CTE/subquery; QUALIFY không có trên PG/SQLite | C06 |
| 12 | Độ ưu tiên `AND` > `OR` | B01 |
| 13 | Chia số nguyên cắt phần lẻ → nhân `1.0`/`100.0`; `NULLIF` tránh chia 0 | C04, D03 |
| 14 | GROUP BY nghiêm ngặt (PG lỗi, SQLite cho chạy) | B07 |
| 15 | Correlated subquery (so với trung bình của chính nhóm) | D02 |
| 16 | EXPLAIN vs EXPLAIN ANALYZE; index composite theo quy tắc leftmost prefix | A08, C08 |

## 2. Thứ tự xử lý & lọc

```
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT (alias, window) → DISTINCT → ORDER BY → LIMIT
```
- `WHERE` lọc **dòng** trước gộp; `HAVING` lọc **nhóm** sau gộp → `HAVING SUM(amount) > 250`.
- Alias định nghĩa ở SELECT chỉ dùng được ở `ORDER BY` (không ở WHERE/HAVING chuẩn).
- Window function tính ở bước SELECT ⇒ không đặt trong WHERE/GROUP BY/HAVING.
- `AND` ưu tiên hơn `OR`: `a OR b AND c` ≡ `a OR (b AND c)`. Ví dụ B01 → `101,102,106,107`.

## 3. NULL — logic 3 giá trị

| Biểu thức | Kết quả |
|---|---|
| `NULL = NULL`, `x = NULL`, `x <> 'closed'` khi x NULL | UNKNOWN → WHERE loại dòng |
| `x IN (NULL)` | không bao giờ TRUE |
| `x NOT IN (…, NULL)` | không bao giờ TRUE → **0 dòng** (B10) |
| `NOT (x = 'closed')` khi x NULL | vẫn UNKNOWN |

- Lọc NULL: `IS NULL` / `IS NOT NULL`. Chuỗi rỗng `''` ≠ NULL.
- Đếm "chưa đóng kể cả NULL": `status <> 'closed' OR status IS NULL` (→ 4).
- Thay NULL: `COALESCE(x, 'mặc định')`; tránh chia 0: `NULLIF(mau, 0)`.
- An toàn thay `NOT IN`: `NOT EXISTS (SELECT 1 FROM b WHERE b.id = a.id)` hoặc anti-join.

## 4. Hàm gộp

| Biểu thức | Trên amount = 200,150,300,120,80,NULL,150 |
|---|---|
| `COUNT(*)` | 7 (đếm dòng) |
| `COUNT(amount)` | 6 (bỏ NULL) ✔ |
| `COUNT(DISTINCT amount)` | 5 |
| `COUNT(amount IS NOT NULL)` | 7 (biểu thức boolean không bao giờ NULL) |

- SUM/AVG bỏ NULL: AVG của (200,150,NULL) = 175, không chia 3.
- Tập rỗng không GROUP BY → **1 dòng**: `COUNT(*)=0`, `SUM=NULL` (dùng `COALESCE(SUM(x),0)`). Có GROUP BY + rỗng → 0 dòng.
- Đếm có điều kiện: `SUM(CASE WHEN cond THEN 1 ELSE 0 END)` hoặc `COUNT(CASE WHEN cond THEN 1 END)`.
- B06 kết quả: `1|3|2|350`, `2|2|2|380`, `3|2|2|270`.
- C05 (paid ≥2 đơn, theo tổng giảm): `1|2|350`, `3|2|270`.
- Cột không gộp trong SELECT phải có trong GROUP BY: PostgreSQL báo lỗi; SQLite vẫn chạy (lấy giá trị tuỳ ý trong nhóm).

## 5. JOIN

| JOIN | Kết quả |
|---|---|
| `INNER` | chỉ cặp khớp |
| `LEFT` | giữ mọi dòng trái, thiếu khớp → cột phải NULL |
| `RIGHT` | ngược lại (giữ bảng phải) |
| `CROSS` | $m \times n$ dòng (3 size × 4 màu = 12) |

- **Anti-join** (khách chưa có đơn): `LEFT JOIN orders o ON … WHERE o.id IS NULL` (dùng cột PK không-NULL) → `Dung`, `Ha`.
- **Lọc bảng phải**: `WHERE o.status='paid'` sau LEFT JOIN loại các dòng NULL ⇒ thành INNER (3 dòng). Sửa: `LEFT JOIN orders o ON o.customer_id=c.id AND o.status='paid'`, rồi `COUNT(o.id)` → khách không có đơn = 0.
- **Self-join** (nhân viên–quản lý): `FROM employees e LEFT JOIN employees m ON e.manager_id = m.id` + `COALESCE(m.name,'-')` → `Lan|-`, `Minh|Lan`, `Nga|Lan`, `Oanh|Minh`.
- **Non-equi join**: `ON o.amount BETWEEN t.lo AND t.hi` (BETWEEN gồm 2 đầu; giá trị ở biên 200 khớp cả Bac và Vang; NULL không khớp tier nào) → Dong 1, Bac 4, Vang 2.
- **Fan-out**: orders 1–n payments ⇒ amount đơn 101 lặp 2 lần: `SUM(o.amount)=550` (sai), `SUM(p.paid)=350`. Sửa: gộp payments trong subquery trước khi JOIN.

## 6. Window function

```sql
fn(...) OVER (PARTITION BY nhom ORDER BY cot [ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW])
```
- Không gộp dòng: 7 dòng vào → 7 dòng ra (A07).
- Xếp hạng (90, 85, 85, 80):

| | Lan | Minh | Nga | Oanh |
|---|---|---|---|---|
| `ROW_NUMBER` | 1 | 2 | 3 | 4 |
| `RANK` (nhảy hạng) | 1 | 2 | 2 | 4 |
| `DENSE_RANK` (liền) | 1 | 2 | 2 | 3 |

- `LAG(col, k, default)` / `LEAD`: `revenue - LAG(revenue,1,0) OVER (ORDER BY day)` với 100,130,90,90 → `100, 30, -40, 0`.
- Lũy kế theo nhóm: `SUM(revenue) OVER (PARTITION BY store ORDER BY day)` → `A|100, A|230, A|320, B|200, B|350`.
- **Frame mặc định** khi có ORDER BY = `RANGE … CURRENT ROW` ⇒ các dòng cùng giá trị ORDER BY (peers) nhận cùng lũy kế: 100, **220, 220**, 250. Sửa: thêm khoá phân biệt `ORDER BY day, id` (hoặc `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`). `PARTITION BY day` là sai (reset mỗi ngày).
- **Top-1 mỗi nhóm** (PG + SQLite):
```sql
WITH r AS (
  SELECT id, customer_id, amount,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC) AS rn
  FROM orders WHERE amount IS NOT NULL)
SELECT id, customer_id, amount FROM r WHERE rn = 1;
```
(PG xếp NULL **đầu** khi DESC → phải lọc NULL hoặc `NULLS LAST`.)

## 7. Subquery tương quan
```sql
SELECT o.id, o.amount FROM orders o
WHERE o.amount > (SELECT AVG(o2.amount) FROM orders o2 WHERE o2.customer_id = o.customer_id);
```
TB khách 1 = 175 (NULL bị bỏ), khách 2 = 190, khách 3 = 135 → `101|200`, `103|300`, `107|150`. Dòng amount NULL: so sánh UNKNOWN → loại.

## 8. Kiểu số & chia
- `int / int` → số nguyên (PG và SQLite): `1/7 = 0`. Sửa: `SUM(...) * 1.0 / COUNT(*)` hoặc `CAST(... AS REAL/NUMERIC)`.
- Phần trăm tăng trưởng: $\text{pct}=\dfrac{dt - dt_{trước}}{dt_{trước}}\times 100$ → `ROUND((dt - truoc) * 100.0 / NULLIF(truoc, 0), 1)`.

## 9. Bài code mẫu

**C09 — báo cáo theo thành phố** (gộp đơn paid theo khách *trước* rồi LEFT JOIN để tránh fan-out & giữ mọi thành phố):
```sql
SELECT COALESCE(c.city, 'Chua ro')            AS thanh_pho,
       COUNT(*)                               AS so_khach,
       COUNT(p.customer_id)                   AS so_khach_mua,
       COALESCE(SUM(p.tong), 0)               AS doanh_thu
FROM customers c
LEFT JOIN (SELECT customer_id, SUM(amount) AS tong
           FROM orders WHERE status = 'paid'
           GROUP BY customer_id) p ON p.customer_id = c.id
GROUP BY COALESCE(c.city, 'Chua ro')
ORDER BY doanh_thu DESC, thanh_pho ASC;
-- HN 2 2 620 | HCM 1 1 80 | Chua ro 1 0 0 | DN 1 0 0
```
(Cách khác: JOIN trực tiếp với `ON … AND o.status='paid'`, rồi `COUNT(DISTINCT c.id)`, `COUNT(DISTINCT o.customer_id)`.)

**D03 — báo cáo tháng với CTE + window, không self-join**:
```sql
WITH m AS (
  SELECT SUBSTR(order_date, 1, 7) AS thang, SUM(amount) AS doanh_thu
  FROM orders WHERE status = 'paid'
  GROUP BY SUBSTR(order_date, 1, 7)
), w AS (
  SELECT thang, doanh_thu,
         LAG(doanh_thu) OVER (ORDER BY thang)  AS truoc,
         SUM(doanh_thu) OVER (ORDER BY thang)  AS luy_ke
  FROM m)
SELECT thang, doanh_thu,
       ROUND((doanh_thu - truoc) * 100.0 / NULLIF(truoc, 0), 1) AS tang_truong_pct,
       luy_ke
FROM w ORDER BY thang;
-- 2026-01 350 NULL 350 | 2026-02 200 -42.9 550 | 2026-03 150 -25.0 700
```

## 10. EXPLAIN & index
- `EXPLAIN`: chỉ kế hoạch + chi phí **ước tính**, không chạy. `EXPLAIN ANALYZE`: **chạy thật** (DELETE/UPDATE sẽ thay đổi dữ liệu → bọc `BEGIN; … ROLLBACK;`), in thời gian & số dòng thực tế. Khác lệnh `ANALYZE` (cập nhật thống kê).
- Index B-tree composite `(customer_id, order_date)` dùng theo **leftmost prefix**: tốt cho `customer_id = …`, `customer_id IN (…)`, `customer_id = … AND order_date >= …`, `customer_id = … ORDER BY order_date DESC LIMIT k`. Chỉ lọc `order_date` (cột thứ 2) + chọn ~1/3 bảng → **Seq Scan**.

## 11. Bẫy thường gặp

| Phương án nhiễu | Vì sao sai |
|---|---|
| `WHERE SUM(amount) > 250` / `WHERE tong > 250` (alias) | WHERE chạy trước GROUP BY & SELECT |
| `SELECT` đứng đầu thứ tự xử lý | Đó là thứ tự viết, không phải logic |
| `city = NULL`, `city IN (NULL)`, `city = ''` | NULL chỉ bắt được bằng `IS NULL` |
| `COUNT(amount IS NOT NULL)` | Đếm boolean (không NULL) → bằng COUNT(*) |
| CROSS JOIN = $m+n$ | Là $m\times n$ |
| Window function trả 1 dòng/nhóm (3) | Window giữ nguyên số dòng |
| `NOT IN` với subquery chứa NULL "trả 4" | Trả 0; dùng NOT EXISTS |
| `status <> 'closed'` / `NOT (status='closed')` để đếm cả NULL | Bỏ sót dòng NULL |
| LEFT JOIN + WHERE trên bảng phải "vẫn 5 dòng" | Thành INNER; chuyển điều kiện vào ON. Đổi COUNT(*) hay RIGHT JOIN không cứu được |
| PG "lấy name nhỏ nhất" khi thiếu GROUP BY | PG báo lỗi; chỉ SQLite cho chạy |
| Gộp tập rỗng → không dòng / SUM=0 | 1 dòng, COUNT=0, SUM=NULL |
| `QUALIFY rn = 1` | Không có ở PG/SQLite |
| `WHERE ROW_NUMBER() OVER(...) = 1` | Window không được trong WHERE |
| `HAVING amount = MAX(amount)` | amount không phải cột nhóm/gộp |
| Tier BETWEEN tạo dòng NULL cho đơn NULL | LEFT JOIN từ tiers; đơn NULL không khớp → bị bỏ |
| `1/7 ≈ 0.1429` | Chia số nguyên → 0 |
| JOIN payments "SUM(amount)=350" | Fan-out → 550 |
| `PARTITION BY day` để lũy kế riêng từng giao dịch | Reset theo ngày; đúng là thêm `id` vào ORDER BY |
| Lũy kế "100,150,220,250" | Frame RANGE gộp peers → 100,220,220,250 |
| AVG chia cả dòng NULL | AVG bỏ NULL |
| `EXPLAIN ANALYZE` an toàn/chỉ đọc | Chạy thật câu lệnh |

## 12. Tra nhanh

| Nếu gặp… | Dùng… |
|---|---|
| Lọc theo tổng/số đếm nhóm | `GROUP BY … HAVING agg(...) …` |
| Lọc giá trị thiếu | `IS NULL` / `IS NOT NULL` |
| Đếm giá trị có ghi | `COUNT(col)`; đếm dòng `COUNT(*)` |
| Đếm/tổng có điều kiện | `SUM(CASE WHEN … THEN 1 ELSE 0 END)` |
| Giữ hết bảng chính | `LEFT JOIN`, điều kiện bảng phụ đặt trong `ON` |
| Tìm bản ghi không có khớp | `LEFT JOIN … WHERE b.pk IS NULL` hoặc `NOT EXISTS` |
| Sinh mọi tổ hợp | `CROSS JOIN` |
| Quan hệ cha–con cùng bảng | Self `LEFT JOIN` + `COALESCE` |
| Ghép theo khoảng giá trị | `ON x BETWEEN lo AND hi` |
| JOIN 1–n rồi SUM | Gộp bảng n trong subquery/CTE trước |
| Top-1 mỗi nhóm | CTE + `ROW_NUMBER() OVER (PARTITION BY … ORDER BY … DESC)` → `WHERE rn=1` |
| Xếp hạng có hoà | `RANK` (nhảy) / `DENSE_RANK` (liền) |
| So với kỳ trước | `LAG(col, 1, default) OVER (ORDER BY t)` |
| Cộng dồn | `SUM(col) OVER (PARTITION BY g ORDER BY t, id)` hoặc frame `ROWS` |
| So với TB của chính nhóm | Correlated subquery hoặc `AVG(...) OVER (PARTITION BY g)` |
| Tỉ lệ / phần trăm | Nhân `1.0`/`100.0` trước khi chia; `NULLIF(mau,0)` |
| NULL → 0 / nhãn | `COALESCE(x, 0)` / `COALESCE(city,'Chua ro')` |
| Tháng từ chuỗi ngày | `SUBSTR(order_date, 1, 7)` |
| Xem kế hoạch / đo thực tế | `EXPLAIN` / `EXPLAIN ANALYZE` (bọc transaction nếu ghi) |
| Index composite (a,b) | Điều kiện phải có cột `a` mới dùng index hiệu quả |

## Đáp án nhanh
A01 D · A02 A · A03 D · A04 C · A05 A · A06 D · A07 B · A08 A · B02 A · B03 C · B05 D · B07 B · B08 C · B10 D · C01 D · C03 A · C04 C · C06 C · C08 D · D01 C
Điền: B01 `101/102/106/107` · B04 `Dung/Ha` · B06 `1|3|2|350, 2|2|2|380, 3|2|2|270` · B09 `100/30/-40/0` · C02 `Lan|-, Minh|Lan, Nga|Lan, Oanh|Minh` · C05 `1|2|350, 3|2|270` · C07 `A|100, A|230, A|320, B|200, B|350` · D02 `101|200, 103|300, 107|150`
