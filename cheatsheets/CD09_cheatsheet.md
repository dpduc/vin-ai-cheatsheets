# CĐ-09 · Học máy và đánh giá mô hình — Cheatsheet

> 30 câu (A: Nhận biết 1–8 · B: Thông hiểu 9–18 · C: Vận dụng 19–27 · D: Vận dụng cao 28–30). Mã câu dạng T09-xxx.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | k-fold CV: train $k$ lần, mỗi lần trên $\frac{k-1}{k}N$ mẫu | A05 |
| 2 | Validation để chọn siêu tham số/mô hình; test chỉ dùng 1 lần cuối | A01, B07 |
| 3 | kNN: cần chuẩn hoá, k nhỏ thì overfit, dự đoán chậm theo N; kNN có trọng số $1/d$ | A03, C04 |
| 4 | RMSE, F1, Macro-F1, accuracy, ROC-AUC (đếm cặp) | A07, B08, C06, C07 |
| 5 | Biến danh mục không thứ tự → one-hot | A08 |
| 6 | Chẩn đoán bias/variance qua lỗi người – train – val; phân rã Bias²+Var+σ² | A04, C09 |
| 7 | Recall vs precision; diễn giải AUC | A06, B09 |
| 8 | Logistic: ranh giới $w^\top x+b=0$; hệ số = log-odds, $e^{\beta}$ = tỉ số odds | A02, C03 |
| 9 | Random forest giảm variance (bagging + đặc trưng ngẫu nhiên); ridge co hệ số | B03, B06 |
| 10 | k-means 1 vòng; entropy & information gain | B05, B04 |
| 11 | Rò rỉ dữ liệu: chia theo nhóm, theo thời gian, oversample sau khi chia, chọn đặc trưng trong fold, target encoding out-of-fold, cột chứa thông tin tương lai | B02, B01, B10, C02, C08, C01 |
| 12 | Quy tắc một sai số chuẩn (1-SE) | C05 |
| 13 | Chọn ngưỡng theo chi phí/lợi nhuận; precision phụ thuộc prevalence | D01, D02, D03 |

## 2. Chia dữ liệu & Cross-validation

- **Train**: học trọng số. **Validation**: chọn siêu tham số, so sánh mô hình. **Test**: báo cáo cuối, chỉ dùng **1 lần**.
- **k-fold**: $N=1000, k=5$ → 5 lần train, mỗi lần 800 mẫu, chấm trên 200 mẫu.
- Điểm CV của cấu hình **tốt nhất trong nhiều cấu hình** bị **lạc quan** (do đã dùng để chọn) → cần test riêng hoặc **nested CV**. Tăng số fold không sửa được chuyện này.
- **Cách chia đúng theo cấu trúc dữ liệu**:
  - Nhiều mẫu cùng 1 thực thể (4 ảnh/sản phẩm, nhiều lượt/khách) → **GroupKFold / chia theo ID**.
  - Dự đoán tương lai → **chia theo thời gian** (train tháng cũ, test tháng mới nhất); CV dạng expanding window / `TimeSeriesSplit`.
  - Stratified chỉ giữ tỉ lệ nhãn, **không** xử lý được leakage theo nhóm/thời gian.
- **Mọi bước học từ dữ liệu** (chuẩn hoá, chọn đặc trưng, oversampling, target encoding) phải **fit trong từng fold train**, không fit trên toàn bộ.

```python
from sklearn.model_selection import GroupKFold, TimeSeriesSplit
from sklearn.pipeline import Pipeline   # bọc scaler/selector/model -> fit trong từng fold
```

**Quy tắc 1-SE** (C05): ngưỡng = $\min(\text{RMSE}_{CV}) + SE_{\min}$ = 344+10 = 354 → chọn mô hình **đơn giản nhất** có RMSE CV ≤ 354 → **độ sâu 4** (352).

## 3. Rò rỉ dữ liệu (data/label leakage)

| Tình huống | Vì sao rò rỉ | Sửa |
|---|---|---|
| Ảnh cùng sản phẩm ở cả train & test (97% → 82%) | Mô hình nhớ sản phẩm | Chia theo mã sản phẩm |
| Chia ngẫu nhiên dữ liệu theo thời gian (AUC 0,93 → 0,84) | Học từ tương lai, drift | Chia theo thời gian |
| Oversample **trước** khi chia (recall 0,96) | Bản sao cùng khách ở cả train & test | Chia trước, oversample **chỉ train** |
| Chọn 30/5000 đặc trưng trên toàn bộ 200 mẫu, nhãn xáo trộn vẫn AUC 0,81 | Bước chọn đã "nhìn" nhãn fold test; AUC thật ≈ 0,5 | Chọn đặc trưng **trong** từng fold |
| Target encoding `agent_id` trên toàn train (train 0,97 / test 0,79) | Mẫu tự mã hoá chính nhãn của nó, đại lý ít mẫu | **Out-of-fold** + **smoothing** về tỉ lệ chung |
| Cột cập nhật **sau** thời điểm dự đoán | Chứa thông tin nhãn | Loại bỏ / lấy snapshot tại thời điểm đặt |

Cột rò rỉ trong bài khách sạn (dự đoán lúc đặt): `reservation_status` (chính là nhãn), `reservation_status_date`, `assigned_room_type` (biết khi nhận phòng), `booking_changes` (cập nhật đến ngày nhận phòng → chỉ dùng giá trị tại lúc đặt). Hợp lệ: `lead_time`, `adr`, `deposit_type`, `previous_cancellations` (các lần **trước**).

Smoothing target encoding: $\hat{p}_j = \dfrac{n_j\bar{y}_j + m\,\bar{y}}{n_j + m}$.

## 4. Chỉ số đánh giá

| Chỉ số | Công thức |
|---|---|
| RMSE | $\sqrt{\frac1n\sum(y_i-\hat y_i)^2}$ |
| Precision | $\frac{TP}{TP+FP}$ — trong số **bị gắn cờ**, bao nhiêu đúng |
| Recall (TPR) | $\frac{TP}{TP+FN}$ — trong số **thực sự dương**, bao nhiêu bị bắt |
| FPR | $\frac{FP}{FP+TN}$ |
| F1 | $\frac{2PR}{P+R}=\frac{2TP}{2TP+FP+FN}$ |
| Macro-F1 | trung bình cộng F1 các lớp (lớp nhỏ ảnh hưởng ngang lớp lớn) |
| ROC-AUC | $P(\text{score}_{+}>\text{score}_{-})$; = (số cặp (+,−) xếp đúng + 0,5·số cặp hoà)/($n_+n_-$) |

Kết quả đã tính:
- A07 RMSE: sai số −1,2,0,3,0 → $\sqrt{14/5}$ = **1,67**.
- B08 F1: P = 45/60 = 0,75; R = 45/75 = 0,6 → **0,667**.
- C06: accuracy = 910/1000 = 0,91; F1 áo ≈ 0,958, giày = 110/170 ≈ 0,647, túi = 10/55 ≈ 0,182 → Macro-F1 ≈ **0,60** (đáp án 0,60 và 0,91). Accuracy cao do lớp áo áp đảo.
- C07 AUC: 4 dương × 4 âm = 16 cặp; đúng 4+3+2,5+2 = 11,5 → **0,719**.
- AUC 0,85 **không** phải accuracy, precision hay recall ở một ngưỡng nào cả; nó là xác suất xếp hạng đúng một cặp.

**Precision phụ thuộc prevalence** (D03): validation cân bằng 50/50 làm precision bị thổi phồng. Precision thực tế:
$$\text{Prec}=\frac{TPR\cdot\pi}{TPR\cdot\pi + FPR(1-\pi)}$$
Với 200.000 ảnh/ngày, $\pi$ = 0,5% → 1.000 vi phạm, 199.000 hợp lệ:

| Ngưỡng | TP | FP (hợp lệ bị gắn cờ) | Precision thật |
|---|---|---|---|
| 0,95 | 600 | 199 | 0,751 |
| 0,80 | 850 | 1.990 | 0,299 |
| 0,50 | 950 | 9.950 | 0,087 |

Chính sách hai ngưỡng: ≥0,95 gỡ tự động (199 FP ≤ 200); 0,80–0,95 đưa người duyệt (2.840 − 799 = **2.041** ≤ 3.000); <0,80 cho qua → lọt **150**/ngày, recall hệ thống **85%**. (Nếu hạ ngưỡng duyệt xuống 0,50 thì 10.101 ảnh, vượt năng lực duyệt.)

## 5. Bias – Variance & Regularization

- Lỗi người (≈ Bayes) 3%, train 27%, val 29% → khoảng cách train–người lớn = **bias cao (underfit)** → mô hình mạnh hơn, train lâu hơn. Train–val nhỏ **không** có nghĩa mô hình ổn.
- Train ≪ val (cây sâu: 100% vs 78%) = **variance cao**.
- Phân rã: $\mathbb E[(y-\hat f(x_0))^2]=\text{Bias}^2+\text{Var}+\sigma^2$, $\text{Bias}=\bar{\hat f}-f(x_0)$.
  - C09: trung bình dự đoán 3,0 → Bias² = 0,25; Var (chia 5) = 2,5/5 = 0,5; σ² = 0,25 → **1,00**.
- **Ridge (L2)** $\sum(y_i-w^\top x_i)^2+\lambda\|w\|_2^2$: tăng λ → hệ số **co về 0 (không đúng bằng 0)**, lỗi train tăng, bias↑ variance↓. Chỉ **Lasso (L1)** mới đưa hệ số về đúng 0.
- **Random forest** = bagging (bootstrap) + chọn ngẫu nhiên đặc trưng ở mỗi split → các cây ít tương quan → lấy trung bình giúp **giảm variance**. "Cây sau học phần dư của cây trước" là **boosting** (giảm bias).

## 6. Các mô hình cụ thể

**Logistic regression**: $P(y=1|x)=\sigma(w^\top x+b)$; $\sigma(z)\ge 0{,}5 \iff z\ge0$ → ranh giới $\{x: w^\top x+b=0\}$ là **siêu phẳng** (tuyến tính).
- Hệ số β: $x$ tăng 1 thì log-odds cộng thêm β, tức **odds nhân với $e^{\beta}$**. β = −0,69 → odds × 0,5 (không phải xác suất × 0,5, cũng không phải trừ 0,69).

**kNN**: không có bước tối ưu (lazy), dự đoán phải tính khoảng cách tới mọi mẫu (O(N)); đặc trưng có thang đo lớn lấn át → **chuẩn hoá**; k nhỏ → ranh giới gồ ghề, **overfit**.
- C04: q = (1,1); 3 láng giềng gần nhất là P1 (d = 0,5, bot), P5 (0,707, bot), P2 (1, người); w = 1/d = 2; 1,414; 1 → điểm bot = 3,414/4,414 = **0,77**.

**k-means** (1 vòng = gán điểm vào tâm gần nhất → tâm mới = trung bình các điểm của cụm): B05 μ1 = (1,1) nhận A, B, C; μ2 = (5,5) nhận D, E, F → $\mu_2^{(1)}=(4+7+8)/3$ = **6,33**.

**Cây quyết định**: $H=-\sum p\log_2p$; $IG=H(\text{cha})-\sum\frac{n_k}{n}H(k)$.
- B04: H(4/10) = 0,971; nhánh (4,1): 0,722; nhánh (0,5): 0 → IG = 0,971 − 0,5·0,722 = **0,610**.

**Mã hoá danh mục**: mã 0/1/2 cho biến không có thứ tự sẽ ép mô hình coi các giá trị có thứ tự và cách đều nhau → dùng **one-hot**. Biến có nhiều giá trị (high cardinality, ~3.000) → target encoding out-of-fold + smoothing.

## 7. Chọn ngưỡng theo chi phí / lợi nhuận

- Ngưỡng lý thuyết (xác suất đã hiệu chỉnh tốt): gắn cờ khi $p\,C_{FN} > (1-p)\,C_{FP}$, tức $p^*=\dfrac{C_{FP}}{C_{FP}+C_{FN}}$.
- **D02 (overbooking)**: $C_{FP}$ = 1,5tr, $C_{FN}$ = 0,6tr, 2.800 ca hủy/10.000. Chi phí = FP·1,5 + (2800−TP)·0,6 (triệu):
  - Không bán vượt: 2800·0,6 = **1.680**
  - 0,50: 1.350 + 420 = 1.770 (còn tệ hơn không làm gì)
  - 0,70: 450 + 720 = **1.170** ← chọn
  - 0,85: 120 + 1.080 = 1.200
  - $p^*$ = 1,5/2,1 ≈ **0,714**, khớp với ngưỡng 0,70.
  - Theo dõi sau triển khai: số ca phải chuyển khách (FP), tỉ lệ hủy thực tế, calibration (reliability curve, Brier score), drift, tổng chi phí thực.
- **D01 (ưu đãi churn)**: lợi nhuận = 0,4·TP·1tr − 0,05tr·(TP+FP): 0,7 → 95; 0,5 → **122,5**; 0,3 → 101; 0,2 → 60 → đáp án **122,5 triệu**.

## 8. Bẫy thường gặp

- "k-fold train trên toàn bộ N" hoặc "train trên 1 fold" → sai, mỗi lần train trên $(k-1)/k$.
- "Chọn cấu hình trên test rồi báo cáo luôn điểm test" → lạc quan.
- "k = 1 ranh giới mượt hơn" → ngược lại; "kNN huấn luyện bằng gradient descent" → sai.
- "Logistic tự học hệ số riêng cho từng kênh khi mã 0/1/2" → sai, chỉ có 1 hệ số. Chuẩn hoá cột 0/1/2 cũng không sửa được.
- "Train ≈ val nên mô hình ổn" → phải so với lỗi của người (bias).
- Recall bị nhầm với precision (trong số bị gắn cờ) hoặc specificity (người thật không bị gắn cờ).
- Ranh giới logistic là $\sigma(\cdot)=0$ hoặc $w^\top x+b=0{,}5$ hoặc "mặt cong chữ S" → sai, đúng là $w^\top x+b=0$.
- RF "giới hạn độ sâu 1" / "mỗi cây dùng toàn bộ dữ liệu và đặc trưng" / "học phần dư" → sai (phần dư là boosting).
- Ridge "đưa hệ số về đúng 0" (đó là Lasso); "lỗi train giảm khi λ tăng" → sai.
- AUC bị diễn giải thành accuracy@0,5 / precision / recall ở mọi ngưỡng → sai.
- "CV score bi quan vì chỉ học 80%" / "tăng lên 10 fold là hết chệch" → không giải quyết optimism do lựa chọn.
- Stratified / tăng tỉ lệ test / lặp nhiều seed → không sửa được leakage theo thời gian hoặc theo nhóm.
- Oversample cả test; "oversampling luôn làm tăng recall thật" → sai.
- Đổi sang random forest để "sửa" AUC bị rò rỉ do chọn đặc trưng → sai, lỗi nằm ở quy trình.
- Tính target encoding trên cả train + test → rò rỉ nặng hơn; tăng số cây không sửa được.
- Odds "giảm 0,69 đơn vị" / xác suất × $e^\beta$ → sai, odds × $e^\beta$.
- Precision đo trên validation cân bằng đem dùng cho vận hành → sai khi prevalence chỉ 0,5%.
- Ngưỡng 0,5 mặc định khi chi phí FP ≠ FN → phải tính chi phí, và luôn so với phương án "không làm gì".

## 9. Tra nhanh

| Nếu gặp… | Dùng… |
|---|---|
| Nhiều mẫu cùng 1 thực thể | GroupKFold, chia theo ID |
| Dự đoán tương lai, dữ liệu có thời gian | Chia theo thời gian, TimeSeriesSplit |
| Cần điểm không chệch sau khi tune | Test hold-out riêng hoặc nested CV |
| Mất cân bằng lớp | Oversample/SMOTE **chỉ trên train** (trong fold); đánh giá bằng PR/recall/F1 |
| p ≫ n, cần chọn đặc trưng | Chọn đặc trưng trong Pipeline, bên trong từng fold |
| Danh mục không thứ tự, ít giá trị | One-hot |
| Danh mục nhiều giá trị | Target encoding out-of-fold + smoothing |
| kNN / k-means / mô hình dùng khoảng cách | Chuẩn hoá đặc trưng |
| Train cao, val thấp (variance) | Regularization, RF/bagging, thêm dữ liệu, giảm độ sâu |
| Train lỗi ≫ lỗi người (bias) | Mô hình mạnh hơn, train lâu hơn, thêm đặc trưng |
| Nhiều mô hình CV gần bằng nhau | Quy tắc 1-SE → chọn mô hình đơn giản nhất |
| Chi phí FP ≠ FN | $p^*=C_{FP}/(C_{FP}+C_{FN})$, kiểm chứng bằng bảng chi phí |
| Lợi nhuận chiến dịch | $\text{benefit}\cdot TP - \text{cost}\cdot(TP+FP)$, quét các ngưỡng |
| Prevalence thực ≠ prevalence validation | Tính lại precision từ TPR, FPR và π |
| Năng lực duyệt có hạn + giới hạn FP tự động | Hai ngưỡng: tự động / người duyệt / cho qua |
| Lớp nhỏ quan trọng như lớp lớn | Macro-F1 thay cho accuracy |
| Cột cập nhật sau thời điểm dự đoán | Loại bỏ hoặc lấy snapshot tại thời điểm dự đoán |
