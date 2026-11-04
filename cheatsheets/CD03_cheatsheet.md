# CĐ-03 · Xác suất — Cheatsheet

> 30 câu (A: nhận biết 8 · B: thông hiểu 10 · C: vận dụng 9 · D: vận dụng cao 3). Bối cảnh: API/request, A/B test, phát hiện gian lận, SRE/latency.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | Độc lập $P(A\cap B)=P(A)P(B)$ ≠ xung khắc $P(A\cap B)=0$; xung khắc (P>0) ⇒ **không** độc lập | A04 |
| 2 | "Ít nhất một" → phần bù $1-(1-p)^n$ | A08, C01 |
| 3 | Công thức cộng $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ | A02 |
| 4 | Xác suất có điều kiện $P(A\mid B)=P(A\cap B)/P(B)$ | A03 |
| 5 | Đếm: tổ hợp $\binom{n}{k}$ vs chỉnh hợp $A_n^k$ vs lặp $n^k$; tử & mẫu cùng cách đếm | A01, B01 |
| 6 | Kỳ vọng $E[X]=\sum x\,p(x)$; tuyến tính luôn đúng (kể cả phụ thuộc) | A05, A06, D02 |
| 7 | Phương sai: $\mathrm{Var}(aX)=a^2\mathrm{Var}X$; tổng/hiệu cần $\pm2\mathrm{Cov}$ | A06, B06, B05 |
| 8 | Poisson: $E=\mathrm{Var}=\lambda$; đổi $\lambda$ theo độ dài khoảng thời gian | A07, B10, C08 |
| 9 | Nhị thức $\binom{n}{k}p^k(1-p)^{n-k}$; siêu bội khi rút không hoàn lại | B08, C01 |
| 10 | Chuẩn: $z=(x-\mu)/\sigma$, quy tắc 68–95–99,7, phân vị $\mu+z\sigma$ | B09, C07 |
| 11 | SD của trung bình $\sigma/\sqrt n$ | B07 |
| 12 | Xác suất toàn phần (trung bình có trọng số) + Bayes | B02, B03, C02 |
| 13 | Base rate: precision phụ thuộc tỉ lệ nền; accuracy gây hiểu lầm khi lớp hiếm | B04, C04, D01 |
| 14 | Nghịch lý Simpson (trọng số nhóm khác nhau) | C03 |
| 15 | Kỳ vọng số lần thử (retry), ngưỡng quyết định theo chi phí kỳ vọng | C05, C06 |
| 16 | Xấp xỉ chuẩn cho nhị thức/Poisson; khoảng 95% $\mu\pm1{,}96\sigma$ | C09, D03 |
| 17 | Biến chỉ báo + tuyến tính kỳ vọng (đếm worker rảnh) | D02 |

## 2. Biến cố, đếm

- **Độc lập**: $P(A\mid B)=P(A)$ ⇔ $P(A\cap B)=P(A)P(B)$. **Xung khắc**: $A\cap B=\varnothing$. Với $P(A),P(B)>0$ hai điều này **loại trừ nhau**.
- Hợp: $P(A\cup B)=P(A)+P(B)-P(A\cap B)$ (A02: $0{,}3+0{,}2-0{,}1=0{,}4$). Chỉ dùng $P(A)P(B)$ cho giao khi **đã biết** độc lập.
- Có điều kiện: $P(\text{mua}\mid\text{mobile})=0{,}06/0{,}4=0{,}15$ — **chia** cho xác suất điều kiện.
- Phần bù: $P(\ge1)=1-P(0)$; A08: $1-0{,}98^{50}\approx0{,}636$ (không phải $50\cdot0{,}02=1$ — đó là số lỗi kỳ vọng).

| Kiểu chọn | Công thức | Ví dụ |
|---|---|---|
| Không thứ tự, không lặp | $\binom{n}{k}=\frac{n!}{k!(n-k)!}$ | $\binom{10}{3}=120$ (A01) |
| Có thứ tự, không lặp | $A_n^k=\frac{n!}{(n-k)!}$ | $A_{10}^3=720$ |
| Có thứ tự, có lặp | $n^k$ | $8^3=512$ |

- B01 (birthday-style): 3 request vào 3 worker khác nhau $=\frac{8\cdot7\cdot6}{8^3}=\frac{336}{512}\approx0{,}656$.
- C01 (siêu bội, không hoàn lại): $1-\binom{16}{5}/\binom{20}{5}=1-4368/15504\approx0{,}718$; nhị thức $1-0{,}8^5$ là sai (coi có hoàn lại).

## 3. Kỳ vọng & phương sai

$$E[X]=\sum_x x\,P(X=x),\quad \mathrm{Var}(X)=E[X^2]-(E[X])^2$$

- A05: $0\cdot0{,}7+1\cdot0{,}2+2\cdot0{,}1=0{,}4$ ($E[X^2]=0{,}6$ là nhiễu).
- **Luôn đúng**: $E[X+Y]=E[X]+E[Y]$, $E[aX+b]=aE[X]+b$.
- **Chỉ đúng khi không tương quan**: $\mathrm{Var}(X+Y)=\mathrm{Var}X+\mathrm{Var}Y$, $E[XY]=E[X]E[Y]$.
- $\mathrm{Var}(X\pm Y)=\mathrm{Var}X+\mathrm{Var}Y\pm2\,\mathrm{Cov}(X,Y)$; $\mathrm{Var}(aX)=a^2\mathrm{Var}X$.
- B06: $\mathrm{Var}(Y-\hat Y)=9+4-2\cdot3=7$.
- B05 (độc lập): SD tổng $=\sqrt{3^2+4^2+12^2}=\sqrt{169}=13$ ms (**cộng phương sai, không cộng SD**).
- B07: $\mathrm{SD}(\bar X)=\sigma/\sqrt n=40/10=4$ ms ($\mathrm{Var}(\bar X)=\sigma^2/n=16$).
- **Biến chỉ báo** (D02): $I_j=1$ nếu worker $j$ rảnh, $P=(4/5)^{10}$ → $E[\#\text{rảnh}]=5\cdot0{,}8^{10}\approx0{,}537$ (không cần độc lập).
- **Retry tối đa 3 lần** (C05), $p=0{,}8$: $E[N]=1+P(\text{lần1 fail})+P(\text{2 lần đầu fail})=1+0{,}2+0{,}04=1{,}24$.

## 4. Các phân phối

| Phân phối | PMF/PDF | $E$ | $\mathrm{Var}$ |
|---|---|---|---|
| $\mathrm{Bin}(n,p)$ | $\binom{n}{k}p^k(1-p)^{n-k}$ | $np$ | $np(1-p)$ |
| $\mathrm{Poisson}(\lambda)$ | $e^{-\lambda}\lambda^k/k!$ | $\lambda$ | $\lambda$ |
| Siêu bội (rút không hoàn lại) | $\binom{K}{k}\binom{N-K}{n-k}/\binom{N}{n}$ | $nK/N$ | |
| Mũ (thời gian chờ) | $\lambda e^{-\lambda t}$ | $1/\lambda$ | $1/\lambda^2$ |
| $N(\mu,\sigma^2)$ | $z=(x-\mu)/\sigma$ | $\mu$ | $\sigma^2$ |

- B08: $\binom{10}{3}0{,}3^3 0{,}7^7=120\cdot0{,}027\cdot0{,}0824\approx0{,}267$.
- A07: Poisson $\lambda=4$ → Var $=4$ (SD $=2$, $1/\lambda=0{,}25$ là thời gian chờ).
- B10: $P(X\le2)=e^{-3}(1+3+4{,}5)\approx0{,}423$.
- C08: 6 lỗi/giờ → 10 phút: $\lambda=6\cdot\frac{10}{60}=1$, $P(0)=e^{-1}\approx0{,}368$.
- **Chuẩn**: $\Phi(1)=0{,}8413$, $\Phi(1{,}645)=0{,}95$, $\Phi(1{,}96)=0{,}975$, $\Phi(2)=0{,}9772$, $\Phi(2{,}326)=0{,}99$, $\Phi(3)=0{,}99865$.
- B09: $N(200,25^2)$, $P(X>250)=P(Z>2)\approx2{,}3\%$ (một đuôi).
- C07: P95 $=\mu+1{,}645\sigma=120+1{,}645\cdot15\approx144{,}7$ ms.
- **Xấp xỉ chuẩn**: Bin → $N(np,\,np(1-p))$; Poisson → $N(\lambda,\lambda)$. C09: $\mu=90$, $\sigma=\sqrt{81}=9$ → $90\pm1{,}96\cdot9\approx[72;108]$.

```python
from scipy.stats import binom, poisson, norm
binom.pmf(3, 10, 0.3)        # 0.267
poisson.cdf(2, 3)            # 0.423
norm.sf(250, 200, 25)        # 0.0228   (sf = 1 - cdf)
norm.ppf(0.95, 120, 15)      # 144.67
```

## 5. Xác suất toàn phần, Bayes, base rate

$$P(B)=\sum_i P(B\mid A_i)P(A_i),\qquad P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}$$

- B02: $0{,}5\cdot0{,}01+0{,}3\cdot0{,}02+0{,}2\cdot0{,}05=0{,}021$ (trung bình **có trọng số**).
- B03: $P(\text{spam}\mid W)=\frac{0{,}2\cdot0{,}4}{0{,}2\cdot0{,}4+0{,}8\cdot0{,}05}=\frac{0{,}08}{0{,}12}\approx0{,}667$.
- **Precision** với prevalence $\pi$, recall $r$, FPR $f$: $\ \mathrm{Prec}=\dfrac{\pi r}{\pi r+(1-\pi)f}$.
  - B04: $\pi=1\%$ → 31%; $\pi=0{,}1\%$ → 4,3%. Lớp càng hiếm, FP càng lấn át.
- Hai tầng **độc lập có điều kiện** (C02): nhân likelihood: $\frac{0{,}002\cdot0{,}99\cdot0{,}9}{0{,}002\cdot0{,}99\cdot0{,}9+0{,}998\cdot0{,}01\cdot0{,}02}\approx0{,}899$.
- **Specificity tối thiểu** (D01): $\mathrm{Prec}\ge0{,}5$ ⇔ $\pi r\ge(1-\pi)f$ ⇒ $f\le\frac{0{,}005\cdot0{,}9}{0{,}995}=0{,}00452$ ⇒ spec $\ge0{,}9955$.
- **Ngưỡng theo chi phí** (C06): review khi $0{,}8\cdot p\cdot2000>20$ ⇔ $p>0{,}0125$.

## 6. Nghịch lý Simpson (C03)

Tỉ lệ gộp $=\sum_d P(\text{mua}\mid d)P(d)$ — trọng số khác nhau giữa A và B:
A: $0{,}2\cdot15\%+0{,}8\cdot20\%=19\%$; B: $0{,}7\cdot17{,}1\%+0{,}3\cdot22\%=18{,}6\%$. B thắng ở **cả hai** nhóm nhưng thua ở số gộp vì dồn nhiều mobile (tỉ lệ nền thấp). → So theo từng nhóm, kiểm tra lại randomization.

## 7. Bài tự luận — lời giải mẫu

**C04 – mô hình gian lận đăng ký** (40.000 lượt/ngày, $\pi=0{,}5\%$, recall 95%, FPR 2%)
- (a) Gian lận thật $=200$; TP $=190$; FP $=39.800\cdot0{,}02=796$; gắn cờ $\approx986$; precision $=190/986\approx19{,}3\%$ → ~4/5 tài khoản bị khoá là khách thật.
- (b) Accuracy bị chi phối bởi lớp đa số: "luôn đoán hợp lệ" đạt $99{,}5\%$ > 98% nhưng recall $=0$. Báo cáo precision/recall, số FP tuyệt đối, PR-AUC thay accuracy.
- (c) $\pi=5\%$: $\frac{0{,}05\cdot0{,}95}{0{,}05\cdot0{,}95+0{,}95\cdot0{,}02}=\frac{0{,}0475}{0{,}0665}\approx71{,}4\%$. Biện pháp khác: nâng ngưỡng điểm / xác minh nhẹ (OTP, step-up) thay vì khoá → rủi ro: giảm recall, lọt gian lận, thêm ma sát. (Rủi ro của lọc theo IP: bỏ sót gian lận ngoài dải IP, kẻ gian đổi IP, thiên lệch nhóm người dùng.)

**D03 – capacity planning inference** (Poisson $\lambda=120$/phút, 25 req/phút/replica, latency $N(300,40^2)$, lỗi 0,1%)
- (a) $X\approx N(120,120)$, $\sigma\approx10{,}95$; $P(X>C)\le1\%$ ⇒ $C\ge120+2{,}326\cdot10{,}95\approx145{,}5$ → $C=146$; replica $=\lceil146/25\rceil=6$ (5 replica = 125 không đủ).
- (b) Tổng 5 lần: $\mu=1500$ ms, $\sigma=40\sqrt5\approx89{,}4$ ms; $P(S>1700)=P(Z>2{,}236)\approx0{,}0127$ (~1,3%). SD **không** phải $5\cdot40=200$.
- (c) Số lỗi $\sim\mathrm{Bin}(7200;0{,}001)\approx\mathrm{Poisson}(\lambda=7{,}2)$; $P(X\ge15)=1-P(X\le14)\approx0{,}0073$ (~0,7%). Giả định có thể sai: các lần lỗi không độc lập (sự cố dồn cụm, outage), tỉ lệ lỗi/lưu lượng không hằng theo thời gian (Poisson thuần nhất), latency có đuôi nặng (không chuẩn), các lần gọi trong agent tương quan.

## 8. Bẫy thường gặp

- **Xung khắc ≠ độc lập**: xung khắc với xác suất dương ⇒ phụ thuộc mạnh ($P(B\mid A)=0$).
- **"Ít nhất một"** mà cộng/nhân $n\cdot p$ → ra số kỳ vọng (có thể ≥1). Tính "đúng một" là bỏ sót 2, 3,… lần.
- **Hợp** quên trừ giao; hoặc tự giả định độc lập khi đề đã cho giao.
- **Có điều kiện**: nhân thay vì chia; dùng giao thay cho có điều kiện.
- **Tổ hợp vs chỉnh hợp**: bộ đặc trưng không thứ tự → chia $k!$; tử & mẫu phải cùng một cách đếm.
- **$E[X^2]$ hay trung bình cộng giá trị** thay cho $E[X]$ có trọng số.
- **$\mathrm{Var}(2X)=4\mathrm{Var}X$**, không phải 2; **hiệu** vẫn cộng phương sai, hạng Cov mang dấu trừ.
- **Cộng SD** thay vì cộng phương sai; quên lấy căn (ra ms²).
- **SD trung bình** chia $\sqrt n$, không chia $n$.
- **Poisson**: Var $=\lambda$ (không $\lambda^2$); $1/\lambda$ là thời gian chờ; không đổi $\lambda$ theo khoảng thời gian.
- **Hai đuôi vs một đuôi**: $P(|Z|>2)\approx4{,}6\%$ khi đề chỉ hỏi phía chậm (2,3%).
- **$N(\mu,25^2)$**: $\sigma=25$, không phải 625 hay 50.
- **Rút không hoàn lại** dùng siêu bội, không dùng nhị thức.
- **Precision ≠ recall**, và precision **phụ thuộc tỉ lệ nền** dù recall/FPR không đổi.
- **Accuracy cao** khi lớp hiếm vô nghĩa (luôn đoán âm đạt 99,5%).
- **Simpson**: B tốt hơn từng nhóm nhưng thua gộp không phải lỗi dữ liệu.
- **Ngưỡng 0,5** mặc định bỏ qua chi phí bất đối xứng; quên hệ số chặn 80% (chia, không nhân).
- **Tuyến tính kỳ vọng** không cần độc lập; "10 job > 5 worker nên không ai rảnh" sai.
- **Khoảng 95%**: $1{,}96\sigma$ (không $1\sigma$ = 68%), $\sigma$ của **tổng** $\sqrt{np(1-p)}$.

## 9. Tra nhanh

| Gặp tình huống | Dùng |
|---|---|
| Ít nhất một lần xảy ra trong $n$ lần độc lập | $1-(1-p)^n$ |
| A hoặc B | $P(A)+P(B)-P(A\cap B)$ |
| Biết B, hỏi A | $P(A\cap B)/P(B)$ |
| Nhiều nguồn/vùng, hỏi tỉ lệ chung | Xác suất toàn phần (trọng số) |
| Biết kết quả, hỏi nguyên nhân | Bayes |
| Đúng $k$ thành công trong $n$ lần độc lập | Nhị thức |
| Rút $n$ phần tử khác nhau từ lô hữu hạn | Siêu bội / $\binom{}{}$ |
| Số sự kiện trong khoảng thời gian | Poisson, $\lambda\propto$ độ dài khoảng |
| Không có sự kiện trong khoảng $t$ | $e^{-\lambda t}$ |
| Tỉ lệ vượt ngưỡng của biến chuẩn | $z=(x-\mu)/\sigma$, $1-\Phi(z)$ |
| Phân vị / SLO P95, P99 | $\mu+1{,}645\sigma$ / $\mu+2{,}326\sigma$ |
| SD tổng các bước độc lập | $\sqrt{\sum\sigma_i^2}$ |
| SD trung bình $n$ mẫu | $\sigma/\sqrt n$ |
| Var hiệu có tương quan | $\mathrm{Var}X+\mathrm{Var}Y-2\mathrm{Cov}$ |
| Precision ở tỉ lệ nền khác | $\pi r/(\pi r+(1-\pi)f)$ |
| Nhiều bộ lọc độc lập có điều kiện | Nhân các likelihood rồi Bayes |
| Ngưỡng hành động theo chi phí | lợi ích kỳ vọng > chi phí ⇒ $p>\text{cost}/(\text{hiệu quả}\cdot\text{thiệt hại})$ |
| Đếm "bao nhiêu cái thỏa" (kỳ vọng) | Biến chỉ báo + tuyến tính |
| Nhị thức/Poisson lớn | Xấp xỉ chuẩn $N(np,np(1-p))$ / $N(\lambda,\lambda)$ |
| Capacity với xác suất quá tải ≤ α | $C=\lceil\lambda+z_{1-\alpha}\sqrt\lambda\rceil$ |
| Tỉ lệ gộp mâu thuẫn tỉ lệ nhóm | Simpson → so từng nhóm |

## 10. Đáp án nhanh

✓ = đáp án chính thức từ PDF · ✎ = tự giải (đã kiểm bằng Python)

| Câu | Đáp án | Câu | Đáp án |
|---|---|---|---|
| A04 | B ✓ | B07 | B (4 ms) ✓ |
| A08 | 0,636 ✓ | B03 | 0,667 ✎ |
| A07 | D (4) ✓ | C07 | 144,7 ms ✎ |
| A01 | D (120) ✓ | C08 | D ($e^{-1}\approx0{,}368$) ✓ |
| A05 | C (0,4) ✓ | C03 | A (Simpson) ✓ |
| A03 | D (0,15) ✓ | C04 | 986 cờ, 190 thật, prec ≈19,3%; acc luôn-hợp-lệ 99,5%; prec IP rủi ro ≈71,4% ✎ |
| A02 | D (0,4) ✓ | C01 | A (≈0,718) ✓ |
| A06 | B ✓ | C05 | 1,24 ✎ |
| B09 | A (≈2,3%) ✓ | C02 | 0,899 ✎ |
| B01 | B (336/512) ✓ | C09 | C (72–108) ✓ |
| B06 | C (7) ✓ | C06 | C (0,0125) ✓ |
| B08 | D (≈0,267) ✓ | D01 | 0,9955 ✎ |
| B10 | 0,423 ✎ | D03 | C=146, 6 replica; 1500 ms, σ≈89,4, P≈0,013; Poisson(7,2), P(≥15)≈0,007 ✎ |
| B04 | A (31% → 4%) ✓ | D02 | D ($5\cdot0{,}8^{10}\approx0{,}537$) ✓ |
| B02 | B (0,021) ✓ | | |
| B05 | 13 ms ✎ | | |
