# CĐ-05 · Lập trình Python — Cheatsheet

> 30 câu (A: nhận biết 8 · B: thông hiểu 10 · C: vận dụng 9 · D: vận dụng cao 3). Bối cảnh: log truy cập/API, microservice, retry, hàng đợi tác vụ.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | Kiểu immutable: `int float str bool tuple frozenset bytes`; mutable: `list dict set` | A04 |
| 2 | Khóa dict phải hashable → tuple được, list/set/dict không | A01 |
| 3 | f-string format: `{n:,}` phân cách nghìn, `{r:.1%}` nhân 100 + làm tròn | A03 |
| 4 | `//` floor division (làm tròn xuống, `-7//2 = -4`), `/` luôn ra float | A02 |
| 5 | dict giữ thứ tự chèn; gán lại khóa cũ không đổi vị trí; thiếu khóa → `KeyError` | A08, A06 |
| 6 | `(... for ...)` là generator; `[]` list, `{}` set, `tuple(...)` dựng toàn bộ | A05 |
| 7 | Generator chỉ duyệt **một lần**; lazy — chỉ chạy khi `next()` | B07, C06 |
| 8 | `dict.copy()` là **shallow copy** — list lồng bên trong vẫn dùng chung | B04 |
| 9 | Gán slice `buf[1:3] = [...]` thay đoạn, độ dài có thể đổi | B01 |
| 10 | Dict comprehension: khóa trùng ghi đè, giữ vị trí lần đầu | B06 |
| 11 | `d1 \| d2` (3.9+) tạo dict mới, bên phải thắng, không sửa `d1` | B03 |
| 12 | `re.findall` có nhiều group → list tuple **chuỗi**; `match` chỉ ở đầu chuỗi | B10, C07 |
| 13 | Starred unpacking `first, *mid, last` → `mid` luôn là list | B02 |
| 14 | `finally` chạy trước khi `return` trả giá trị ra ngoài | B08 |
| 15 | Truyền tham chiếu: `append` sửa object gốc; gán lại tên thì không | B05 |
| 16 | Class attribute mutable dùng chung giữa mọi instance | B09 |
| 17 | `sorted(key=lambda kv: (-kv[1], kv[0]))` sắp nhiều tiêu chí khác chiều | C01 |
| 18 | Closure + `nonlocal`: mỗi lần gọi factory có state riêng | C04 |
| 19 | Default argument được tính **một lần lúc định nghĩa hàm** | C02 |
| 20 | Late binding: lambda trong comprehension bắt biến, không bắt giá trị | C05 |
| 21 | `dict.fromkeys(keys, [])` → mọi khóa trỏ cùng một list | C03 |
| 22 | `@dataclass`: sinh `__init__/__repr__/__eq__`; default mutable → `ValueError`; `order/frozen/compare=False` | A07, C08, D02 |
| 23 | `t[0] += [...]` với tuple chứa list: list bị sửa **và** vẫn ném `TypeError` | D01 |
| 24 | Gom nhóm list dict bằng thư viện chuẩn (`setdefault`, `get`) | C09 |
| 25 | Decorator có tham số + retry/backoff + `functools.wraps` | D03 |

## 2. Kiểu dữ liệu, toán tử, format

| Immutable (hashable) | Mutable (không hashable) |
|---|---|
| `int, float, bool, str, bytes, tuple, frozenset` | `list, dict, set, bytearray` |

- Khóa dict/phần tử set: `("/api/orders", 503)` OK; `{"/api/orders", 503}` là set, `{"a": 1}` là dict, `[..]` list → đều `TypeError: unhashable`.

```python
7 // 2, 7 / 2, -7 // 2        # 3 3.5 -4   (// làm tròn về -∞, không phải về 0)
f"{1234567:,}"                 # '1,234,567'
f"{0.0456:.1%}"                # '4.6%'  (x100 rồi làm tròn 1 chữ số)
f"{3.14159:.2f}"  f"{42:>5}"  f"{7:03d}"   # '3.14' '   42' '007'
```

## 3. dict & list

```python
rec = {"level": "ERROR", "code": 503}
rec["retry"] = 2; rec["code"] = 500
len(rec), list(rec)            # 3 ['level', 'code', 'retry']  (gán lại không đổi thứ tự)
rec["status"]                  # KeyError  -> dùng rec.get("status", default)
cfg = defaults | user          # dict mới, user ghi đè; defaults không đổi  (|= sửa tại chỗ)
buf = [0,1,2,3,4,5]; buf[1:3] = ["x"]   # [0, 'x', 3, 4, 5]
first, *mid, last = "GET /api/v1/users 200 35ms".split()   # mid = ['/api/v1/users', '200']
{w: words.count(w) for w in words}      # khóa trùng: ghi đè, vị trí theo lần đầu
sorted(freq.items(), key=lambda kv: (-kv[1], kv[0]))   # tần suất giảm, hòa -> chữ cái tăng
```
- `d.copy()`, `list(x)`, `x[:]` = **shallow**: `cp["id"]=8` không ảnh hưởng gốc, nhưng `cp["tags"].append(...)` có. Cần độc lập hoàn toàn → `copy.deepcopy`.
- `dict.fromkeys(["web","db"], [])` → cùng 1 list. Đúng: `{k: [] for k in keys}` hoặc `defaultdict(list)`.

## 4. Hàm: tham chiếu, default, closure, lambda

```python
def normalize(xs):
    xs.append(0)   # sửa list của caller
    xs = [9]       # chỉ gán lại tên cục bộ
    xs.append(1)
    return xs      # data=[5] -> data [5, 0], out [9, 1]

timeout = 5
def call(url, t=timeout): ...  # t mặc định = 5 mãi mãi (tính lúc def)
timeout = 30                   # call("a") -> 'a:5'; call("b", timeout) -> 'b:30'

def make_counter():
    n = 0
    def inc(step=1):
        nonlocal n             # thiếu nonlocal -> UnboundLocalError
        n += step; return n
    return inc                 # c1: 1,6,7 ; c2: 1,11  -> in "7 11"

[lambda: 2**k for k in range(4)]          # gọi -> [8, 8, 8, 8] (k cuối = 3)
[lambda k=k: 2**k for k in range(4)]      # sửa: bind giá trị -> [1, 2, 4, 8]
```

## 5. Generator & iterator

```python
(len(l) for l in lines)        # generator: lazy, tiết kiệm bộ nhớ
sizes = (len(l) for l in ["GET /a","POST /b","GET /c"])
sum(sizes), sum(sizes)         # 19 0  (lần 2 đã cạn, không lỗi)
```
- Lazy: tạo generator không chạy gì. `next()` chỉ kéo đến phần tử đầu tiên thỏa điều kiện:
  `print("start"); print(next(errs))` → `start`, `read ok`, `read E1`, `E1`.

## 6. Ngoại lệ & try/finally

- `finally` luôn chạy; `return "data"` được tính trước, `finally` in `cleanup`, rồi giá trị trả ra → `try ⏎ cleanup ⏎ data`. (`return` trong `finally` sẽ ghi đè.)
- Thiếu khóa dict → `KeyError`; thiếu attribute → `AttributeError`; index list → `IndexError`.
- `batch = (["req-1"], 3); batch[0] += ["req-2"]`: `+=` gọi `list.__iadd__` (sửa list **thành công**) rồi gán lại `batch[0]` → `TypeError`. Kết quả: `TypeError ⏎ (['req-1', 'req-2'], 3)`.

## 7. Regex (`re`)

| Hàm | Ý nghĩa | Ví dụ `s="user=42 retry=3"` |
|---|---|---|
| `re.match` | chỉ khớp **đầu chuỗi** | `re.match(r"\d+", s)` → `None` |
| `re.search` | khớp đầu tiên bất kỳ đâu | `.group()` → `'42'` |
| `re.findall` 0 group | list chuỗi khớp toàn bộ | |
| `re.findall` 1 group | list nội dung group | `r"=(\d+)"` → `['42', '3']` |
| `re.findall` ≥2 group | list **tuple chuỗi** | `[('200','12'), ('503','480')]` |


## 8. Class & dataclass

```python
class Collector:
    lines = []                 # class attribute: a, b dùng chung -> len 3 3
    def __init__(self): self.lines = []   # sửa: instance attribute
```

```python
@dataclass                     # mặc định: __init__, __repr__, __eq__ (không __hash__/__lt__)
class Job:
    name: str
    tags: list = field(default_factory=list)   # `tags: list = []` -> ValueError ngay lúc định nghĩa lớp

@dataclass(order=True, frozen=True)   # order: __lt__...; frozen + eq -> có __hash__
class Event:
    level: int
    msg: str = field(compare=False)    # bỏ khỏi so sánh/eq/hash
# Event(2,"b") == Event(2,"c") -> True -> set còn 2 phần tử
# sorted(...)[0].msg = 'a' (level 1);  print -> "2 a True"
```

## 9. Mẫu bài code viết tay

**C09 – gom nhóm log theo service**
```python
def summarize(records):
    out = {}                               # dict giữ thứ tự xuất hiện đầu tiên
    for r in records:                      # không sửa records
        if "service" not in r or "status" not in r:
            continue
        s = out.setdefault(r["service"], {"n": 0, "err": 0, "max_ms": None})
        s["n"] += 1
        s["err"] += int(r["status"] >= 500)
        ms = r.get("ms", 0)
        s["max_ms"] = ms if s["max_ms"] is None else max(s["max_ms"], ms)
    return out
# {'auth': {'n': 3, 'err': 1, 'max_ms': 80}, 'pay': {'n': 2, 'err': 1, 'max_ms': 1200}}
```
Rubric chính: lọc thiếu khóa, `get("ms", 0)`, đếm `>= 500`, thứ tự khóa, không mutate input.

**D03 – decorator retry có tham số**
```python
import functools, time

def retry(times=3, base=0.5, on=(ConnectionError, TimeoutError), sleep=time.sleep):
    if times < 1:
        raise ValueError("times must be >= 1")      # kiểm tra ngay khi gọi retry(...)
    def deco(fn):
        @functools.wraps(fn)                         # giữ __name__
        def wrapper(*args, **kwargs):
            for i in range(times):
                try:
                    return fn(*args, **kwargs)
                except on:                           # ngoại lệ khác (ValueError) bay ra ngay
                    if i == times - 1:
                        raise                        # ném lại chính ngoại lệ cuối, giữ traceback
                    sleep(base * 2 ** i)             # 0.5, 1.0, 2.0 ...
        return wrapper
    return deco
# get_price("A1") -> 42, calls 3, waits [0.5, 1.0]
```

## 10. Đáp án nhanh (tự giải — PDF không kèm đáp án; mọi output đã chạy lại bằng Python)

| Câu | Đáp án | Câu | Đáp án |
|---|---|---|---|
| A04 | A `int, str, tuple, frozenset` | B10 | D `[('200', '12'), ('503', '480')]` |
| A03 | B `1,234,567 \| 4.6%` | B02 | `GET ['/api/v1/users', '200'] 35ms` |
| A07 | A `__init__, __repr__, __eq__` | B07 | C `19 0` |
| A05 | A `(len(l) for l in lines)` | B08 | A `try ⏎ cleanup ⏎ data` |
| A02 | B `3 3.5 -4` | B05 | `[5, 0] [9, 1]` |
| A01 | D tuple | B09 | D `3 3` |
| A08 | `3 ['level', 'code', 'retry']` | C01 | C `['error', 'timeout', 'ok', 'retry']` |
| A06 | D `KeyError` | C04 | `7 11` |
| B04 | D `{'id': 7, 'tags': ['api', 'retry']}` | C09 | code mẫu §9 |
| B01 | C `[0, 'x', 3, 4, 5]` | C07 | A `None 42 ['42', '3']` |
| B06 | `{'error': 3, 'timeout': 1, 'retry': 1, 'ok': 1}` | C06 | `start` / `read ok` / `read E1` / `E1` |
| B03 | B `5 3 3` | C08 | C ValueError lúc định nghĩa lớp |
| C03 | A `{'web': ['timeout'], 'db': ['timeout']}` | C05 | C `[8, 8, 8, 8]` |
| C02 | `a:5 b:30` | D01 | D `TypeError ⏎ (['req-1', 'req-2'], 3)` |
| D03 | code mẫu §9 | D02 | `2 a True` |

## 11. Bẫy thường gặp

- **`{"/api/orders", 503}`** trông giống tuple nhưng là **set** → không làm khóa được.
- **`.1%` không cắt cụt**: 4.56% → `4.6%`, không phải `4.5%`; `{n:,}` dùng dấu phẩy, không phải chấm.
- **Shallow copy**: phương án "gốc không đổi" (A) sai vì list lồng dùng chung.
- **`findall` nhiều group** → tuple chuỗi, không phải int, không phải toàn bộ match.
- **Generator dùng lần 2** trả rỗng → `sum = 0`, **không** `StopIteration`.
- **Generator lazy**: `start` in trước mọi `read`; `next` dừng ngay sau E1 (không đọc E2).
- **`finally` không làm mất giá trị return** (không thành `None`).
- **`-kv[1]`** đảo chiều tần suất; hòa thì `kv[0]` tăng: `'ok' < 'retry'`, `'error' < 'timeout'`.
- **`re.match` ≠ `re.search`**: chuỗi bắt đầu bằng `user` → match `None`.
- **Late binding lambda** → `[8,8,8,8]` (k cuối = 3), không phải `16` và không `NameError`.
- **`tuple[0] += list`**: list đã đổi dù có `TypeError` (phương án A sai).
- **`compare=False`** → `Event(2,"x") == Event(2,"y")` là `True`; set gộp 2 event level 2.
- **Decorator retry**: quên `wraps` mất `__name__`; ngủ sau lần cuối; bắt `Exception` chung làm retry cả `ValueError`; `raise e` mới vẫn được nhưng `raise` trần gọn nhất.

## 12. Tra nhanh

| Gặp tình huống | Dùng |
|---|---|
| Khóa dict ghép nhiều giá trị | `tuple` `(endpoint, status)` |
| Đọc khóa có thể thiếu | `d.get(k, default)` / `k in d` |
| Gom nhóm theo khóa | `d.setdefault(k, {...})` / `defaultdict(list)` |
| Mỗi khóa một list riêng | `{k: [] for k in keys}` (không `fromkeys(keys, [])`) |
| Ghép cấu hình, bên phải ưu tiên | `defaults \| user` (hoặc `{**a, **b}`) |
| Sao chép có cấu trúc lồng | `copy.deepcopy` |
| Xử lý file lớn từng dòng | generator expression `( ... )` / `yield` |
| Sắp nhiều tiêu chí khác chiều | `key=lambda x: (-x[1], x[0])` |
| State riêng cho hàm | closure + `nonlocal` |
| Lambda trong vòng lặp | `lambda k=k: ...` |
| Default rỗng mutable | `x=None` (hàm, `def f(x=[])` dùng chung) / `field(default_factory=list)` |
| Thuộc tính riêng mỗi object | gán trong `__init__` (`self.x = []`) |
| Object so sánh/sort được, hashable | `@dataclass(order=True, frozen=True)` |
| Decorator có tham số | 3 tầng `retry(...)→deco(fn)→wrapper` + `functools.wraps` |
