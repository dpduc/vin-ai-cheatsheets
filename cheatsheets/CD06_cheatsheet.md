# CĐ-06 · Xử lý dữ liệu với pandas — Cheatsheet

> 30 câu (A: nhận biết 8 · B: thông hiểu 10 · C: vận dụng 9 · D: vận dụng cao 3). Bối cảnh: đơn hàng, đặt phòng khách sạn, log web.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | `read_csv(sep=";", decimal=",")` cho CSV kiểu VN/châu Âu | A02 |
| 2 | Lọc nhiều điều kiện: `&` `|` `~` + **ngoặc từng vế**; không dùng `and/or/in` | A01, A04, B01 |
| 3 | `isin([...])` để lọc theo danh sách; `~isin` tìm bản ghi "mồ côi" | A04, B07 |
| 4 | `query("col == @bien")` — biến Python cần `@` | A03, C01 |
| 5 | NaN ≠ NaN → dùng `isna()`; `count` bỏ NaN, `size` đếm cả NaN | A08, A06, B04 |
| 6 | `concat` giữ index cũ (trùng) → `ignore_index=True`; cột thiếu → NaN | A07, C04 |
| 7 | `groupby` bỏ khóa NaN mặc định (`dropna=True`) | B06 |
| 8 | `transform` trả Series cùng độ dài/index với bảng gốc (tỷ trọng, fillna theo nhóm) | A05, C02, C08 |
| 9 | `loc` theo nhãn (gồm cả đầu cuối) vs `iloc` theo vị trí (bỏ đầu cuối) | B03 |
| 10 | `merge`: cột trùng tên → `_x/_y`; khóa khác kiểu → ValueError | B08, C05 |
| 11 | Copy-on-Write (pandas ≥ 3.0): gán chuỗi `df[mask]["c"] = v` không đổi df | B02 |
| 12 | `pivot` lỗi khi trùng cặp; `pivot_table` gộp bằng `mean` mặc định | B05 |
| 13 | `to_datetime(format=..., errors="coerce")`, `.dt.hour/.month/.days`, Timedelta | B09, C06, C07 |
| 14 | Nullable integer `"Int64"` giữ `<NA>` | B10 |
| 15 | Lấy nguyên dòng max theo nhóm: `loc[groupby(...).idxmax()]` | C03 |
| 16 | Tỷ lệ tổng hợp = trung bình **có trọng số**, không phải mean của rate | D01 |
| 17 | `merge_asof` (backward, khớp cả bằng) cho bảng giá theo hiệu lực | D02 |
| 18 | Báo cáo theo giờ liên tục: `floor("h")` + `reindex(date_range)` / `resample` | C09 |
| 19 | Báo cáo theo danh mục: left merge từ bảng danh mục + `fillna(0).astype(int)` | D03 |

## 2. Đọc dữ liệu & kiểu dữ liệu

```python
pd.read_csv("f.csv", sep=";", decimal=",")      # 1250,5 -> 1250.5   (A02)
# thousands="," chỉ bỏ dấu phân cách nghìn, KHÔNG đổi dấu thập phân
s.astype("Int64")       # 2.0, NaN -> 2, <NA>     (B10)
s.astype(int)           # NaN -> lỗi IntCastingNaNError
pd.to_numeric(s, errors="coerce")   # "-" -> NaN
```

## 3. Lọc dòng

```python
df[(df["gia"] > 1000) & (df["tinh"] == "HN")]   # ngoặc bắt buộc (& ưu tiên cao hơn >)
df[df["status"].isin([404, 500])]
df[~df["path"].str.startswith("/api")]
df["ms"].between(100, 300)        # mặc định inclusive="both"  -> gồm 100 và 300
bk.query("thanh_pho == @ks")      # @ tham chiếu biến
bk.query("loai != 'Std' or dem >= 4")   # trong query được dùng and/or
```
- `loc[:, "a":"c"]` nhãn, **gồm** `c`; `iloc[:, 2:4]` → cột vị trí 2,3 (bỏ 4).
- `loc["a":"c"]` (không có `:,`) là cắt **dòng** theo nhãn.

## 4. NaN

- `s == np.nan` luôn False → `(s == np.nan).sum() = 0`; dùng `s.isna().sum()`.
- `sum()` bỏ qua NaN của chính cột đó; không bị ảnh hưởng bởi NaN ở cột khác.
- `groupby("k")` **bỏ dòng có khóa NaN** → tổng theo nhóm có thể < tổng cột. Muốn giữ: `groupby("k", dropna=False)`.

## 5. groupby / agg / transform

| Hàm | Kết quả | NaN |
|---|---|---|
| `g.size()` | số dòng/nhóm | đếm cả NaN |
| `g["c"].count()` | số giá trị không NaN | bỏ NaN |
| `g["c"].nunique()` | số giá trị khác nhau | bỏ NaN |
| `g["c"].agg("sum")` | 1 dòng/nhóm (index = khóa) | |
| `g["c"].transform("sum")` | **cùng độ dài & index** với bảng gốc | |

```python
don["ty_le"] = don["tien"] / don.groupby("kh")["tien"].transform("sum")   # tỷ trọng trong nhóm
bk["gia"].fillna(bk.groupby("loai")["gia"].transform("mean"))            # điền theo mean nhóm
bk.loc[bk.groupby("ks")["gia"].idxmax()]          # nguyên dòng có giá max mỗi nhóm
bk.groupby("ks")["huy"].agg(n="size", rate="mean")   # named aggregation
```
- `groupby().max()` lấy max **từng cột độc lập** → cặp (mã, giá) có thể lệch dòng.
- Tỷ lệ toàn chuỗi: $\dfrac{\sum rate_i \cdot n_i}{\sum n_i}$ (D01: $\frac{2+1+1}{20}=0.2$).
- `.to_dict()` của kết quả groupby: khóa sắp tăng dần (vd `{'/book': 1, '/home': 3, '/rooms': 2}`).

## 6. Ghép bảng

```python
pd.concat([t7, t8])                     # index [0,1,2,0,1] (giữ index cũ, không lỗi)
pd.concat([web, app], ignore_index=True)   # index 0..n-1; cột chỉ có ở 1 bảng -> NaN ở bảng kia
don.merge(sp, on="ma_sp")               # cột trùng ngoài khóa -> gia_x, gia_y (suffixes)
don[~don["ma_kh"].isin(kh["ma_kh"])]    # đơn mồ côi (anti-join)
don.merge(kh, how="left", indicator=True).query("_merge == 'left_only'")   # cách 2
kh["ma_kh"] = kh["ma_kh"].astype(int)   # ép cùng kiểu trước merge (int vs object -> ValueError)
pd.merge_asof(bk.sort_values("ngay"), gia, left_on="ngay", right_on="tu_ngay")
```
- `merge_asof`: hai bảng phải **sort** theo khóa; mặc định `direction="backward"`, `allow_exact_matches=True` → lấy dòng `tu_ngay <= ngay` gần nhất. Kết quả theo thứ tự đã sort (D02: `['B1','B3','B2'] [900,1200,1500]`).

## 7. Reshape

- `pivot`: cặp (index, columns) trùng → **ValueError**.
- `pivot_table`: `aggfunc="mean"` mặc định → (100+300)/2 = 200.

## 8. Copy-on-Write (pandas ≥ 3.0)

```python
df[df["gia"] > 900]["gia"] = 1000     # chained assignment: df KHÔNG đổi (chỉ cảnh báo, không exception)
df.loc[df["gia"] > 900, "gia"] = 1000 # đúng
```
Hàm không được sửa input → `d = log.copy()` hoặc chỉ tạo object mới.

## 9. Datetime

```python
pd.to_datetime(s, format="%d/%m/%Y", errors="coerce")   # "30/02/2026" -> NaT
s.dt.hour / .dt.month / .dt.day / .dt.floor("h")
(co - ci).dt.days                     # Timedelta -> số ngày
(t_max - t_min).total_seconds() / 3600   # số giờ (C06: 08:05 -> 00:20 hôm sau = 16.25)
pd.date_range(start, end, freq="h")
```
- `min()/max()` bỏ qua NaT.

## 10. Mẫu bài code viết tay

**C09 – báo cáo theo giờ liên tục**
```python
def bao_cao_gio(log):
    d = log.copy()
    d["ts"] = pd.to_datetime(d["ts"], format="%Y-%m-%d %H:%M:%S", errors="coerce")
    d = d.dropna(subset=["ts"])
    d["ms"] = pd.to_numeric(d["ms"], errors="coerce")       # "-" -> NaN
    d["loi"] = d["status"] >= 500
    d["gio"] = d["ts"].dt.floor("h")
    g = d.groupby("gio").agg(so_req=("status", "size"),
                             ty_le_loi=("loi", "mean"),
                             ms_tb=("ms", "mean"))           # mean bỏ NaN
    g = g.reindex(pd.date_range(g.index.min(), g.index.max(), freq="h"))
    g["so_req"] = g["so_req"].fillna(0).astype(int)
    g["ty_le_loi"] = g["ty_le_loi"].fillna(0.0)             # ms_tb giữ NaN
    return g
```

**D03 – báo cáo theo khách sạn (kể cả KS không có booking)**
```python
def bao_cao_ks(bk, ks):
    b = bk.copy()
    ok = b["trang_thai"] == "ok"
    b["huy"] = (~ok).astype(int)
    b["dem"] = (b["checkout"] - b["checkin"]).dt.days.where(ok, 0)
    b["dt"] = b["dem"] * b["gia_dem"]
    g = b.groupby("ma_ks").agg(so_dat=("ma_dat", "size"), so_huy=("huy", "sum"),
                               dem_ban=("dem", "sum"), doanh_thu=("dt", "sum")).reset_index()
    r = ks[["ma_ks", "ten"]].merge(g, on="ma_ks", how="left")
    for c in ["so_dat", "so_huy", "dem_ban", "doanh_thu"]:
        r[c] = r[c].fillna(0).astype(int)
    r["ty_le_huy"] = (r["so_huy"] / r["so_dat"].where(r["so_dat"] > 0)).fillna(0.0)
    r = r[["ma_ks", "ten", "so_dat", "ty_le_huy", "dem_ban", "doanh_thu"]]
    return r.sort_values(["doanh_thu", "ma_ks"], ascending=[False, True]).reset_index(drop=True)
```
(Ví dụ: HL 4200, DL 3300, SG 0.)

## 11. Đáp án nhanh (tự giải)

| Câu | Đáp án | Câu | Đáp án |
|---|---|---|---|
| A02 | B | B04 | `{'/book': 1, '/home': 3, '/rooms': 2}` / `{'/book': 1, '/home': 2, '/rooms': 2}` |
| A07 | D `[0,1,2,0,1]` | B05 | D |
| A03 | C | B01 | `4 480` |
| A04 | B | B10 | C |
| A08 | A `0 2` | C03 | D |
| A06 | D `size()` | C06 | `{0: 1, 8: 2, 9: 1, 23: 1}` / `16.25` |
| A05 | C | C02 | `[0.2, 0.6, 0.6, 1.0, 0.4, 0.2]` / `['Binh', 'An', 'Chi']` |
| A01 | C | C08 | A `[600.0, 750.0, 1500.0, 900.0, 1400.0, 1300.0]` |
| B09 | `1 3 31` | C04 | `(5, 4) 3 2` |
| B03 | A | C01 | `['B3', 'B6', 'B5'] 3000` |
| B08 | B | C05 | D |
| B06 | C `570 450` | C07 | `[3, 1, 3, 4]` / `{7: 6, 8: 5}` |
| B07 | A | D01 | C (0.2) |
| B02 | C | D02 | D |

## 12. Bẫy thường gặp

- **`thousands=","`** thay cho `decimal=","` → không đọc được số thập phân; đảo `sep`/`decimal` cũng sai.
- **`and` / `in` với Series** → `ValueError: truth value ... ambiguous`. Thiếu ngoặc với `&` → lỗi so sánh do thứ tự ưu tiên. `|` là "hoặc", không phải "và".
- **`s == [404, 500]`** so sánh từng phần tử theo vị trí (lỗi độ dài), không phải "thuộc".
- **`log.loc[[404, 500]]`** lọc theo nhãn index, không theo giá trị cột.
- **`query("c == ks")`** tìm cột tên `ks`; `'ks'` là chuỗi literal; f-string `{ks}` mất dấu nháy → lỗi cú pháp.
- **concat index trùng không báo lỗi** — chỉ là index lặp lại.
- **`count()` thay `size()`** → bỏ sót request có `user_id` NaN; `value_counts` cho từng giá trị, không phải tổng.
- **transform ≠ agg**: agg ra 3 dòng (số nhóm), transform ra 8 dòng (số dòng gốc).
- **`iloc[:, 2:4]`** chỉ 2 cột (bỏ cận phải); `iloc` không nhận nhãn chuỗi.
- **merge không tự ép kiểu** "101" ↔ 101; đổi `how` không giải quyết. Khóa merge không cần làm index.
- **`indicator=True` với `how="left"`** chỉ có `both`/`left_only` — lọc `right_only` ra rỗng.
- **CoW**: không phải exception; df giữ nguyên.
- **pivot không "giữ dòng sau"**; pivot_table không cộng (mean).
- **`astype(int)` với NaN** không thành 0, không giữ NaN → lỗi; pandas có `Int64` nullable nên không bắt buộc `dropna`.
- **`sort_values("gia").groupby().head(1)`** lấy giá **thấp nhất** (sort tăng).
- **Mean của các rate** (0.283) hay median ≠ tỷ lệ toàn bộ khi các nhóm có cỡ khác nhau.
- **merge_asof**: ngày trùng đúng mốc hiệu lực vẫn khớp (B2 15/07 → 1500); thứ tự kết quả theo bảng đã sort.
- **Chênh lệch thời gian qua nửa đêm**: tính bằng Timedelta, đừng trừ giờ (`0 - 8`).
- **Đêm lưu trú qua tháng** được gán vào tháng của `ci` khi group theo `ci.dt.month`.

## 13. Tra nhanh

| Gặp tình huống | Dùng |
|---|---|
| CSV `;` + thập phân `,` | `read_csv(sep=";", decimal=",")` |
| Lọc nhiều điều kiện | `(c1) & (c2)`, `|`, `~` |
| Giá trị thuộc danh sách / bảng khác | `isin(...)`, phủ định `~isin` |
| Biến Python trong query | `@bien` |
| Kiểm tra thiếu | `isna()` / `notna()` |
| Đếm dòng/nhóm (kể cả NaN) | `groupby().size()` |
| Đếm giá trị khác nhau | `nunique()` |
| Gắn giá trị nhóm vào từng dòng | `groupby()[c].transform(f)` |
| Điền thiếu theo nhóm | `fillna(groupby().transform("mean"))` |
| Nguyên dòng max/min mỗi nhóm | `df.loc[groupby()[c].idxmax()]` |
| Nối dọc, đánh lại index | `pd.concat([...], ignore_index=True)` |
| Join theo khóa | `merge(on=, how=)`; khác kiểu → `astype` trước |
| Bản ghi không khớp | `~isin` hoặc `indicator=True` + `left_only` |
| Ghép theo khoảng hiệu lực/ngày gần nhất trước | `pd.merge_asof` (đã sort) |
| Bảng chéo có trùng cặp | `pivot_table(aggfunc=...)` |
| Gán có điều kiện | `df.loc[mask, col] = v` |
| Int có ô thiếu | `astype("Int64")` |
| Chuỗi → số, giá trị rác | `pd.to_numeric(errors="coerce")` |
| Chuỗi → ngày, ngày sai | `pd.to_datetime(format=, errors="coerce")` |
| Số ngày / số giờ giữa 2 mốc | `.dt.days` / `.total_seconds()/3600` |
| Chuỗi giờ liên tục kể cả giờ trống | `dt.floor("h")` + `reindex(pd.date_range(..., freq="h"))` hoặc `resample("h")` |
| Tỷ lệ toàn bộ từ bảng tóm tắt | $\sum(rate\cdot n)/\sum n$ |
| Báo cáo đủ mọi thực thể danh mục | `danh_muc.merge(agg, how="left")` + `fillna(0).astype(int)` |
| Sắp nhiều khóa khác chiều | `sort_values([a, b], ascending=[False, True])` |
