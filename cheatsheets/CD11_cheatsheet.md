# CĐ-11 · MLOps và vận hành mô hình — Cheatsheet

> 30 câu: A (Nhận biết, 1–8), B (Thông hiểu, 9–18), C (Vận dụng, 19–27), D (Vận dụng cao, 28–30). Đáp án ở mục cuối.

## 1. Ý tưởng chính được kiểm tra

| Ý tưởng | Câu |
|---|---|
| p95 = 95% request có latency ≤ giá trị đó; nearest-rank: phần tử thứ $\lceil 0.95n \rceil$ | A01, B01 |
| Chi phí/request = giá giờ ÷ (throughput × 3600); batch tăng throughput → giảm chi phí/request | A07, B08, C07 |
| Chi phí LLM API = token in × giá in + token out × giá out; so sánh với tự host → hoàn vốn | B03, D01 |
| Data drift (P(X) đổi) vs concept drift (P(Y\|X) đổi) vs label shift vs training-serving skew | A05, C06 |
| Giám sát khi nhãn trễ: drift input + phân phối điểm dự đoán + data quality; PSI | B07, C05 |
| Tái lập: DVC hash / snapshot dữ liệu bất biến; pin image bằng digest; thuật toán tất định | A03, A08, B05, C09 |
| Dynamic batching; throughput = b / t(b); prefill vs decode (TTFT) | A02, C02, B02 |
| Shadow / canary / blue-green / A/B; canary cần đủ mẫu (rule of three) | A06, B06, D02 |
| CACE (Sculley 2015); version hoá feature để rollback được | A04, B10 |
| Eval gate: golden set bị overfit → holdout ẩn, làm mới từ log | B04, D03 |
| Docker layer cache: COPY requirements + pip install trước COPY . . | B09 |
| Định luật Little, autoscaling + cold start, cache hit, bộ nhớ INT4 | C01, C08, C04, C03 |

## 2. Latency & percentiles

- **p95 = 420 ms**: ~95% request có latency **≤ 420 ms**, 5% chậm hơn. Không phải "95% thời gian", không phải trung bình.
- **Nearest-rank**: sắp xếp tăng dần, lấy phần tử thứ $k=\lceil 0.95\,n\rceil$. Với $n=24$: $k=\lceil 22.8\rceil=23$ → giá trị lớn thứ 2 (ví dụ đề: **702 ms**). Đuôi (tail) bị chi phối bởi vài request chậm.
- **TTFT** (time to first token) = thời gian **prefill**, xử lý toàn bộ prompt → tăng gần tuyến tính theo độ dài context. **Decode** sinh từng token nhờ KV cache → token/s gần như không đổi khi prompt dài hơn.

## 3. Throughput, batching, chi phí

| Công thức | Ví dụ trong đề |
|---|---|
| Chi phí / 1.000 req $= \dfrac{\text{giá/giờ}}{\text{req/s}\times 3600}\times 1000$ | 2,40 USD/h, 20 req/s → **0,0333 USD** |
| Throughput $= \dfrac{b}{t(b)}$ | $t(b)=20+5b$ ms, $b=16$ → 100 ms → **160 req/s** |
| Batch lớn: chi phí/req $\propto 1/\text{throughput}$ | 50→200 req/s ⇒ chi phí **còn 1/4** (p95 tăng vẫn OK nếu trong SLO) |
| Chọn GPU: đủ công suất $\sum \text{throughput}\ge$ tải, rồi min giá/giờ | 50 req/s: 2×X (60 req/s) = **2,40** < 1×Y = 3,60 |
| LLM API $= N\,(T_{in}p_{in}+T_{out}p_{out})$ | 1.000 × (1500×0,5 + 300×2)/1e6 = **1,35 USD** |
| Hoàn vốn $= \dfrac{\text{chi phí chuyển đổi}}{\text{API/tháng} - \text{tự host/tháng}}$ | API 7.040; host 2×1,8×730+1.500 = 4.128 → 12.000/2.912 = **4,1 tháng** |

- **Dynamic batching** (Triton): gom request đến gần nhau trong cửa sổ chờ ngắn (vài ms) thành một batch → tăng throughput GPU, đổi lại tăng nhẹ latency.
- **Cache phía trước model**: $E[\text{lat}] = t_{lookup} + (1-h)\,t_{model}$ = 15 + 0,65×900 = **600,0 ms** (mọi request đều tốn lookup).
- **Định luật Little**: $L=\lambda W$. Số request đồng thời = 12×2,5 = 30; mỗi replica dùng được 8×0,8 = 6,4 slot → $\lceil 30/6.4\rceil$ = **5 replica**.
- **Autoscaling**: GPU-giờ cố định 8×24 = 192; theo tải 14×8 + 10×1 = 122 → tiết kiệm **≈36%**. Cold start 3 phút ⇒ phải **scale-out trước** giờ cao điểm (scheduled / predictive scaling).
- **Bộ nhớ trọng số lượng tử hoá**: $N\cdot\frac{bits}{8} + \frac{N}{g}\cdot 2$ byte. 34e9×0,5 = 17 GB + (34e9/128)×2 = 0,53 GB → **17,53 GB** (1 GB = $10^9$ byte).

## 4. Drift & giám sát

| Loại | Thay đổi | Ví dụ |
|---|---|---|
| Data drift / covariate shift | $P(X)$ đổi, $P(Y\mid X)$ giữ nguyên | tỉ lệ email tiếng Anh 10%→45%, tiêu chí spam không đổi |
| Concept drift | $P(Y\mid X)$ đổi | định nghĩa gian lận/spam thay đổi |
| Label shift | $P(Y)$ đổi, $P(X\mid Y)$ giữ nguyên | tỉ lệ spam tăng |
| Training-serving skew | pipeline feature lúc serve ≠ lúc train | serve viết lại bằng Java, quên lowercase |

- **PSI** $=\sum_i (a_i-e_i)\ln\frac{a_i}{e_i}$ (ln tự nhiên). Đề: **0,136**. Ngưỡng hay dùng: <0,1 ổn; 0,1–0,25 cần theo dõi; >0,25 drift lớn.
- **Nhãn trễ (90 ngày)**: theo dõi drift của feature đầu vào + **phân phối điểm dự đoán** + kiểm tra data quality (null, range, schema). Không dùng dự đoán làm nhãn.
- **Chống skew**: viết test đối chiếu trong CI, đưa cùng request mẫu qua preprocessing của train và của serve, assert feature khớp nhau (hoặc dùng chung code/feature store).

## 5. Tái lập (reproducibility)

- **DVC**: Git chỉ chứa `.dvc` / `dvc.lock` (hash nội dung); dữ liệu thật nằm ở remote storage (S3, GCS...).
- **Registry**: mỗi model version phải trỏ tới **snapshot dữ liệu bất biến** (DVC hash, time-travel của Delta/Iceberg/BigQuery), kèm commit code và hyperparams. Tên bảng bị ghi đè hằng ngày thì không tái lập được.
- **Docker**: `FROM python:3.11-slim@sha256:<digest>`. Tag là mutable, chỉ digest mới bất biến.
- **GPU nondeterminism**: đặt seed cho mọi nguồn ngẫu nhiên và bật thuật toán tất định:
```python
import random, numpy as np, torch
random.seed(0); np.random.seed(0); torch.manual_seed(0); torch.cuda.manual_seed_all(0)
torch.use_deterministic_algorithms(True)
torch.backends.cudnn.benchmark = False
# env: CUBLAS_WORKSPACE_CONFIG=:4096:8
```
- **Docker layer cache**: lớp nào đổi thì mọi lớp sau đó build lại.
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt   # được cache khi chỉ sửa code
COPY . .
CMD ["python", "serve.py"]
```

## 6. Chiến lược triển khai

| Chiến lược | Bản chất |
|---|---|
| **Shadow** | model mới nhận **bản sao** request thật, chỉ ghi log; user vẫn nhận kết quả model cũ |
| **Canary** | model mới phục vụ thật một phần nhỏ (1–5%), tăng dần nếu ổn |
| **Blue-green** | dựng môi trường song song, chuyển 100% một lần, giữ môi trường cũ để quay lại |
| **A/B test** | chia ngẫu nhiên hai nhóm user để so sánh chỉ số kinh doanh |

- **Rule of three**: 0 lỗi trên $n$ request ⇒ cận trên CI 95% của tỉ lệ lỗi ≈ $3/n$. Với $n=150$: ≈ **2%**, lớn hơn 0,4% của model cũ ⇒ **chưa đủ** bằng chứng. Số request tối thiểu để phân biệt mức $p$: $n \gtrsim 3/p$ (0,4% → ~750 request mới có thể thấy 0 lỗi có ý nghĩa).
- **CACE** (Sculley 2015, *Hidden Technical Debt*): feature bị rối vào nhau, đổi một feature/tham số là hành vi cả model đổi. Model không phải module độc lập.
- **Version hoá feature**: không đổi định nghĩa feature tại chỗ (giữ tên cột). Tạo `so_giao_dich_7d` mới, giữ bản cũ đến khi hết khả năng rollback. Nếu không, rollback model v4 vẫn nhận feature mới và dự đoán sai.

### Khung trả lời D02 (kế hoạch canary)
1. **Giai đoạn**: shadow → 1% → 5% → 25% → 50% → 100%. Mỗi giai đoạn chạy tối thiểu N request (ví dụ ≥ 3/0,002 = 1.500 cho lỗi 5xx, nhiều hơn nữa cho add-to-cart) và ≥ 24 h để phủ chu kỳ ngày. Chọn user bằng **hash(user_id)** cố định (sticky), loại nhân viên/bot, phân tầng theo nền tảng.
2. **Chỉ số + rollback** (so với nhóm đối chứng v6 **cùng thời điểm**): p95 > 250 ms; 5xx > 0,2% + biên (ví dụ > 0,4%); add-to-cart giảm có ý nghĩa thống kê (ví dụ < 5,8%); tỉ lệ null/lỗi của 2 feature mới; drift điểm dự đoán.
3. **Cơ chế**: feature flag / router weight về v6 trong vài phút (không cần redeploy), giữ v6 warm. 2 feature mới phải có giá trị mặc định/fallback, v6 không phụ thuộc chúng. Sau rollback: postmortem, lưu log canary, sửa lỗi rồi chạy lại từ shadow.

## 7. Eval gate cho LLM/RAG (B04, D03)

- **Overfit golden set**: chọn prompt qua 35 vòng trên cùng 150 mẫu ⇒ điểm CI bị thổi phồng (91% CI → 84% trên dữ liệu thật). Sửa: **holdout ẩn** (chỉ chạy ở gate, không dùng để tune) + làm mới định kỳ từ log production. Nhân bản mẫu không thêm thông tin.
- **Dữ liệu eval**: câu hỏi thật từ log + case khó/adversarial + câu không có trong tài liệu (phải từ chối); nhãn do chuyên gia gán (đáp án + tài liệu nguồn); version hoá bằng DVC/hash; chia dev/holdout.
- **Luật pass/fail**: ngưỡng tuyệt đối (ví dụ accuracy ≥ 90%, faithfulness ≥ x) **và** không giảm quá δ so với baseline nhánh main; phân khúc quan trọng (thời hạn bảo hành) có ngưỡng riêng, ví dụ 100% hoặc không có regression nào.
- **Tính ngẫu nhiên**: temperature = 0, cố định seed/version model, chạy k lần lấy trung bình/CI, dùng LLM-judge có rubric + exact match cho các số. Giới hạn thời gian/chi phí (tập nhỏ mỗi PR, tập đầy đủ khi merge/nightly). Báo cáo trên PR: điểm tổng và theo phân khúc, delta so với main, các mẫu fail, chi phí.

## 8. Bẫy thường gặp

| Phương án nhiễu | Vì sao sai |
|---|---|
| "Chỉ 5% ≤ 420 ms" / "95% thời gian" / "trung bình của 95%" | p95 là ngưỡng mà 95% **request** nằm dưới |
| Coi P(X) đổi là concept drift | Concept drift là P(Y\|X) đổi |
| Commit toàn bộ dữ liệu vào Git; lưu câu SQL | Git không hợp với file lớn; dữ liệu nguồn thay đổi nên SQL chạy lại ra kết quả khác |
| Dynamic batching = cache kết quả / chia request ra nhiều GPU / chỉnh batch khi train | Nó gom nhiều request thành batch lúc **inference** |
| `python:3.11-slim` "đã cố định" | Tag có thể bị đẩy lại; chỉ `@sha256` mới bất biến |
| TTFT tăng vì mạng/KV cache tắt/sinh nhiều token hơn | Nguyên nhân là prefill tỉ lệ với độ dài prompt |
| "n = 150 > 30 nên theo CLT là đủ"; "0% < 0,4% nên tốt hơn" | Sự kiện hiếm: cận trên ≈ 3/n = 2% |
| Lưu trọng số / số dòng / seed là đủ tái lập | Cần snapshot dữ liệu bất biến |
| Nâng ngưỡng trên cùng golden set; chạy lại LLM; nhân bản mẫu | Không chữa được overfit vào tập eval |
| Gộp RUN; `--no-cache-dir`; base image đầy đủ | Vấn đề nằm ở **thứ tự** COPY làm mất cache |
| Rollback lỗi do concept drift/checksum/quá tải | Do đổi định nghĩa feature tại chỗ |
| Dùng dự đoán làm nhãn; AUC hằng ngày; chỉ theo dõi HTTP | Chưa có nhãn thật; vẫn giám sát được drift/score |
| Chi phí tăng 16× hoặc theo p95; "không đổi" | Chi phí/req = giá giờ / throughput |
| 57% / 87,5% / "autoscaler tức thì" | Tính GPU-giờ thật: (192−122)/192; cold start cần scale trước |
| Nâng AUC offline; nhiều seed; unit test chạy được | Không bắt được skew giữa train và serve |
| Seed dài hơn; data leak; rebuild image "mới nhất" | Do kernel GPU không tất định; image mới nhất càng làm khó tái lập |
| 1 GPU X (30 < 50); "Y chỉ 1,50" | Không đủ công suất / tính theo phần nhỏ trong khi đề nói tính nguyên GPU |

## 9. Tra nhanh

| Nếu gặp... | Dùng... |
|---|---|
| Tính p-percentile nearest-rank | sort, lấy phần tử $\lceil p\,n\rceil$ |
| Chi phí GPU / N request | giá/h ÷ (req/s·3600) × N |
| Throughput khi batch | $b/t(b)$ (đổi ms sang s) |
| Số replica | Little: $L=\lambda W$; replica = $\lceil L/(\text{slot}\times\text{util})\rceil$ |
| Latency trung bình có cache | $t_{lookup}+(1-h)t_{miss}$ |
| Bộ nhớ INT-k + scale | $N k/8 + (N/g)\cdot 2$ byte |
| Hoàn vốn tự host | one-off ÷ (tiết kiệm/tháng) |
| Đo drift 1 feature | PSI $\sum(a-e)\ln(a/e)$ |
| Nhãn đến trễ | drift input + score + data quality |
| 0 lỗi trên n mẫu | cận trên 95% ≈ 3/n |
| Thử model không ảnh hưởng user | shadow |
| Đưa ra thật từng phần | canary (hash user_id, so với control cùng lúc) |
| Prod sai mà offline tốt | nghi training-serving skew → test parity |
| Rollback model vẫn sai | feature bị đổi định nghĩa → version hoá feature |
| Tái lập dữ liệu / image / GPU | DVC hoặc time-travel / `@sha256` / deterministic algorithms + seed |
| Build Docker chậm | COPY requirements → pip install → COPY . . |
| Điểm CI cao, prod thấp (LLM) | holdout ẩn + làm mới từ log |

## 10. Đáp án

| Câu | Đáp án | Câu | Đáp án | Câu | Đáp án |
|---|---|---|---|---|---|
| 1 (A07) | 0,0333 | 11 (B05) | B | 21 (C07) | A |
| 2 (A01) | D | 12 (B04) | B | 22 (C02) | 160 |
| 3 (A05) | B | 13 (B09) | D | 23 (C04) | 600,0 |
| 4 (A03) | A | 14 (B10) | A | 24 (C06) | A |
| 5 (A02) | B | 15 (B07) | C | 25 (C05) | 0,136 |
| 6 (A06) | C | 16 (B01) | 702 | 26 (C09) | D |
| 7 (A04) | B | 17 (B03) | B | 27 (C01) | 5 |
| 8 (A08) | D | 18 (B08) | D | 28, 29 | Tự luận (mục 6, 7) |
| 9 (B02) | A | 19 (C03) | 17,53 | 30 (D01) | 4,1 |
| 10 (B06) | B | 20 (C08) | B | | |
