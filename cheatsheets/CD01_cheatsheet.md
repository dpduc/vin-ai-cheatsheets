# Cheatsheet CĐ-01 · Đại số tuyến tính

Oct 18, 2026 · @Bean

## 1. Vector: tích vô hướng, chuẩn, cosine, phép chiếu

Đề hỏi nhiều nhất về embedding: tích vô hướng ra một SỐ, cosine bỏ qua độ dài, phép chiếu dùng để "gỡ" một hướng.

```latex
u \cdot v = \sum_i u_i v_i \qquad \|w\|_1 = \sum_i |w_i| \qquad \|w\|_2 = \sqrt{\sum_i w_i^2}
```

```latex
\cos(q,d) = \frac{q \cdot d}{\|q\|\,\|d\|}
```

```latex
\text{proj}_g w = \frac{w \cdot g}{\|g\|^2}\, g \qquad w_\perp = w - \text{proj}_g w \;\;(w_\perp \cdot g = 0)
```

- u · v = 0 ⇔ hai vector vuông góc. Kết quả là số vô hướng, không phải vector (bẫy Câu 5).
- Nhân d với k > 0: tích vô hướng nhân k, cosine giữ nguyên (Câu 18).
- Vector đã chuẩn hóa ‖q‖ = ‖d‖ = 1: xếp hạng theo L2 tương đương xếp hạng theo cosine.

```latex
\|q-d\|^2 = 2 - 2\cos(q,d) \;\Rightarrow\; \cos = 1 - \tfrac{\|q-d\|^2}{2}
```

- Hard debiasing: chiếu từ lên hướng giới tính rồi trừ đi; kiểm tra lại bằng w⊥ · g = 0 (Câu 21).

## 2. Ma trận: kích thước, chuyển vị, nghịch đảo, định thức

Luôn kiểm tra kích thước trước: (m×n)(n×p) = m×p, hai số "trong" phải bằng nhau.

```latex
(AB)^\top = B^\top A^\top \qquad (AB)^{-1} = B^{-1}A^{-1}
```

```latex
\begin{bmatrix} a & b \\ c & d \end{bmatrix}^{-1} = \frac{1}{ad-bc}\begin{bmatrix} d & -b \\ -c & a \end{bmatrix}
```

```latex
\det(kA) = k^n \det A \quad \det(A^\top)=\det A \quad \det(AB)=\det A\,\det B \quad \det(A^{-1}) = 1/\det A
```

- nn.Linear(in, out) lưu W cỡ out×in; Y = XWᵀ + b → (batch×in)(in×out) = batch×out (Câu 6: 32×512).
- AB ≠ BA; BA có thể xác định dù khác kích thước AB (Câu 14: B 2×3 · A 3×2 = 2×2).
- Nghịch đảo 2×2: đổi chỗ đường chéo chính, đổi dấu đường chéo phụ, chia det. Không phải nghịch đảo từng phần tử.
- Ma trận trực giao (xoay, RoPE): R⁻¹ = Rᵀ = R(−θ), det = 1, giữ độ dài.
- Định thức 3×3: khai triển theo hàng/cột có nhiều số 0, dấu xen kẽ + − +.
- Ý nghĩa hình học: |det M| = hệ số phóng diện tích/thể tích; det < 0 → lật hướng (soi gương); det = 0 → ép dẹt, không khả nghịch. Diện tích không bao giờ âm (Câu 23: 4 × |−5| = 20, bị lật).

## 3. Hạng, độc lập tuyến tính, hệ phương trình

Hạng = số hàng (hoặc cột) độc lập tuyến tính = số giá trị kỳ dị khác 0.

- n vector trong Rⁿ phụ thuộc tuyến tính ⇔ det(ma trận ghép) = 0. Tìm tham số k: giải det = 0 (Câu 26: 6k − 12 = 0 → k = 2).
- Mẹo nhanh: tìm tổ hợp, ví dụ v₃ = 2v₁ + v₂ (Câu 7D: (5,5,0) = 2(1,2,−1) + (3,1,2)).
- Cột tính từ các cột khác (đặc trưng dẫn xuất) không làm tăng hạng: Câu 24 có f₃ = f₁ + 2f₂ → rank = 2. Đây là đa cộng tuyến, khiến XᵀX suy biến.
- Rank ≤ min(số hàng, số cột).

```latex
\text{rank}(A) = \text{rank}([A\,|\,b]) = n \Rightarrow \text{nghiệm duy nhất}\qquad = r < n \Rightarrow \text{vô số}\qquad \text{rank}(A) < \text{rank}([A|b]) \Rightarrow \text{vô nghiệm}
```

- "3 phương trình 3 ẩn" không đảm bảo có nghiệm duy nhất. Câu 12: PT2 = 2×vế trái PT1 nhưng vế phải 3 ≠ 2 → mâu thuẫn → vô nghiệm.

## 4. Trị riêng, vector riêng, chéo hóa

Với ma trận 2×2, mọi thứ đi qua phương trình đặc trưng dùng trace và det.

```latex
Av = \lambda v \qquad \lambda^2 - (\text{tr}A)\,\lambda + \det A = 0 \qquad \sum \lambda_i = \text{tr}A \quad \prod \lambda_i = \det A
```

- Kiểm tra vector riêng: tính Av, xem có cùng phương với v không (Câu 1: A(2,1) = (12,6) = 6·(2,1)).
- Câu 8: trị riêng 2, 3, −1 → det = −6, tr = 4.
- Ma trận đối xứng: trị riêng thực, vector riêng trực giao.
- p(A) có trị riêng p(λ), vector riêng giữ nguyên (Câu 19: λ³ − λ → 6 và 0). A⁻¹ có trị riêng 1/λ.

```latex
A = SDS^{-1} \;\Rightarrow\; A^n = SD^nS^{-1} \qquad x_n = c_1\lambda_1^n v_1 + c_2\lambda_2^n v_2
```

- Dài hạn do trị riêng có trị tuyệt đối lớn nhất quyết định: lớn hơn 1 thì bùng nổ, nhỏ hơn 1 thì tiêu biến. Không xét theo det hay từng phần tử (Câu 27: trị riêng 1,1 và 0,1, nên chuẩn của h tăng cỡ 1,1 mũ t).
- Markov (cột tổng bằng 1): luôn có trị riêng 1; vector riêng của nó, chuẩn hóa tổng bằng 1, là trạng thái dừng. Trị riêng thứ hai quyết định tốc độ hội tụ.
- Câu 28: trị riêng 1 với v = (3,2), trị riêng 0,5 với v = (1,−1). Pro sau 3 tháng = 0,35, tiến về 0,4; sai lệch giảm một nửa mỗi tháng.

```latex
x_n = 0.2\begin{pmatrix}3\\2\end{pmatrix} + 0.4\,(0.5)^n\begin{pmatrix}1\\-1\end{pmatrix}
```

## 5. SVD, PCA, bình phương tối thiểu, LoRA

**SVD.** Giá trị kỳ dị = căn bậc hai trị riêng của XᵀX; rank = số σ khác 0 (Câu 9: √16, √9, 0 → 4, 3, 0; rank 2).

```latex
X = U\Sigma V^\top \qquad \sigma_i = \sqrt{\lambda_i(X^\top X)} \qquad \min_{\text{rank } k}\|R - R_k\|_F = \sqrt{\sigma_{k+1}^2 + \dots + \sigma_r^2}
```

- Eckart–Young: cắt cụt SVD là xấp xỉ hạng k tốt nhất. Câu 25: σ = 6, 3, 2, giữ 1 → sai số √(9+4) ≈ 3,61.

**PCA.** Căn giữa dữ liệu, lập ma trận hiệp phương sai, lấy vector riêng của trị riêng lớn nhất làm PC1 (w₁).

```latex
C = \frac{1}{n-1}X_c^\top X_c \qquad z = w_1^\top(x-\mu) \qquad \hat{x} = \mu + z\,w_1 \qquad \text{EVR} = \frac{\lambda_1}{\sum_i \lambda_i}
```

- Sai số tái tạo theo Pythagore: ‖x − μ‖² = z² + sai số². Câu 20: 20 = 4,4² + sai số², nên sai số = 0,8.
- Câu 30: μ = (5,3); C = \[\[10, 4\], \[4, 4\]\]; trị riêng 12 và 2; PC1 = (2,1)/√5; giải thích 12/14 ≈ 85,7%. Score ≈ −4,02; −1,34; 1,34; 4,02, nên giữ 1 chiều là hợp lý.

**Bình phương tối thiểu.** Hệ nhiều phương trình hơn ẩn, không có nghiệm đúng, thì giải phương trình chuẩn.

```latex
A^\top A\,\beta = A^\top y \qquad A^\top A = \begin{bmatrix} n & \sum x \\ \sum x & \sum x^2 \end{bmatrix},\; A^\top y = \begin{bmatrix} \sum y \\ \sum xy \end{bmatrix} \qquad b = \frac{\sum (x-\bar x)(y-\bar y)}{\sum (x-\bar x)^2},\; a = \bar y - b\bar x
```

- Câu 29: Σx = 10, Σx² = 30, Σy = 16, Σxy = 47, nên b = 1,4 và a = 0,5. Dự báo x = 6: y = 8,9 giây.

**LoRA.** Đóng băng W₀ (m×n), chỉ học ΔW = BA với B là m×r, A là r×n. Số tham số = r(m + n) thay vì m·n; rank(ΔW) ≤ r.

- Câu 3: 8 × (1024 + 1024) = 16 384, so với 1 048 576 khi tinh chỉnh toàn bộ (≈ 1,6%).

## 6. Bẫy thường gặp và đáp án nhanh

Các phương án nhiễu trong đề lặp lại cùng vài lỗi sai.

- (AB)ᵀ = AᵀBᵀ: sai thứ tự. Đúng là BᵀAᵀ.
- det(kA) = k·det A: sai. Đúng là kⁿ·det A (n = cấp ma trận).
- Giá trị kỳ dị = trị riêng của XᵀX: sai, phải lấy căn.
- Số phương trình bằng số ẩn nên nghiệm duy nhất: sai, phải xét hạng.
- Ổn định xét theo det hoặc độ lớn phần tử: sai, xét trị riêng lớn nhất.
- Diện tích âm khi det âm: sai, diện tích dùng trị tuyệt đối của det; dấu chỉ báo lật hướng.
- Nghịch đảo từng phần tử: sai với mọi ma trận.
- p(A) phải tính lại vector riêng: sai, vector riêng giữ nguyên.
- Tích vô hướng ra vector: sai, ra một số.

Đáp án do mình tự tính, dùng để đối chiếu sau khi làm.

| Câu | Đáp án | Mẹo / bước chính |
| --- | --- | --- |
| 1 | A | A(2,1) = 6(2,1) |
| 2 | B | BᵀAᵀ, 4×2 |
| 3 | B | 8 × 2048 = 16 384 |
| 4 | A | 3 + 4 + 0 + 12 = 19 |
| 5 | A | 8 − 5 − 3 = 0 |
| 6 | C | 32×512 |
| 7 | D | v₃ = 2v₁ + v₂ |
| 8 | C | det = −6, tr = 4 |
| 9 | A | σ = 4, 3, 0; rank 2 |
| 10 | 21 | 2(12 − 2) + 1(1 − 0) |
| 11 | 4 | λ² − 3λ − 4 = 0 |
| 12 | A | Vô nghiệm |
| 13 | C | Rᵀ = R(−θ) |
| 14 | C | 2×2: 5, 3 / 1, 3 |
| 15 | C | 2³ × 5 = 40 |
| 16 | 0,667 | 12 / (3 × 6) |
| 17 | D | det = 2 |
| 18 | B | Dot ×2, cosine giữ nguyên |
| 19 | B | λ³ − λ: 6 và 0 |
| 20 | D | z = 4,4; sai số 0,8 |
| 21 | B | proj = g → (2, −2, 1) |
| 22 | 0,82 | 1 − 0,36/2 |
| 23 | B | det = −5 → diện tích 4 × 5 = 20, lật hướng |
| 24 | 2 | f₃ = f₁ + 2f₂ |
| 25 | 3,61 | √(9 + 4) |
| 26 | 2 | 6k − 12 = 0 |
| 27 | D | λ = 1,1 và 0,1 |
| 28 | Tự luận | λ = 1; 0,5 · Pro tháng 3 = 0,35 · giới hạn 0,4 |
| 29 | 8,9 | a = 0,5; b = 1,4 |
| 30 | Tự luận | λ = 12; 2 · PC1 = (2,1)/√5 · 85,7% |
