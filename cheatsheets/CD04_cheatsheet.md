# CĐ-04 · Thống kê và kiểm định A/B — Cheatsheet

> 30 câu (A: nhận biết 8 · B: thông hiểu 10 · C: vận dụng 9 · D: vận dụng cao 3). Bối cảnh: thử nghiệm nút **Đặt hàng** (sticky button) của sàn TMĐT — chuyển đổi, độ trễ p95, guardrail.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | Trung vị bền với ngoại lai, trung bình bị kéo; p95 = ngưỡng mà ~95% quan sát ≤ | A01, A02 |
| 2 | Độ lệch chuẩn **mẫu** chia $n-1$; Pearson $r$ (nhớ dấu) | B01, B10 |
| 3 | Ngoại lai Tukey: $[Q_1-1{,}5\,IQR;\ Q_3+1{,}5\,IQR]$ | B02 |
| 4 | Tương quan ≠ nhân quả; biến gây nhiễu → cần A/B ngẫu nhiên | A08, D03 |
| 5 | SE tỉ lệ $\sqrt{\hat p(1-\hat p)/n}$; z hai tỉ lệ (gộp) & CI hiệu (không gộp) | A03, B05, B03, C01 |
| 6 | Sai lầm loại I ($\alpha$) / loại II ($\beta$), power $=1-\beta$ tại MDE | A05, A06 |
| 7 | Nghĩa frequentist của CI; độ rộng $\propto z$ | A04, B04 |
| 8 | p-value hai phía $=2[1-\Phi(|z|)]$ | B06 |
| 9 | Không bác bỏ $H_0$ ≠ chứng minh $H_0$; thiếu power | B07 |
| 10 | SRM: $\chi^2$ goodness-of-fit trên số người mỗi nhánh | B09 |
| 11 | Nhìn trộm (peeking) làm FPR ≫ 5% → cố định cỡ mẫu / sequential test | B08 |
| 12 | Cỡ mẫu, power, thời lượng chạy | C06, C03, D01, D02 |
| 13 | Đơn vị phân tích ≠ đơn vị ngẫu nhiên hoá → delta method / bootstrap theo user | C02, C05 |
| 14 | Holm–Bonferroni cho nhiều so sánh | C07 |
| 15 | Triggered analysis + pha loãng hiệu ứng | C09 |
| 16 | CUPED: phương sai × $(1-\rho^2)$ → cỡ mẫu × $(1-\rho^2)$ | C04 |
| 17 | Guardrail (non-inferiority): **cả CI** phải dưới ngưỡng | A07, C08 |
| 18 | Simpson's paradox khi tỉ lệ chia nhánh đổi theo giai đoạn | D03 |

## 2. Thống kê mô tả

- **Trung vị**: sắp xếp trước; $n$ lẻ → phần tử thứ $(n+1)/2$. A01: sorted `150,160,165,170,180,...,2900` → trung vị **180**, trung bình $4325/9\approx$ **480,6** (bị 2900 kéo).
- **p95 = 820 ms**: ~95% yêu cầu xong trong ≤ 820 ms, 5% chậm hơn. Là ngưỡng, không phải trung bình; tính trên yêu cầu, không trên người.
- **Độ lệch chuẩn mẫu**: $s=\sqrt{\sum(x_i-\bar x)^2/(n-1)}$. B01: `42,50,46,38,54` → $\bar x=46$, $\sum d^2=160$, $s=\sqrt{40}=$ **6,32**.
- **Pearson**: $r=\dfrac{\sum(x-\bar x)(y-\bar y)}{\sqrt{\sum(x-\bar x)^2\sum(y-\bar y)^2}}$. B10: p95 tăng, CR giảm → $r=$ **−0,92**.
- **Tukey** (B02, 12 giá trị): $Q_1=(135+140)/2=137{,}5$, $Q_3=(180+190)/2=185$, $IQR=47{,}5$ → ngưỡng $[66{,}25;\ 256{,}25]$ → ngoại lai **260, 420**. Hệ số 3 là ngưỡng "cực đoan".

```python
import statistics as st
st.median(x); st.mean(x); st.stdev(x)      # stdev: n-1 ; pstdev: n
st.correlation(x, y)                       # Pearson r
np.percentile(lat, 95)
```

## 3. Ước lượng & khoảng tin cậy

- $SE(\hat p)=\sqrt{\hat p(1-\hat p)/n}$ (A03). Không có căn → phương sai; $\sqrt{n\hat p(1-\hat p)}$ là SD của **số đếm**.
- Hiệu hai nhánh độc lập (**không gộp**): $SE_{B-A}=\sqrt{\frac{p_A(1-p_A)}{n_A}+\frac{p_B(1-p_B)}{n_B}}$; CI $=(\hat p_B-\hat p_A)\pm 1{,}96\,SE$.
  - B03: $p_A=8\%$, $p_B=9{,}2\%$, $n=5000$ → biên $=$ **1,10 điểm %**.
  - C01: mỗi nhánh $SE=0{,}6/1{,}96=0{,}306$ → $SE_{B-A}=\sqrt{2}\cdot0{,}306=0{,}433$ → $1{,}0\pm0{,}85=[+0{,}15;+1{,}85]$ → **có ý nghĩa** dù hai CI chồng lấn. Cộng **phương sai**, không cộng biên.
- **CI 95%**: tính chất của **quy trình** — lặp lại nhiều lần, ~95% số khoảng chứa tham số thật. Không nói về từng người dùng, không nói 95% ước lượng mới rơi vào đoạn cũ.
- Độ rộng $=2z\cdot SE$ → 95%→99%: $2{,}576/1{,}96\approx$ **1,31 lần** (B04). $1{,}645$ là z của 90% hai phía / 95% một phía.
- CI không chứa 0 ⇔ $p<0{,}05$ (hai phía).

## 4. Kiểm định giả thuyết

| | $H_0$ đúng | $H_1$ đúng |
|---|---|---|
| Bác bỏ $H_0$ | **Loại I** (FP), xác suất $\alpha$ | Đúng — **power** $=1-\beta$ |
| Không bác bỏ | Đúng, $1-\alpha$ | **Loại II** (FN), xác suất $\beta$ |

- z hai tỉ lệ, **tỉ lệ gộp** $\hat p=\frac{x_A+x_B}{n_A+n_B}$: $z=\dfrac{\hat p_B-\hat p_A}{\sqrt{\hat p(1-\hat p)(1/n_A+1/n_B)}}$. B05: $8\%$ vs $9\%$, $n=8000$, $\hat p=0{,}085$ → $z=$ **2,27**.
- p hai phía $=2[1-\Phi(|z|)]$. B06: $z=1{,}80$ → $2\cdot0{,}0359=$ **0,072** > 0,05 → không bác bỏ.
- **Power 80% với MDE 0,5 điểm %**: nếu hiệu ứng thật đúng = MDE, xác suất bác bỏ $H_0$ ≈ 80%; hiệu ứng nhỏ hơn → power thấp hơn.
- B07: $p=0{,}33$, CI $[-0{,}4;+1{,}2]$ chứa cả 0 lẫn mức đáng giá +0,5 → **chưa kết luận được, thiếu power**. "Absence of evidence ≠ evidence of absence".
- **SRM** (B09): kỳ vọng 70/30 của 50.000 → $E=35000/15000$; $\chi^2=\frac{400^2}{35000}+\frac{400^2}{15000}=$ **15,24** > 10,83 → SRM ($p<0{,}001$), **không đọc kết quả**, tìm lỗi phân nhánh/log.
- **Peeking** (B08): xem p mỗi ngày 14 ngày, dừng khi $p<0{,}05$ → FPR ≈ **20%**. Sửa: chốt cỡ mẫu trước, đọc 1 lần; hoặc sequential (alpha spending, mSPRT).

## 5. Cỡ mẫu, power, thời lượng

$$n_{\text{/nhánh}}=\frac{(z_{1-\alpha/2}+z_{1-\beta})^2\,[p_1(1-p_1)+p_2(1-p_2)]}{(p_2-p_1)^2},\quad z=1{,}96;\ 0{,}84$$

```python
n = lambda p1, p2: (1.96 + 0.84)**2 * (p1*(1-p1) + p2*(1-p2)) / (p2 - p1)**2
n(.08, .09)    # 12191   -> C06: 12,2 nghìn người
n(.05, .055)   # 31195   -> D01: MDE tương đối +10% => p2 = 5,5%
n(.06, .066)   # 25707   -> D02
```
- **Thời lượng** = $n$/(lưu lượng mỗi nhánh/ngày). D01: $15000\cdot40\%/2=3000$/nhánh/ngày → $31195/3000=10{,}4$ → **11 ngày** (làm tròn lên).
- **Power** (C03): $\text{power}\approx\Phi\!\left(\frac{|p_B-p_A|}{SE}-1{,}96\right)$, $SE=\sqrt{\frac{.05\cdot.95}{6000}+\frac{.06\cdot.94}{6000}}=0{,}00416$ → $2{,}40-1{,}96=0{,}44$ → $\Phi(0{,}44)=$ **0,67**. (Bẫy: $\Phi(2{,}40)=0{,}99$ quên trừ 1,96.)
- $n\propto\sigma^2/\Delta^2$: MDE giảm một nửa → $n$ ×4.
- **CUPED** (C04): $\sigma^2\to\sigma^2(1-\rho^2)$; $\rho=0{,}6$ → $n=0{,}64\cdot20000=$ **12.800**.
- Chạy ít nhất **trọn tuần** (chu kỳ ngày trong tuần, novelty effect), kể cả khi $n$ đủ sớm hơn.

## 6. Thiết kế & phân tích nâng cao

- **Chỉ số**: OEC (chỉ số chính, vd tỉ lệ đặt hàng) · **guardrail** (không được xấu đi: p95 độ trễ, tỉ lệ lỗi API, huỷ đơn) · diagnostic (số lần nhấn nút — giải thích cơ chế). Màu/vị trí nút là **treatment**.
- **Guardrail non-inferiority** (C08): ngưỡng +50 ms, CI $[+12;+71]$ → **chưa đạt** (cận trên > 50) — không phải "chắc chắn vi phạm", cũng không "đạt" vì điểm 41 < 50.
- **Đơn vị phân tích ≠ đơn vị ngẫu nhiên hoá** (C02): lượt xem cùng user tương quan → SE quá nhỏ. Tỉ số $\sum\text{click}/\sum\text{view}$ theo user → **delta method** hoặc **bootstrap theo user**.
- **CI cho p95** (C05): bootstrap theo user (resample user kèm mọi request), tính $\Delta$p95, lấy phân vị 2,5% & 97,5%.
- **Holm** (C07): sắp p tăng, so $p_{(k)}$ với $\alpha/(m-k+1)$, dừng ở lần trượt đầu: $0{,}004<0{,}0125$ ✓, $0{,}015<0{,}0167$ ✓, $0{,}03>0{,}025$ ✗ → **V1, V2**. Bonferroni ($\alpha/m$) chỉ V1. Không hiệu chỉnh: FWER tới $1-0{,}95^4\approx18{,}5\%$.
- **Triggered analysis** (C09): chỉ phân tích người vào trang thanh toán (điều kiện như nhau ở 2 nhánh): +2 điểm % → toàn app pha loãng $25\%\cdot2=$ **+0,5 điểm %**.
- **Simpson** (D03): xem §7.

```python
def boot_p95_diff(df, B=2000):          # df: user, arm, latency
    users = {a: df[df.arm == a].groupby("user")["latency"].apply(list) for a in "AB"}
    out = []
    for _ in range(B):
        s = {a: np.concatenate(users[a].sample(len(users[a]), replace=True).values) for a in "AB"}
        out.append(np.percentile(s["B"], 95) - np.percentile(s["A"], 95))
    return np.percentile(out, [2.5, 97.5])
```

## 7. Mẫu bài tự luận

**D02 – Kế hoạch thử nghiệm nút sticky**
- (a) $H_0$: CR(B) = CR(A); $H_1$: khác (hai phía). **OEC**: tỉ lệ đặt hàng thành công / người vào trang thanh toán (6,0%). **Guardrail** khai báo trước: p95 API đặt hàng B không cao hơn A quá 50 ms và < SLO 600 ms; tỉ lệ huỷ đơn 24h không tăng quá +0,5 điểm % (4,0%); tỉ lệ lỗi API không tăng. Diagnostic: tỉ lệ nhấn nút.
- (b) Ngẫu nhiên hoá theo **user** (ổn định qua phiên/thiết bị). $p_1=6\%$, $p_2=6{,}6\%$ → $n\approx$ **25.708/nhánh** (≈51.400 tổng). 12.000 người/ngày, 50/50 → ~4,3 ngày → chạy **7 ngày** (trọn chu kỳ tuần, giảm novelty; có thể 14 ngày nếu cần), chỉ tính người lần đầu vào trang thanh toán trong kỳ.
- (c) **A/A test**/kiểm tra log; **SRM** $\chi^2$ trên 50/50 (bác bỏ ở $p<0{,}001$ → dừng đọc); kiểm tra chỉ số trước thử nghiệm cân bằng; triggering giống nhau. Chống peeking: cố định $n$ & ngày kết thúc, chỉ đọc 1 lần cuối (hoặc sequential có alpha spending).
- (d) Triển khai khi: OEC tăng có ý nghĩa ($p<0{,}05$, CI không chứa 0, lý tưởng ≥ +0,6) **và** mọi guardrail đạt (cả CI dưới ngưỡng). Guardrail trượt → không ship, sửa rồi chạy lại. Chỉ số phụ chỉ để giải thích (nhiều chỉ số → Holm/Bonferroni), không dùng để "cứu" OEC không ý nghĩa.

**D03 – Nghịch lý Simpson**
| Giai đoạn | A | B | B − A |
|---|---|---|---|
| 1 (khuyến mãi, 90/10) | 9000/90000 = 10,0% | 1050/10000 = 10,5% | +0,5 |
| 2 (thường, 50/50) | 2500/50000 = 5,0% | 2700/50000 = 5,4% | +0,4 |
| Gộp | 11500/140000 = 8,21% | 3750/60000 = 6,25% | **−1,96** |
- (b) Biến gây nhiễu = **giai đoạn**: ảnh hưởng cả CR (khuyến mãi CR cao) lẫn tỉ lệ được vào nhánh (A có 64% người từ giai đoạn KM, B chỉ 17%). Trong mỗi giai đoạn, phân nhánh vẫn ngẫu nhiên → so sánh hợp lệ.
- (c) Phân tầng theo giai đoạn rồi gộp có trọng số (vd trung bình +0,45 điểm %, hoặc CMH / hồi quy có biến giai đoạn). Giai đoạn 2: $\hat p=0{,}052$, $SE=\sqrt{0{,}052\cdot0{,}948\cdot2/50000}=0{,}0014$, $z=0{,}004/0{,}0014=$ **2,85** > 1,96 ($p\approx0{,}004$) → B tốt hơn có ý nghĩa; CI không gộp $[+0{,}12;+0{,}68]$ điểm %.
- (d) Không thay đổi tỉ lệ chia nhánh giữa chừng (hoặc chỉ phân tích theo từng giai đoạn/tầng); tránh chạy trong giai đoạn bất thường (KM); báo cáo luôn theo tầng + kiểm tra SRM theo ngày.

## 8. Đáp án nhanh

✓ = đáp án chính thức từ PDF · ✎ = tự giải (đã kiểm Python)

| Câu | Đáp án | Câu | Đáp án |
|---|---|---|---|
| A07 | C ✓ (p95 độ trễ + lỗi API) | B08 | B ✓ (FPR ~20%) |
| A02 | D ✓ | B03 | **1,10** điểm % ✎ |
| A08 | C ✓ (biến gây nhiễu) | C06 | **12,2** nghìn ✎ (12.191) |
| A03 | B ✎ $\sqrt{\hat p(1-\hat p)/n}$ | C01 | C ✓ $[+0{,}15;+1{,}85]$ |
| A05 | C ✓ (loại I, $\alpha$) | C02 | A ✓ (delta method/bootstrap user) |
| A04 | C ✓ | C03 | **0,67** ✎ |
| A06 | C ✓ | C07 | D ✓ (V1, V2) |
| A01 | D ✓ (180; 480,6) | C09 | D ✓ (+0,5) |
| B06 | A ✓ ($p\approx0{,}072$) | C04 | A ✓ (12.800) |
| B05 | **2,27** ✎ | C08 | A ✓ (chưa đạt) |
| B01 | **6,32** ✎ | C05 | C ✓ (bootstrap theo user) |
| B10 | **−0,92** ✎ | D01 | **11** ngày ✎ |
| B02 | C ✓ (260, 420) | D02 | Tự luận ✎ — xem §7 ($n\approx25.708$/nhánh, 7 ngày) |
| B04 | A ✓ (1,31 lần) | D03 | Tự luận ✎ — xem §7 ($z\approx2{,}85$) |
| B07 | B ✓ (thiếu power) | | |
| B09 | **15,24** ✎ (SRM) | | |

Tổng: 19 câu đáp án chính thức (bỏ qua ghi chú "bản in"), 11 câu tự giải (A03, 8 câu điền số, 2 tự luận).

## 9. Bẫy thường gặp

- **Trung vị lấy phần tử giữa danh sách chưa sắp** (2900) hoặc tự ý bỏ ngoại lai rồi tính (175; 178,1).
- **p95 "95% chậm hơn"** là đảo chiều; p95 không phải trung bình 95% nhanh nhất.
- **Tương quan mạnh / $p$ rất nhỏ** không loại trừ biến gây nhiễu hay self-selection.
- **SE**: quên chia $n$ hoặc quên căn; $\sqrt{np(1-p)}$ là của số đếm.
- **Loại I khống chế bởi power** — sai, bởi $\alpha$. Khi $H_0$ đúng, xác suất không bác bỏ là $1-\alpha=95\%$, không phải power.
- **Power 80% cho mọi hiệu ứng** — sai, chỉ tại MDE; không phải xác suất hậu nghiệm "B tốt hơn".
- **p-value**: lấy $\Phi(z)$ (phần thân), $1-p$, hoặc quên ×2 cho hai phía.
- **Độ rộng CI tỉ lệ với 99/95** hay dùng 1,645 thay 1,96; độ tin cậy cao hơn → CI **rộng** hơn.
- **CI hiệu**: dùng SE 1 nhánh (±0,6) hoặc cộng thẳng biên (±1,2); hai CI chồng lấn **không** suy ra không ý nghĩa.
- **$p>0{,}05$ ⇒ bằng nhau** — sai; đổi sang một phía sau khi thấy dữ liệu / chạy đến khi $p<0{,}05$ = p-hacking.
- **SRM**: quên chia cho $E$ hoặc dùng 50/50 thay 70/30 kỳ vọng.
- **Peeking**: "mỗi lần dùng đúng α" không đủ — xác suất ≥1 lần vượt tích luỹ.
- **Coi lượt xem/request là độc lập** khi ngẫu nhiên hoá theo user → p nhỏ giả tạo; Bonferroni/một phía không chữa được.
- **log rồi t-test** so trung bình nhân, không phải p95.
- **CUPED**: dùng $1-\rho$, $\sqrt{1-\rho^2}$ (SD) hay $\rho^2$ thay vì $1-\rho^2$.
- **MDE tương đối** +10% của 5% là 5,5% (không phải 15%); thời lượng phải tính theo **lưu lượng mỗi nhánh** và làm tròn **lên**.
- **Power**: tra $\Phi(2{,}40)$ quên trừ 1,96.
- **Holm**: ngưỡng $\alpha/m^2$ không tồn tại; Holm dừng ở lần trượt đầu tiên.
- **Triggered**: so người đã nhấn nút (self-selected) với toàn nhánh A; báo +2 điểm % cho toàn app (phải pha loãng).
- **Guardrail**: điểm ước lượng < ngưỡng chưa đủ; OEC tăng không cho phép bỏ guardrail.
- **Gộp dữ liệu nhiều giai đoạn với tỉ lệ chia khác nhau** → Simpson (+0,5, +0,4 nhưng gộp −1,96).

## 10. Tra nhanh

| Gặp tình huống | Dùng |
|---|---|
| Đo "điển hình" khi có ngoại lai | Trung vị (không trung bình) |
| Đuôi chậm độ trễ | p95 / p99 |
| Phát hiện ngoại lai | Tukey $Q_{1,3}\mp1{,}5\,IQR$ |
| SD từ mẫu nhỏ | `stdev` (chia $n-1$) |
| So hai tỉ lệ (kiểm định) | z hai tỉ lệ, SE **gộp** |
| CI cho hiệu hai tỉ lệ | SE **không gộp**, $\pm1{,}96\,SE$ |
| Tính p hai phía | $2[1-\Phi(|z|)]$ |
| Chia nhánh lệch cấu hình? | SRM: $\chi^2=\sum(O-E)^2/E$, ngưỡng 10,83 ($p=0{,}001$) |
| Cỡ mẫu cho MDE | $(z_{\alpha/2}+z_\beta)^2[\sum p(1-p)]/\Delta^2$ |
| Số ngày chạy | $\lceil n/\text{user mỗi nhánh mỗi ngày}\rceil$, ≥ 1 tuần |
| Power cho $n$ cho trước | $\Phi(|\Delta|/SE-1{,}96)$ |
| Giảm phương sai / cỡ mẫu | CUPED: × $(1-\rho^2)$ |
| Chỉ số tỉ số, nhiều sự kiện/user | Delta method / bootstrap theo user |
| CI cho phân vị (p95) | Bootstrap theo user, phân vị 2,5–97,5% |
| Nhiều biến thể / nhiều chỉ số | Holm–Bonferroni (FWER) |
| Muốn xem kết quả giữa chừng | Sequential test (alpha spending, mSPRT) |
| Tính năng chỉ ở 1 trang | Triggered analysis, rồi pha loãng × tỉ lệ trigger |
| Guardrail "không tệ hơn X" | Cận trên CI < X (non-inferiority) |
| Tỉ lệ chia nhánh đổi theo thời gian | Phân tầng theo giai đoạn (tránh Simpson) |
| Tương quan quan sát, muốn nhân quả | A/B ngẫu nhiên hoá |
