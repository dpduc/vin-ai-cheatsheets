# CĐ-02 · Giải tích và tối ưu — Cheatsheet

> 30 câu (A: nhận biết 8 · B: thông hiểu 10 · C: vận dụng 9 · D: vận dụng cao 3). Bối cảnh: hồi quy giá nhà, phân loại ảnh, lọc spam, autograd tự viết, mạng dự đoán gia hạn gói.

## 1. Ý tưởng chính được kiểm tra

| # | Ý tưởng | Câu |
|---|---|---|
| 1 | MSE $=\frac1n\sum(y_i-\hat y_i)^2$ | A08 |
| 2 | Gradient chỉ hướng **tăng** nhanh nhất, tốc độ $\lVert\nabla f\rVert$; GD đi **ngược** gradient | A04, A05 |
| 3 | Đạo hàm theo hướng: $\nabla f\cdot \mathbf u$, $\mathbf u$ **đơn vị** | B04 |
| 4 | Quy tắc chuỗi $f'(g(x))\,g'(x)$; quy tắc tích | A02, A01, B02, B01, C01 |
| 5 | $\sigma' = \sigma(1-\sigma)$ (max 0,25); $\tanh' = 1-\tanh^2$ | A07 |
| 6 | Jacobian của $W\mathbf x$ là $W$ (đầu ra × đầu vào); $\nabla_\mathbf w\lVert X\mathbf w-\mathbf y\rVert^2 = 2X^\top(X\mathbf w-\mathbf y)$ | A03, B03 |
| 7 | Softmax + CE: $\partial L/\partial\mathbf z = \mathbf p-\mathbf y$; sigmoid + BCE: $(p-y)\mathbf x$ | B08, C09 |
| 8 | Cross-entropy trung bình batch: $-\frac1n\sum\ln p_{\text{đúng}}$ | B09 |
| 9 | Loss tăng vọt → nan = learning rate quá lớn; $w_{t+1}=(1-\eta\lambda)w_t$ phân kỳ khi $\lvert1-\eta\lambda\rvert>1$ | A06, C06 |
| 10 | Một bước GD bằng tay | B06, C09 |
| 11 | Momentum: triệt tiêu thành phần đổi dấu, cộng dồn thành phần cùng dấu | B05, C05 |
| 12 | Adam: bias correction $1/(1-\beta^t)$; bước đầu ≈ $\eta\,\text{sign}(g)$ | B07, C07 |
| 13 | Reverse-mode (backprop) rẻ khi $n$ tham số lớn, 1 đầu ra | B10 |
| 14 | Sai phân tiến $O(h)$ vs trung tâm $O(h^2)$ | C02 |
| 15 | Backprop qua mạng nhỏ ReLU; ReLU chết → gradient 0 | C08, D02 |
| 16 | Hessian: trị riêng trái dấu → điểm yên ngựa | C03 |
| 17 | Taylor bậc hai $L+\mathbf g^\top\Delta+\frac12\Delta^\top H\Delta$; Newton $-H^{-1}\mathbf g$ | C04, D01 |
| 18 | Autograd: nút rẽ nhánh phải **cộng dồn** grad (`+=`); residual $I+\partial F/\partial x$ | D03 |

## 2. Đạo hàm cơ bản & quy tắc chuỗi

| Hàm | Đạo hàm |
|---|---|
| $\sqrt{1+x^2}$ | $\dfrac{x}{\sqrt{1+x^2}}$ (A01) |
| $x e^{-x^2}$ | $e^{-x^2}(1-2x^2)$ (B02) |
| $\ln(1+e^{2x})$ | $\dfrac{2e^{2x}}{1+e^{2x}}$ → tại $e^{2x}=3$: $6/4=1{,}5$ (B01) |
| $\ln(1+r^2),\ r=wx-y$ | $\dfrac{2r}{1+r^2}\cdot x$ → $x=2,y=1,w=1$: $r=1$, kết quả $2$ (C01) |
| $\sigma(z)$ | $\sigma(1-\sigma)$, max $0{,}25$ tại $z=0$ (A07) |
| $\tanh z$ | $1-\tanh^2 z$ |
| $\text{ReLU}(z)$ | $1$ nếu $z>0$, $0$ nếu $z<0$ |

- Chuỗi: $\frac{dy}{dx}=f'(g(x))\,g'(x)$ — $f'$ tính **tại $g(x)$**, rồi **nhân** $g'(x)$ (A02).
- Nhiều biến: nút có nhiều nhánh ra → **cộng** gradient từ mọi nhánh: $\frac{\partial L}{\partial u}=\sum_k \frac{\partial L}{\partial c_k}\frac{\partial c_k}{\partial u}$.

## 3. Gradient, Jacobian, ma trận

- $\nabla f\cdot\mathbf u=\lVert\nabla f\rVert\cos\theta$ → max khi $\mathbf u\parallel\nabla f$. Gradient **không** chuẩn hoá, chỉ thông tin **cục bộ** (A04).
- Đạo hàm theo hướng $\mathbf v$: chuẩn hoá $\mathbf u=\mathbf v/\lVert\mathbf v\rVert$. B04: $f=3x^2+xy-2y^2$, $\nabla f(1,2)=(8,-7)$, $\mathbf u=(0{,}6;0{,}8)$ → $4{,}8-5{,}6=-0{,}8$.
- $\mathbf y=W\mathbf x$, $\mathbf x\in\mathbb R^4,\mathbf y\in\mathbb R^2$ → Jacobian $=W$, $2\times4$ (A03). Backprop: $\frac{\partial L}{\partial\mathbf x}=W^\top\frac{\partial L}{\partial\mathbf y}$.
- $L=\lVert X\mathbf w-\mathbf y\rVert^2$: $\nabla L=2X^\top(X\mathbf w-\mathbf y)\in\mathbb R^d$; Hessian $2X^\top X$ (B03).
- Softmax + CE: $\partial L/\partial z_j=p_j-y_j$ (B08). Sigmoid + BCE: $\nabla_\mathbf w L=(p-y)\mathbf x$, $\partial L/\partial b=p-y$ — thừa số $\sigma'$ đã triệt tiêu.

## 4. Loss

```python
mse = np.mean((y - yhat)**2)                  # A08: (1+4+0)/3 = 1.67
ce  = -np.mean(np.log(p[np.arange(n), y]))    # B09: -(ln0.7 + ln0.25)/2 = 0.871
bce = -(y*np.log(p) + (1-y)*np.log(1-p))
```

## 5. Gradient descent & learning rate

- Cập nhật: $\mathbf w\leftarrow\mathbf w-\eta\nabla L(\mathbf w)$ (A05). $L(\mathbf w-\eta\mathbf g)\approx L(\mathbf w)-\eta\lVert\mathbf g\rVert^2$.
- B06: $L=(w_1-3)^2+2(w_2+1)^2$, $\nabla L(0,0)=(-6,4)$, $\eta=0{,}1$ → $(0{,}6;-0{,}4)$.
- C09: $p=\sigma(0)=0{,}5$, $\nabla=(0{,}5-1)(1,2)=(-0{,}5;-1)$ → $\mathbf w=(0{,}05;0{,}1)$.
- Loss bậc hai $\frac{\lambda}{2}w^2$: $w_{t+1}=(1-\eta\lambda)w_t$.
  - $0<1-\eta\lambda<1$: hội tụ đơn điệu; $-1<\cdot<0$: hội tụ dao động; $<-1$: **phân kỳ dao động**.
  - C06: $L=2w^2$ ($\lambda=4$), $\eta=0{,}6$: hệ số $-1{,}4$ → $w_3=(-1{,}4)^3=-2{,}744$; phân kỳ khi $\eta>0{,}5$.
- Dấu hiệu LR quá lớn: loss $2{,}3\to4{,}85\to37\to10^4\to$ nan (A06). LR nhỏ → giảm chậm; overfit → val loss tăng, train loss vẫn giảm.

## 6. Momentum & Adam

```python
# Momentum (dạng đề dùng)
v = beta*v + grad(w);  w = w - eta*v
# C05: L=w^2, w0=1, eta=0.1, beta=0.9:  v1=2, w1=0.8;  v2=0.9*2+1.6=3.4, w2=0.46
# Adam
m = b1*m + (1-b1)*g;  v = b2*v + (1-b2)*g**2
mh = m/(1-b1**t);     vh = v/(1-b2**t)
w -= eta * mh/(np.sqrt(vh)+eps)
```
- Momentum: $\mathbf v_t=\sum_k\beta^k\nabla L_{t-1-k}$ → thành phần zig-zag triệt tiêu, thành phần đều cộng dồn tới $\frac1{1-\beta}$ lần (B05). Không dùng Hessian, không đảm bảo cực tiểu toàn cục.
- Bias correction: $m_0=0$ → $m_1=(1-\beta_1)g=0{,}1g$ nhỏ; chia $1-\beta_1^t$ trả về $g$ (B07). Hệ số giảm dần về 1.
- Adam bước 1: $\hat m_1=g$, $\hat v_1=g^2$ → bước $\approx\eta\,\text{sign}(g)=0{,}01$ cho **mọi** tham số bất kể $|g|$ (C07).

## 7. Autodiff & kiểm gradient

- $f:\mathbb R^n\to\mathbb R^m$: forward-mode ~ $n$ lượt, reverse-mode ~ $m$ lượt. Mạng $10^6$ tham số, loss vô hướng → reverse rẻ hơn ~$10^6$ lần (B10). Cả hai đều **chính xác**; reverse phải **lưu activation**.
- Sai phân hữu hạn, C02 ($x^3$, $x=1$, $h=0{,}1$): tiến $\frac{1{,}331-1}{0{,}1}=3{,}31$ (sai số $\approx3h$, $O(h)$); trung tâm $\frac{1{,}331-0{,}729}{0{,}2}=3{,}01$ (sai số $h^2$, $O(h^2)$); lùi $=2{,}71$.
- Gradient check: $\frac{L(\theta+h e_i)-L(\theta-h e_i)}{2h}$, $h\sim10^{-5}$, so sai số tương đối. Rủi ro ReLU: nếu tiền kích hoạt sát 0 (trong $\pm h$) thì sai phân vắt qua điểm gãy → lệch giả.

## 8. Hessian, Taylor, Newton

- Điểm dừng: $H$ xác định dương → cực tiểu; âm → cực đại; trị riêng **trái dấu** ($\det H<0$) → **yên ngựa**. Đường chéo dương **chưa đủ** (C03: $f=x^2+4xy+y^2$, $H=\begin{pmatrix}2&4\\4&2\end{pmatrix}$, trị riêng $6,-2$).
- Taylor bậc hai: $L(\mathbf w_0+\Delta)\approx L+\mathbf g^\top\Delta+\tfrac12\Delta^\top H\Delta$. C04: $5+(-1{,}5)+\tfrac12\cdot1=4{,}00$.
- Newton: $\mathbf w_N=\mathbf w_0-H^{-1}\mathbf g$ → loss bậc hai tới **đúng cực tiểu** sau 1 bước.
- D01: $L=w_1^2+w_1w_2+w_2^2-3w_1$, $\mathbf g_0=(-3,0)$, $H=\begin{pmatrix}2&1\\1&2\end{pmatrix}$: $\mathbf w_N=(2,-1)$, $L=-3$; $\mathbf w_G=(1{,}5;0)$, $L=-2{,}25$ → hiệu $0{,}75$.

## 9. Mẫu bài tự luận

**C08 – backprop mạng nhỏ:** $x=2,y=1,w_1=0{,}5,w_2=-1$: $h=1$, $\hat y=-1$, $\delta=\hat y-y=-2$; $\partial L/\partial w_2=\delta h=-2$; $\partial L/\partial w_1=\delta w_2\cdot\text{ReLU}'(1)\cdot x=4$ → $(4;-2)$.

**D03 – autograd gán đè** ($u=wx,\ t=3u,\ v=u^2,\ L=v+t$, $w=1,x=2$)
- (a) $u=2,t=6,v=4,L=10$. Nút $u$ có **2 nhánh ra** ($t$ và $v$).
- (b) $\frac{\partial L}{\partial u}=3+2u=7$; $\frac{\partial L}{\partial w}=7x=14$. Kiểm: $L=w^2x^2+3wx$ → $2wx^2+3x=8+6=14$.
- (c) Gán đè, thứ tự $L\to t\to v\to u$: sau $t$: `u.grad=3`; sau $v$: `u.grad=2u·1=4` (đè mất 3); $u$: `w.grad=4·2=8` ≠ 14. Sai vì bỏ nhánh $t$. Test: so với sai phân trung tâm, hoặc test biểu thức dùng biến 2 lần (`a*a`, `a+a` → grad phải là `2a`, `2`). Sửa: `+=` và reset grad về 0 trước mỗi lượt.
- (d) $\mathbf y=\mathbf x+F(\mathbf x)$: $\frac{\partial L}{\partial\mathbf x}=\frac{\partial L}{\partial\mathbf y}\big(I+\frac{\partial F}{\partial\mathbf x}\big)$ — nhánh identity luôn chuyển nguyên gradient, tránh tích nhiều Jacobian nhỏ → vanishing.

**D02 – mạng 2 lớp + BCE** ($\mathbf x=(1,2)$, $y=1$, $W^{(1)}=\begin{pmatrix}1&-1\\0{,}5&0{,}5\end{pmatrix}$, $\mathbf b^{(1)}=0$, $\mathbf w^{(2)}=(2,1)$, $b^{(2)}=-1{,}5$)
- (a) $\mathbf a=(-1;1{,}5)$, $\mathbf h=(0;1{,}5)$, $z=0$, $p=0{,}5$, $L=\ln2\approx0{,}693$.
- (b) $\partial L/\partial z=p-y=-0{,}5$; $\partial L/\partial\mathbf w^{(2)}=-0{,}5\mathbf h=(0;-0{,}75)$; $\partial L/\partial b^{(2)}=-0{,}5$; $\partial L/\partial\mathbf h=-0{,}5\mathbf w^{(2)}=(-1;-0{,}5)$; $\partial L/\partial\mathbf a=(0;-0{,}5)$ (ReLU chết ở $a_1=-1$); $\partial L/\partial W^{(1)}=\boldsymbol\delta_a\mathbf x^\top=\begin{pmatrix}0&0\\-0{,}5&-1\end{pmatrix}$; $\partial L/\partial\mathbf b^{(1)}=(0;-0{,}5)$. Hàng 1 bằng 0 vì $a_1<0$ → $\text{ReLU}'=0$.
- (c) $\eta=0{,}5$: $W^{(1)}=\begin{pmatrix}1&-1\\0{,}75&1\end{pmatrix}$, $\mathbf b^{(1)}=(0;0{,}25)$, $\mathbf w^{(2)}=(2;1{,}375)$, $b^{(2)}=-1{,}25$. Xuôi lại: $\mathbf a=(-1;3)$, $z=1{,}375\cdot3-1{,}25=2{,}875$, $p\approx0{,}947$, $L\approx0{,}055<0{,}693$.
- (d) Sai phân trung tâm cho 1 phần tử, so sai số tương đối; rủi ro: điểm gãy ReLU tại 0 (ở đây $a_1=-1$ xa 0 nên an toàn).

## 10. Đáp án nhanh

**Chính thức** = có "Đáp án X" trong PDF (bỏ qua "bản in"); **tự giải** = điền số / tự luận, tính lại bằng Python.

| Câu | Đáp án | Nguồn | Câu | Đáp án | Nguồn |
|---|---|---|---|---|---|
| A08 | `1.67` | tự giải | B01 | `1.50` | tự giải |
| A04 | A | chính thức | B03 | B $2X^\top(X\mathbf w-\mathbf y)$ | chính thức |
| A03 | B $W$, $2\times4$ | chính thức | B09 | `0.871` | tự giải |
| A01 | A | chính thức | C09 | D $(0{,}05;0{,}1)$ | chính thức |
| A06 | A (LR quá lớn) | chính thức | C02 | C $3{,}31$ và $3{,}01$ | chính thức |
| A07 | C $\sigma(1-\sigma)$ | chính thức | C04 | `4.00` | tự giải |
| A05 | D | chính thức | C05 | `0.46` | tự giải |
| A02 | C | chính thức | C06 | D $-2{,}744$ | chính thức |
| B10 | A | chính thức | C08 | D $(4;-2)$ | chính thức |
| B04 | `-0.8` | tự giải | C03 | D yên ngựa | chính thức |
| B05 | C | chính thức | C01 | `2.00` | tự giải |
| B02 | B | chính thức | C07 | C $0{,}01$ cả hai | chính thức |
| B07 | B | chính thức | D03 | tự luận, $\partial L/\partial w=14$, lỗi cho 8 (mục 9) | tự giải |
| B06 | A $(0{,}6;-0{,}4)$ | chính thức | D02 | tự luận, $L$: $0{,}693\to0{,}055$ (mục 9) | tự giải |
| B08 | C $\mathbf p-\mathbf y$ | chính thức | D01 | `0.75` | tự giải |

(20 chính thức · 10 tự giải.)

## 11. Bẫy thường gặp

- **Gradient ≠ hướng giảm**: $-\nabla f$ mới là hướng giảm; gradient không có độ dài 1, không chỉ thẳng tới cực tiểu toàn cục.
- **GD sai dạng**: chia cho $\eta$, nhân $\eta$ vào $\mathbf w$, hay **cộng** gradient (ascent) đều sai.
- **Chuỗi**: tính $f'$ tại $x$ thay vì tại $g(x)$; hợp các đạo hàm $f'(g'(x))$; quên nhân $g'$ hoặc quên hệ số $\frac12$ của $(\sqrt u)'$.
- **Quy tắc tích**: bỏ số hạng $x'\cdot e^{-x^2}$; sai dấu $(-x^2)'=-2x$; quên hệ số 2 trong $2(w_2+1)^2\to4(w_2+1)$.
- **Sigmoid**: $\sigma^2$ hay $1-\sigma^2$ (của tanh) đều sai.
- **Jacobian vs layout**: $W^\top$ chỉ xuất hiện khi lan truyền ngược; $\mathbf x^\top$ là đạo hàm theo trọng số; $W^\top W$ / $2X^\top X$ là Hessian.
- **$2(X\mathbf w-\mathbf y)$** là gradient theo phần dư, thiếu $X^\top$; $2X(\cdot)$ sai kích thước.
- **Softmax+CE**: $-\mathbf y\oslash\mathbf p$ là gradient theo $\mathbf p$; $\mathbf y-\mathbf p$ sai dấu. **Sigmoid+BCE**: không nhân thêm $\sigma'(0)=0{,}25$.
- **Đạo hàm theo hướng** phải dùng vector đơn vị (không chia → 5 lần lớn hơn).
- **Momentum** không tự giảm LR, không dùng Hessian; **Adam** bias correction không phải LR schedule, không thay $\sqrt{\hat v}$. Bỏ hiệu chỉnh bias → $0{,}0316$; chỉ hiệu chỉnh $m$ → $0{,}316$.
- **Batch lớn** làm gradient ổn định hơn, không gây bùng nổ.
- **Hệ số $1-\eta\lambda$ âm**: $w$ đổi dấu mỗi bước — phân kỳ nếu $|\cdot|>1$, đừng bỏ dấu âm.
- **Sai phân trung tâm** không chính xác tuyệt đối với bậc ≥ 3; sai phân lùi khác sai phân tiến.
- **Backprop**: đảo thứ tự gradient $w_1/w_2$; dùng $\delta=y-\hat y$ (sai dấu); quên nhân $x$.
- **Hessian**: $\det H<0$ là yên ngựa (không phải cực đại); đường chéo dương chưa đủ; không cần $H$ đường chéo.
- **Autograd**: gán đè `grad =` mất nhánh; quên reset grad giữa các lượt → cộng dồn sai.
- Reverse-mode **không** tiết kiệm bộ nhớ (phải lưu activation); thứ tự nhân Jacobian chỉ ảnh hưởng chi phí.

## 12. Tra nhanh

| Gặp tình huống | Dùng |
|---|---|
| Đo lỗi hồi quy | MSE $\frac1n\sum(y-\hat y)^2$ |
| Hàm hợp | $f'(g(x))\,g'(x)$ |
| Hàm có nhiều nhánh dùng chung biến | Cộng gradient các nhánh |
| Tốc độ thay đổi theo hướng $\mathbf v$ | $\nabla f\cdot\mathbf v/\lVert\mathbf v\rVert$ |
| Lớp tuyến tính $W\mathbf x$ | Jacobian $W$; ngược: $W^\top\delta$ |
| Least squares | $\nabla=2X^\top(X\mathbf w-\mathbf y)$, $H=2X^\top X$ |
| Softmax + CE / sigmoid + BCE | $\mathbf p-\mathbf y$ / $(p-y)\mathbf x$ |
| Loss nổ → nan | Giảm learning rate |
| GD trên $\frac\lambda2w^2$ | $w_t=(1-\eta\lambda)^tw_0$, ổn định khi $\eta<2/\lambda$ |
| Thung lũng hẹp, zig-zag | Momentum |
| Gradient khác thang đo giữa tham số | Adam (bước ≈ $\eta$) |
| Nhiều tham số, loss vô hướng | Reverse-mode autodiff (backprop) |
| Kiểm code gradient | Sai phân trung tâm $O(h^2)$ |
| Phân loại điểm dừng | Trị riêng Hessian (dấu) |
| Dự đoán loss sau bước | Taylor bậc hai |
| Loss bậc hai, 1 bước tới cực tiểu | Newton $-H^{-1}\mathbf g$ |
| Gradient qua mạng sâu | Residual $\mathbf x+F(\mathbf x)$ |
