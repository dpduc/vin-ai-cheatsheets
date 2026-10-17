# CĐ-07 · NumPy và độ phức tạp thuật toán — Cheatsheet

> 30 câu (A: Nhận biết, B: Thông hiểu, C: Vận dụng, D: Vận dụng cao). Mã câu dạng T07-xxx.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | `*` là nhân từng phần tử; `@` là tích vô hướng/ma trận; `np.outer` ra ma trận | A03 |
| 2 | `ndim` = số trục, `size` = tích các chiều, `nbytes = size × itemsize` | A01, B04, C04 |
| 3 | Mask boolean: `(x > t).sum()` đếm số phần tử thoả điều kiện | A04 |
| 4 | `axis=0` gộp theo cột (đi dọc hàng), `axis=1` gộp theo hàng; `argmax` trả chỉ số | A02, C01 |
| 5 | `keepdims=True` giữ shape `(m,1)` để broadcast chia theo hàng | B03 |
| 6 | Quy tắc broadcasting: so từ phải sang, mỗi cặp bằng nhau hoặc có 1 | B02, C04 |
| 7 | Slice = view (sửa lan về gốc); fancy index = copy | B01 |
| 8 | `np.vectorize` chỉ là vòng lặp Python bọc lại, không nhanh hơn | B05 |
| 9 | `transpose` đổi trục giữ nội dung; `reshape` chỉ đổi cách đọc → xáo dữ liệu | C02 |
| 10 | Prefix sum `cumsum` → tổng cửa sổ trượt $O(n)$ không vòng lặp | C03 |
| 11 | Top-k: `np.argpartition` $O(n)$ + sort $k$ phần tử | D03 |
| 12 | Độ phức tạp thao tác list/dict/set/deque của CPython | A05, B07, C06, B10 |
| 13 | Big-O: giữ số hạng tăng nhanh nhất, bỏ hằng số | A07 |
| 14 | Đếm vòng lặp: tam giác $n(n+1)/2$, harmonic $n\log n$, nhị phân $\log_2 n$ | A06, C05, A08 |
| 15 | Đệ quy: 2 nhánh → $O(2^n)$; slicing `a[1:]` trong đệ quy → $O(n^2)$ | B06, C07 |
| 16 | `heapq`: heap min giữ top-k; `nlargest` $O(n\log k)$ | B08, B09 |
| 17 | Khử trùng giữ thứ tự: `dict.fromkeys` | B10 |
| 18 | Two-sum: hai con trỏ trên mảng đã sắp / hash set | C08, C09 |
| 19 | Sort ổn định (stable) → sort nhiều khoá: khoá phụ trước, khoá chính sau | D02 |
| 20 | Mảng động nhân đôi: tổng số copy $< 2n$ → append khấu hao $O(1)$ | D01 |

## 2. NumPy cốt lõi

### Thuộc tính & bộ nhớ
- `E.shape == (32, 768)` → `E.ndim = 2`, `E.size = 24576` (không phải 800 = 32+768).
- `nbytes = size × itemsize`; `float32` = 4 byte, `float64` = 8 byte.
  - $5\times10^5 \times 768 \times 4 = 1.536\times10^9$ B → **1.536 GB** (1 GB = $10^9$).
  - `diff = Q[:,None,:] - K[None,:,:]` shape `(2000, 5000, 128)` → $2000·5000·128·4 = 5.12\times10^9$ → **5.12 GB** (lý do nên dùng $\|q\|^2 + \|k\|^2 - 2q·k$ bằng `@`).

### Phép toán vector hoá
```python
c = a * b            # c[i] = a[i]*b[i]   (element-wise)
a @ b                # 1D: vô hướng (tổng tích); 2D: nhân ma trận
np.outer(a, b)       # ma trận (n, n)
np.prod([a, b])      # tích TẤT CẢ phần tử -> 1 số
(scores > 0.8).sum() # đếm True; tương đương np.count_nonzero(scores > 0.8)
```

### Axis & reduction
```python
a = np.array([[4,9],[7,2],[5,6]])   # (3,2)
a.max(axis=0)     # theo cột -> [7, 9]       shape (2,)
a.argmax(axis=1)  # theo hàng -> [1, 0, 1]   shape (3,)
S = Q @ D.T       # (8,384)@(384,1000) -> (8,1000)
S.argmax(axis=1)  # chỉ số đoạn văn tốt nhất mỗi câu hỏi, shape (8,)
```
Mẹo: `axis=k` là trục **bị xoá** khỏi shape.

### keepdims & chuẩn hoá theo hàng
```python
x = np.array([[1.,3.],[2.,6.],[0.,5.]])
m = x.sum(axis=1, keepdims=True)   # shape (3, 1)
p = x / m                          # p[:,0] = [0.25, 0.25, 0.0]
```

### Broadcasting — quy tắc
Căn phải, từng cặp chiều phải **bằng nhau hoặc một bên = 1** (thiếu chiều coi là 1).
| sx | sy | Kết quả |
|---|---|---|
| (2,3,4) | (3,1) | OK → (2,3,4) |
| (4,1,3) | (5,1) | OK → (4,5,3) |
| (32,1) | (1,768) | OK → (32,768) |
| (32,768) | (32,) | **ValueError** (768 ≠ 32) → cần `y[:, None]` |

### View vs copy
```python
a = np.arange(5)
b = a[1:4]; b[0] = 100     # slice = VIEW -> a đổi
c = a[[0,2]]; c[0] = -1    # fancy index = COPY -> a không đổi
a.tolist()  # [0, 100, 2, 3, 4]
```

### Đổi trục ảnh NHWC → NCHW
```python
X.transpose(0, 3, 1, 2)   # (16,224,224,3) -> (16,3,224,224) ĐÚNG
```
- `reshape(16,3,224,224)`: đúng shape nhưng xáo pixel.
- `swapaxes(X,1,3)` → (N,C,W,H): ảnh bị chuyển vị (xoay/lật).
- `transpose(0,2,3,1)` → sai thứ tự.

### Prefix sum — cửa sổ trượt
```python
c = np.concatenate(([0], np.cumsum(req)))  # c[i] = tổng i phần tử đầu
w = c[k:] - c[:-k]                          # tổng mỗi cửa sổ độ dài k
# req=[4,2,6,1,3,7], k=3 -> [12, 9, 10, 11], argmax = 0
```

### Top-k cosine (D03 — mẫu)
```python
def topk_cosine(Q, D, k):
    k = min(k, D.shape[0])
    qn = np.linalg.norm(Q, axis=1, keepdims=True)
    dn = np.linalg.norm(D, axis=1, keepdims=True)
    Qn = np.divide(Q, qn, out=np.zeros_like(Q, dtype=float), where=qn > 0)  # tránh nan
    Dn = np.divide(D, dn, out=np.zeros_like(D, dtype=float), where=dn > 0)
    S = Qn @ Dn.T                                    # (m, n)
    idx = np.argpartition(-S, k - 1, axis=1)[:, :k]  # k lớn nhất, chưa sắp, O(mn)
    part = np.take_along_axis(S, idx, axis=1)
    order = np.argsort(-part, axis=1)                # chỉ sắp k phần tử
    return np.take_along_axis(idx, order, axis=1)
# Time: O(mnd + mn + m k log k); Memory: O(md + nd + mn)
```
Lưu ý: `argpartition(..., k-1)` (kth là chỉ số 0-based); khi `k == n` vẫn hợp lệ.

### `np.vectorize`
Chỉ tiện (broadcasting, nhận mảng), **vẫn gọi f từng phần tử bằng Python** → không đổi Big-O, không biên dịch C, không song song. Muốn nhanh: viết lại bằng ufunc/biểu thức mảng.

## 3. Độ phức tạp thuật toán

### Big-O rút gọn
$T(n) = 4n\log_2 n + 0.5\,n^{1.5} + 200 \Rightarrow O(n^{1.5})$ vì $n^{1.5}$ tăng nhanh hơn $n\log n$.
Thứ tự: $1 < \log n < \sqrt n < n < n\log n < n^{1.5} < n^2 < 2^n$.

### Đếm vòng lặp
| Mẫu | Số lần | Big-O |
|---|---|---|
| `for i in range(n): for j in range(i, n)` | $n(n+1)/2$ (n=12 → **78**) | $O(n^2)$ |
| `for i in 1..n: for j in range(0, n, i)` | $\sum n/i \approx n H_n$ | $O(n\log n)$ |
| Binary search | $\lceil\log_2 n\rceil$ ($10^6$ → **~20**) | $O(\log n)$ |
| `f(n) = f(n-1) + f(n-1)` | $2^{n+1}-1$ lời gọi | $O(2^n)$ |
| `tong(a) = a[0] + tong(a[1:])` | slice copy $n-1, n-2, …$ | $O(n^2)$ |

### Bảng TimeComplexity CPython (trung bình)
| Thao tác | Chi phí |
|---|---|
| `lst.append(x)`, `lst.pop()`, `lst[i]` | $O(1)$ (append khấu hao) |
| `lst.pop(0)`, `lst.insert(0,x)`, `x in lst`, `lst[a:b]` | $O(n)$ / $O(b-a)$ |
| `d[k]=v`, `k in d`, `x in s`, `s.add` | $O(1)$ |
| `x in d.values()` | $O(n)$ — duyệt tuyến tính! |
| `deque.appendleft / popleft` | $O(1)$ |
| `heappush / heappop` | $O(\log n)$ |
| `heapq.nlargest(k, xs)` | $O(n\log k)$ |
| `sorted(xs)` | $O(n\log n)$, **stable** |

### Mảng động nhân đôi (D01)
Copy xảy ra khi đầy ở dung lượng $1,2,4,\dots,64$ → $1+2+\dots+64 = 127$ lần copy cho 100 append. Tổng quát $< 2n$ → append khấu hao $O(1)$.

### heapq — giữ top-k
```python
h = []
for x in [5,1,8,3,9,2]:
    heapq.heappush(h, x)
    if len(h) > 3: heapq.heappop(h)   # bỏ phần tử NHỎ nhất
print(sorted(h), h[0])   # [5, 8, 9] 5   (h[0] = min của top-3)
```

### Khử trùng giữ thứ tự
`list(dict.fromkeys(ids))` — $O(n)$, giữ thứ tự xuất hiện đầu (dict giữ thứ tự chèn từ Python 3.7).

### Two-sum
**Hai con trỏ** (mảng đã sắp): `s < k → l += 1`, `s > k → r -= 1`. Ví dụ `[1,3,4,6,8,11], k=10` → `((4, 6), 5)` (bước: 12, 9, 11, 9, 10).

**C09 — mẫu dùng Counter**, $O(n)$ time (+ $O(u\log u)$ sort các giá trị phân biệt), $O(n)$ memory:
```python
from collections import Counter
def cap_tong_k(gia, k):
    """Time O(n + u log u), Memory O(u); u = số giá trị phân biệt."""
    cnt = Counter(gia)
    res = []
    for a in sorted(cnt):
        b = k - a
        if a < b and b in cnt: res.append((a, b))
        elif a == b and cnt[a] >= 2: res.append((a, a))   # cặp (5,5) cần 2 vị trí
    return res
```

### Sort nhiều khoá bằng stable sort (D02)
Sort **khoá phụ trước**, **khoá chính sau**: `sort(key=price, reverse=True)` rồi `sort(key=cat)` → `['ao','giay','mu','tui','but','vi']`. Hoặc 1 lần: `sort(key=lambda r: (r[1], -r[2]))`.

## 4. Bẫy thường gặp

| Phương án nhiễu | Vì sao sai |
|---|---|
| `a @ b` thay cho vòng lặp `c[i]=a[i]*b[i]` | `@` cộng dồn ra 1 số |
| `E.ndim = 32` / `size = 800` | ndim là số trục; size là tích, không phải tổng |
| `np.count_nonzero(scores) > 0.8` | So sánh *số đếm* với 0.8 → bool |
| `len(scores > 0.8)` | Trả n, không đếm True |
| `np.sum(scores[scores > 0.8])` | Tổng *giá trị*, không phải số lượng |
| `S.argmax()` không axis | Chỉ số phẳng trên toàn ma trận, 1 số |
| `S.max(axis=1)` | Trả điểm cosine, không phải chỉ số |
| `argmax(axis=0)` | Theo từng đoạn văn, shape (1000,) |
| Nghĩ fancy index là view | Fancy index luôn copy |
| `reshape` để đổi NHWC→NCHW | Xáo pixel; phải `transpose` |
| `np.vectorize` giảm Big-O / chạy C / song song | Chỉ là vòng lặp Python |
| `x in d.values()` là $O(1)$ | values() là view, duyệt $O(n)$ → dựng `set` một lần, tổng $O(n+m)$ |
| `sorted(values)` để tra | Sort $O(n\log n)$ mỗi lần, `in` vẫn tuyến tính |
| `list(set(ids))` / `sorted(set(ids))` | Mất thứ tự xuất hiện |
| List comp `x not in ids[:i]` | $O(n^2)$ |
| `insert(0, r)` "luôn O(1)" | Dịch toàn bộ → $O(n)$ mỗi lần, tổng $O(n^2)$; `[r] + recent` cũng $O(n^2)$; heappush không đặt phần tử mới lên đầu |
| `nlargest` là $O(k\log n)$ hay $O(n\log n)$ | Duyệt n phần tử, heap kích thước k → $O(n\log k)$ |
| `tong(a[1:])` là $O(n)$ | Slicing copy → $O(n^2)$ |
| Vòng `range(0, n, i)` là $O(n^2)$ | Harmonic → $O(n\log n)$ |
| $O(n^{1.5}\log n)$ hay $O(n\log n)$ cho $4n\log n + 0.5n^{1.5}$ | Chỉ giữ số hạng trội, không nhân |
| Two-pointer: đếm sai số bước | `buoc += 1` trước khi so sánh, kể cả bước tìm thấy |
| Sort khoá chính trước rồi khoá phụ | Lần sort sau phá thứ tự khoá chính |
| Broadcasting (32,768)+(32,) | Căn phải: 768 vs 32 → lỗi |
| Two-sum `(5,5)` khi chỉ có một số 5 | Cần 2 vị trí khác nhau → kiểm tra count ≥ 2 |
| Cosine với vector 0 | Chia 0 → nan; dùng `np.divide(..., where=norm>0)` |

## 5. Tra nhanh

| Nếu gặp... | Dùng... |
|---|---|
| Nhân từng phần tử | `a * b` |
| Cosine nhiều query × nhiều doc | chuẩn hoá L2 rồi `Q @ D.T` → (m, n) |
| Chỉ số lớn nhất mỗi hàng | `.argmax(axis=1)` |
| Chia mỗi hàng cho tổng hàng | `x / x.sum(axis=1, keepdims=True)` |
| Đếm phần tử thoả điều kiện | `(mask).sum()` / `np.count_nonzero(mask)` |
| Cộng mảng (m,) vào (m,n) theo hàng | `y[:, None]` |
| Ma trận khoảng cách lớn | $\|q\|^2+\|k\|^2-2QK^T$, tránh tensor 3D |
| Ước lượng RAM | `size × itemsize` (float32 = 4B) |
| NHWC → NCHW | `transpose(0,3,1,2)` |
| Tổng cửa sổ trượt | `cumsum` + `c[k:] - c[:-k]` |
| Top-k trong NumPy | `argpartition` + `argsort` k phần tử + `take_along_axis` |
| Top-k trong list Python | `heapq.nlargest(k, xs)` $O(n\log k)$ |
| Thêm/bỏ ở đầu dãy | `collections.deque` (`appendleft`, `popleft`) |
| Tra thành viên nhiều lần | `set` / `dict` keys $O(1)$ |
| Khử trùng giữ thứ tự | `list(dict.fromkeys(xs))` |
| Two-sum mảng đã sắp | Hai con trỏ $O(n)$ |
| Two-sum chưa sắp | `Counter`/hash set $O(n)$ |
| Sort đa khoá, ngược chiều | Stable sort: khoá phụ trước → khoá chính sau; hoặc key tuple `(a, -b)` |
| Tìm trên mảng đã sắp | Binary search / `bisect` $O(\log n)$ |

## 6. Đáp án nhanh
A03 B · A05 A · A06 78 · A07 C $O(n^{1.5})$ · A02 `[7, 9]` / `[1, 0, 1]` · A08 B (~20) · A01 A · A04 B · B08 `[5, 8, 9] 5` · B06 $O(2^n)$ · B07 D · B01 A · B05 D · B10 B · B03 `(3, 1) [0.25, 0.25, 0.0]` · B04 1.536 · B09 C $O(n\log k)$ · B02 D · C06 A · C09 (code) · C01 A · C08 B · C02 B · C07 A $O(n^2)$ · C04 5.12 · C03 `[12, 9, 10, 11] 0` · C05 B $O(n\log n)$ · D02 D · D03 (code) · D01 127
