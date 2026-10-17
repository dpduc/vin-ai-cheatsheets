# CĐ-12 · Logic, đạo đức và pháp lý AI — Cheatsheet

> 30 câu: A (Nhận biết) 1–8 · B (Thông hiểu) 9–18 · C (Vận dụng) 19–27 · D (Vận dụng cao) 28–30.

## Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | Mệnh đề đảo/phản đảo; De Morgan; đếm bảng chân trị | A01, A02, B01, C01 |
| 2 | Lập luận hợp lệ (modus ponens, bắc cầu) và các lỗi suy luận | B03 |
| 3 | Phủ định lượng từ; thứ tự $\forall\exists$ và $\exists\forall$ | B02, C03 |
| 4 | Câu đố "chỉ một người nói thật" → thử từng trường hợp | C02 |
| 5 | Ngụy biện (ad hominem…); tương quan ≠ nhân quả; Simpson; base rate | A03, B04, C04, C05 |
| 6 | Dữ liệu nhạy cảm (Luật BVDLCN 2025, NĐ 356/2025); rút lại đồng ý; giới hạn mục đích | A05, B06, B05 |
| 7 | GDPR Art. 5 (tối thiểu hóa…), Art. 33–34 (báo sự cố trong 72h) | A04, C07 |
| 8 | EU AI Act: 4 mức rủi ro | A07, B09, D02, D03 |
| 9 | OWASP Top 10 for LLM 2025 (LLM04, LLM08…) | A08, B10, D03 |
| 10 | Tiêu chí công bằng (DP, EO, EOdds), disparate impact, quy tắc 4/5 | A06, C08, C09, D01, D02 |
| 11 | k-anonymity | C06 |
| 12 | Model card; automation bias / giám sát con người | B07, B08 |

---

## 1. Logic mệnh đề

**Kéo theo $p \to q$** (chỉ sai khi $p$ đúng, $q$ sai):
| Tên | Dạng | Tương đương với gốc? |
|---|---|---|
| Đảo (converse) | $q \to p$ | Không |
| Phản (inverse) | $\neg p \to \neg q$ | Không (nhưng tương đương với đảo) |
| Phản đảo (contrapositive) | $\neg q \to \neg p$ | **Có** |
| Phủ định | $p \land \neg q$ | là phủ định |

→ A01: đảo của "rò rỉ nhãn → điểm test cao" là **"điểm test cao → rò rỉ nhãn"** (D).

**De Morgan:** $\neg(a \lor b) \equiv \neg a \land \neg b$; $\neg(a \land b) \equiv \neg a \lor \neg b$.
```python
not (is_spam or is_bot)  ==  (not is_spam) and (not is_bot)   # A02 → C
```

**Đếm số tổ hợp làm quy tắc đúng:** đếm số trường hợp vi phạm rồi lấy tổng trừ đi.
- B01: $(p\lor q)\to\neg r$ ($p$ spam, $q$ bot, $r$ hiện quảng cáo) sai khi $(p\lor q)=1$ và $r=1$: 3 tổ hợp → $8-3=$ **5**.
- C01 (5 biến $a,o,l,m,e$, $2^5=32$): $(a\lor o)\land\neg l\land(a\to m)\land(\neg e\to a)$. $l=0$ cố định. $a=1$ thì $m=1$, $o$ và $e$ tự do: 4. $a=0$ thì $o=1$, $e=1$, $m$ tự do: 2. → **6**.
  Mẹo: khóa các biến bị ép giá trị trước, rồi tách trường hợp theo biến xuất hiện nhiều nhất.

## 2. Lập luận hợp lệ

| Dạng | Hợp lệ? |
|---|---|
| Modus ponens: $p\to q,\ p \vdash q$ | Có |
| Modus tollens: $p\to q,\ \neg q \vdash \neg p$ | Có |
| Bắc cầu: $p\to q,\ q\to r,\ p \vdash r$ | Có (B03-B) |
| Khẳng định hậu kiện: $p\to q,\ q \vdash p$ | **Không** (B03-A) |
| Phủ định tiền kiện: $p\to q,\ \neg p \vdash \neg q$ | **Không** (B03-D) |
| "Một số X là A; một số X là B ⇒ một số A là B" | **Không** (B03-C, hai tập có thể không giao nhau) |

## 3. Lượng từ

- $\neg\forall x\,P(x) \equiv \exists x\,\neg P(x)$; $\neg\exists x\,P(x) \equiv \forall x\,\neg P(x)$.
- B02: phủ định của "mọi mô hình đều được test trên ≥1 bộ OOD" là **"có ít nhất một mô hình không được test trên bộ OOD nào"** (A).
- $\exists r\,\forall x \Rightarrow \forall x\,\exists r$, nhưng chiều ngược lại **không** đúng.
- C03: "mọi nhân viên đều có ≥1 vai trò" ($\forall x\exists r$) **đúng**; "có một vai trò được gán cho tất cả" ($\exists r\forall x$) **sai** (analyst không có Chi) → **C**.

## 4. Câu đố nói thật / nói dối (C02)
Giả sử lần lượt từng người là thủ phạm rồi đếm số lời khai đúng. A sai thì chỉ có lời của B đúng (đúng 1 lời khai đúng ✓). B sai thì 2 lời đúng; C sai cũng 2 lời đúng. → **A**.

## 5. Ngụy biện và suy luận thống kê

| Ngụy biện | Nhận diện |
|---|---|
| Ad hominem | Tấn công người nói (thâm niên, tuổi…) thay vì lập luận (A03 → C) |
| Appeal to authority | "Chuyên gia X nói vậy nên đúng" |
| Straw man | Bóp méo lập luận đối phương rồi bác bỏ |
| Slippery slope | "Nếu A thì dần dần dẫn tới thảm họa Z" |

- **Selection bias / confounding** (B04): người *tự bật* nhắc học vốn có động lực cao hơn → không suy ra được nhân quả. Cần **A/B test ngẫu nhiên (RCT)**. Chênh lệch lớn hay p-value nhỏ cũng không chứng minh được nhân quả.
- **Simpson's paradox** (C04): X: $(95+270)/500=$ **0.73**; Y: $420/500=0.84$. X tốt hơn trong *từng* nhóm (0.95 so với 0.90, 0.675 so với 0.60) nhưng thua khi gộp lại, vì X phải làm nhiều ca khó hơn.
- **Base rate / Bayes** (C05): $PPV=\dfrac{TP}{TP+FP}$. $TP=8\times0.95=7.6$; $FP=39\,992\times0.002=79.98$ → $7.6/87.58\approx$ **0.087**. Khi tỉ lệ nền rất thấp, phần lớn cảnh báo là báo nhầm.

## 6. Bảo vệ dữ liệu cá nhân (Luật 91/2025/QH15, NĐ 356/2025, GDPR)

- **Nhạy cảm**: sinh trắc học (vân tay, khuôn mặt/selfie eKYC), sức khỏe, vị trí định vị, tài chính/tín dụng, sắc tộc, tôn giáo, đời sống tình dục… **Cơ bản**: họ tên, ngày sinh, SĐT, quốc tịch, email (A05 → **vân tay**).
- **GDPR Art. 5**: lawfulness/fairness/transparency · **purpose limitation** · **data minimisation** · accuracy · storage limitation · integrity & confidentiality · accountability.
  - Ứng dụng xin danh bạ + GPS dù không cần → vi phạm **tối thiểu hóa dữ liệu** (A04).
  - Dữ liệu thu để giao hàng đem dùng chấm điểm tín dụng → mục đích mới → **cần sự đồng ý riêng** (B05).
- **Rút lại đồng ý** (GDPR Art. 7(3)): ngừng xử lý *từ thời điểm rút*; các lần xử lý trước đó **vẫn hợp pháp** (B06 → A). Giọng nói vẫn là dữ liệu cá nhân (sinh trắc học) dù đã bỏ tên.
- **Sự cố rò rỉ** (GDPR Art. 33–34): ngăn chặn → đánh giá rủi ro → báo **cơ quan giám sát trong 72 giờ** kể từ khi biết → báo **chủ thể dữ liệu** nếu rủi ro cao (dữ liệu sức khỏe). Giữ nguyên log làm bằng chứng (C07 → C).
- **k-anonymity**: mỗi tổ hợp quasi-identifier xuất hiện ≥ k lần; **k = kích thước nhóm nhỏ nhất**. C06: (20–29,Nữ)=3, (30–39,Nam)=2, (20–29,Nam)=3, (30–39,Nữ)=2 → **k=2**. Lưu ý: nhóm (30–39,Nữ) chỉ có một chẩn đoán (Tiểu đường) → vẫn lộ thông tin (cần l-diversity).

## 7. EU AI Act (Regulation (EU) 2024/1689)

| Mức | Ví dụ trong đề | Nghĩa vụ |
|---|---|---|
| **Cấm** (Art. 5) | Nhận diện cảm xúc ở nơi làm việc/trường học (chấm "độ trung thực" qua nét mặt khi phỏng vấn), social scoring, thao túng | Không được bán → **no-go** |
| **Cao** (Annex III) | Lọc/xếp hạng CV (việc làm); chấm điểm tín dụng/cấp hạn mức (khả năng tín dụng); sinh trắc học | Quản lý rủi ro, data governance, tài liệu kỹ thuật, log, **human oversight**, độ chính xác/robustness, đánh giá sự phù hợp trước khi ra thị trường |
| **Hạn chế** (Art. 50) | Chatbot, deepfake | Minh bạch: báo cho người dùng biết họ đang tương tác với AI |
| **Tối thiểu** | Bộ lọc spam, AI trong game | Không có nghĩa vụ riêng (A07) |

- B09: công cụ lọc CV là **rủi ro cao** dù người quyết định cuối vẫn là con người.

## 8. OWASP Top 10 for LLM Applications 2025

| Mã | Tên | Dấu hiệu → biện pháp |
|---|---|---|
| LLM01 | Prompt Injection | Lệnh ẩn trong input/tài liệu (CV dạng PDF) → tách dữ liệu khỏi lệnh, lọc input, giới hạn quyền |
| LLM02 | Sensitive Information Disclosure | Lộ dữ liệu cá nhân hoặc nội bộ → lọc output, kiểm soát truy cập |
| LLM03 | Supply Chain | Model/thư viện bên thứ ba bị nhiễm |
| **LLM04** | **Data and Model Poisoning** | Dữ liệu huấn luyện/fine-tune bị đầu độc (review giả) (A08) |
| LLM05 | Improper Output Handling | Dùng output LLM mà không kiểm tra |
| LLM06 | Excessive Agency | Tool `send_email` tự gửi thư → bỏ quyền tự động, bắt người duyệt |
| LLM07 | System Prompt Leakage | Lộ system prompt |
| **LLM08** | **Vector and Embedding Weaknesses** | Vector DB dùng chung nhiều tenant gây rò rỉ chéo → **lọc theo tenant/quyền ngay ở tầng truy xuất** (B10) |
| LLM09 | Misinformation | Bịa thông tin (hallucination). Là *hậu quả*, không phải gốc |
| LLM10 | Unbounded Consumption | Tiêu tốn tài nguyên hoặc chi phí không giới hạn |

## 9. Công bằng (fairness)

| Tiêu chí | Yêu cầu |
|---|---|
| **Demographic parity** | $P(\hat Y=1\mid A=a)$ bằng nhau, tức tỉ lệ được chọn bằng nhau (A06 → D) |
| **Equal opportunity** | TPR bằng nhau |
| **Equalized odds** | TPR **và** FPR đều bằng nhau |
| Calibration | Trong mỗi nhóm, điểm 0.7 tương ứng ≈70% thật sự dương |
| Individual fairness | Hai cá thể giống nhau được xử lý như nhau |

- $TPR=\dfrac{TP}{TP+FN}$, $FPR=\dfrac{FP}{FP+TN}$. C09: TPR = 0.8 ở cả hai nhóm; FPR là $90/1800=0.05$ so với $114/950=0.12$ → đạt EO nhưng **vi phạm equalized odds** (B).
- **Disparate impact ratio** $=\dfrac{\text{tỉ lệ chọn thấp}}{\text{tỉ lệ chọn cao}}$; **quy tắc 4/5**: cần $\ge 0.8$.
  - C08: $0.28/0.375=$ **0.75** (<0.8).
  - D02: $0.41/0.62\approx0.66$.
  - D01: tỉ lệ cao nhất $168/280=0.6$, ngưỡng $0.8\times0.6=0.48$, cần $0.48\times150=72$ hồ sơ → thêm $72-57=$ **15**.
- **Biến proxy** (GPS, danh sách app, danh bạ, quận…) có thể mã hóa gián tiếp nông thôn/thu nhập. Bỏ thuộc tính nhạy cảm đi chưa đủ: phải đo DI, TPR, FPR theo từng nhóm, bỏ proxy, đặt lại ngưỡng, rồi kiểm toán định kỳ.

## 10. Tài liệu hóa và giám sát con người

- **Model card**: mục đích sử dụng, các trường hợp không nên dùng, dữ liệu đánh giá, hiệu năng theo nhóm, hạn chế đã biết (B07). **Datasheet** mô tả *bộ dữ liệu*. **DPIA** đánh giá tác động lên dữ liệu cá nhân. **SHAP** giải thích từng dự đoán.
- **Automation bias** (B08): tỉ lệ đồng ý 99.7%, 4 giây mỗi ca, giao diện chỉ có nút "Xác nhận" → giám sát chỉ mang tính hình thức (rubber-stamp). Khắc phục: hiện bằng chứng/lý do, đo tỉ lệ bác bỏ, kiểm tra ngẫu nhiên các ca đã duyệt, cài ca thử (canary).

## 11. Khung bài tự luận (D02, D03)

**D02: ví điện tử BNPL**
- (a) Phân loại dữ liệu:
  - Lịch sử giao dịch: tài chính, nhạy cảm → **giữ**, cần đồng ý rõ ràng cho mục đích chấm tín dụng.
  - Danh bạ: dữ liệu của bên thứ ba, không cần thiết → **bỏ**.
  - GPS 30 ngày: vị trí, nhạy cảm, là proxy → **bỏ**.
  - Danh sách app: proxy, vượt quá mục đích → **bỏ**.
  - Selfie eKYC: sinh trắc học → chỉ dùng **xác thực danh tính**, không đưa vào mô hình chấm điểm.
- (b) DI ≈ 0.66 < 0.8 → có bất lợi; proxy là GPS, app, danh bạ; đo DI, TPR, FPR theo vùng; giảm bằng bỏ proxy, reweighting, chỉnh ngưỡng.
- (c) EU: chấm điểm tín dụng là **rủi ro cao** (Annex III); GDPR Art. 22 cho quyền yêu cầu con người can thiệp; cần giải thích lý do từ chối, kênh khiếu nại, người duyệt lại các ca bị từ chối hoặc ca biên.

**D03: gói HR-tech bán tại EU**
- Tính năng 1 (nhận diện cảm xúc khi phỏng vấn) → **bị cấm**, **no-go**.
- Tính năng 2 (sàng lọc CV bằng LLM) → **rủi ro cao**. Rủi ro OWASP: LLM01 (lệnh ẩn trong PDF), LLM06 (`send_email` tự gửi), LLM05. Biện pháp: bỏ tự gửi thư, bắt người duyệt, coi CV là dữ liệu chứ không phải lệnh, kiểm thử công bằng → **go có điều kiện**.
- Tính năng 3 (chatbot RAG) → **rủi ro hạn chế** (phải báo là AI). Rủi ro OWASP: LLM02/LLM08 (lộ tài liệu nội bộ), LLM09 (thông tin sai), LLM07. Biện pháp: phân quyền tài liệu, trích dẫn nguồn, chỉ trả lời dựa trên tài liệu → **go**.

---

## Bẫy thường gặp

| Phương án nhiễu | Vì sao sai |
|---|---|
| Nhầm phản đảo hoặc "khi và chỉ khi" với **đảo** | Đảo là $q\to p$; phản đảo là $\neg q\to\neg p$ |
| `not a or not b` cho $\neg(a\lor b)$ | De Morgan đổi **or → and** |
| Phủ định "mọi…" thành "mọi… đều không" | Phủ định của $\forall$ là $\exists\neg$ |
| Khẳng định hậu kiện, phủ định tiền kiện | Không hợp lệ |
| Nhầm "thiếu kinh nghiệm" là appeal to authority | Công kích người nói là **ad hominem** |
| "Chênh 37 điểm % / p<0.05 nên PM đúng" | Có ý nghĩa thống kê ≠ nhân quả; bị confounding do tự chọn |
| "Người quyết định cuối là con người nên rủi ro tối thiểu" | Lọc CV vẫn là **rủi ro cao** |
| "Xử lý dữ liệu cá nhân nên bị cấm" | Mức cấm chỉ áp cho một số thực hành cụ thể (Art. 5) |
| Spam filter "phải gắn nhãn AI" hoặc "rủi ro cao" | Là **rủi ro tối thiểu** |
| Rút lại đồng ý khiến xử lý trước đó thành bất hợp pháp | Rút lại chỉ có hiệu lực từ thời điểm rút |
| Bỏ tên khỏi metadata là được dùng tiếp | Giọng nói vẫn là dữ liệu sinh trắc |
| Cập nhật chính sách là coi như đồng ý; mã hóa là đủ | Mục đích mới cần **đồng ý riêng**; bảo mật không thay được cơ sở pháp lý |
| "Chưa ai tải về nên không cần báo", "chờ điều tra 1 tháng" | Phải báo trong 72h; có thể báo từng phần |
| Xóa log sau sự cố | Làm mất bằng chứng |
| Chọn LLM09 Misinformation hoặc LLM01 khi dữ liệu huấn luyện bị đầu độc | Gốc rủi ro là **LLM04** |
| Sửa rò rỉ đa tenant bằng câu lệnh trong system prompt | Phải lọc ở **tầng truy xuất**, đây là LLM08 |
| "TPR bằng nhau nên đạt equalized odds" | Còn phải có FPR bằng nhau |
| So sánh số ca TP tuyệt đối | Phải dùng **tỉ lệ** |
| Gộp tỉ lệ X, Y bằng trung bình cộng của hai nhóm | Phải tính trên tổng số ca thực tế (Simpson) |
| Đồng ý 99.7% nghĩa là mô hình chính xác | Là **automation bias**; thêm người bấm trên cùng giao diện cũng vô ích |
| Datasheet hoặc DPIA cho tài liệu mô tả mô hình | Đó là **model card** |

## Tra nhanh

| Nếu gặp… | Dùng… |
|---|---|
| "mệnh đề đảo / phản đảo" | $q\to p$ / $\neg q\to\neg p$ |
| `not (a or b)` | `not a and not b` |
| Phủ định "mọi X có ≥1 Y" | "có X không có Y nào" |
| $\forall\exists$ hay $\exists\forall$ trên bảng | $\exists\forall$ mạnh hơn: tìm một phần tử phủ hết mọi x |
| Đếm tổ hợp thỏa quy tắc | Đếm số vi phạm rồi lấy $2^n$ trừ đi; khóa biến bị ép trước |
| "chỉ một người nói thật" | Thử từng thủ phạm, đếm số lời khai đúng |
| Phát hiện hiếm (truy nã, bệnh hiếm) | Bayes: $PPV=TP/(TP+FP)$ |
| Tỉ lệ gộp ngược với từng nhóm | Simpson; tính (tổng đúng)/(tổng ca) |
| Nhóm tự chọn có kết quả tốt hơn | Confounding → RCT/A/B test |
| Vân tay, khuôn mặt, sức khỏe, vị trí, tài chính | Dữ liệu **nhạy cảm** |
| Thu dữ liệu không cần thiết | Data minimisation |
| Dùng dữ liệu cho mục đích mới | Purpose limitation → xin đồng ý riêng |
| Rò rỉ dữ liệu | Ngăn chặn → đánh giá → báo cơ quan trong 72h → báo người dùng nếu rủi ro cao |
| CV / tuyển dụng, tín dụng, giáo dục | EU AI Act **rủi ro cao** |
| Cảm xúc ở nơi làm việc, social scoring | **Cấm** |
| Chatbot | **Hạn chế** (minh bạch) |
| Spam filter | **Tối thiểu** |
| Dữ liệu huấn luyện bị đầu độc | LLM04 |
| Lệnh ẩn trong tài liệu đầu vào | LLM01 |
| LLM có tool tự hành động | LLM06 Excessive Agency → human-in-the-loop |
| Vector DB dùng chung nhiều tenant | LLM08 → lọc theo tenant ở tầng truy xuất |
| Tỉ lệ chọn bằng nhau | Demographic parity |
| Chỉ TPR bằng nhau | Equal opportunity |
| TPR và FPR bằng nhau | Equalized odds |
| Kiểm tra công bằng tuyển chọn | DI = thấp/cao, cần ≥ 0.8; số tối thiểu $=\lceil 0.8\cdot r_{max}\cdot n\rceil$ |
| Ẩn danh bảng dữ liệu | k = kích thước nhóm QI nhỏ nhất |
| Tài liệu mục đích, hiệu năng theo nhóm, hạn chế | Model card |
| Người duyệt luôn đồng ý, duyệt rất nhanh | Automation bias → hiện bằng chứng, đo tỉ lệ bác bỏ, audit ngẫu nhiên |

### Đáp án nhanh
1C · 2D · 3C · 4C · 5C · 6D · 7D · 8C · 9A · 10B · 11D · **12: 5** · 13A · 14D · 15A · 16C · 17A · 18C · **19: 6** · **20: 0.75** · 21B · 22C · **23: 0.087** · **24: A** · **25: 2** · 26C · **27: 0.73** · 28 tự luận · **29: 15** · 30 tự luận
