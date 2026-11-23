# Bài 1: Chữ ký Schnorr → Dilithium "toy" (§7.1.1–7.1.2)
 
## Phần A: Công cụ toán cần biết
 
### A1. Phép mod và đồng dư
 
$a \bmod n$ là **số dư** khi chia $a$ cho $n$, nằm trong $\{0, 1, \dots, n-1\}$.
 
- $26 \bmod 11 = 4$ (vì $26 = 2 \cdot 11 + 4$)
- $32 \bmod 23 = 9$
Ký hiệu đồng dư:
 
$$a \equiv b \pmod n$$
 
đọc là "$a$ đồng dư $b$ theo modulo $n$": $a$ và $b$ **có cùng số dư** khi chia cho $n$ (tương đương: $n$ chia hết $a - b$).
 
- $32 \equiv 9 \pmod{23}$ vì $32 - 9 = 23$.
- $24 \equiv 1 \pmod{23}$.
**Quy tắc quan trọng:** khi tính mod $n$, **có thể rút gọn bất cứ lúc nào** rồi tính tiếp, kết quả không đổi. Ví dụ:
 
$$8^2 = 64 \equiv 18 \pmod{23}, \qquad 8^4 = (8^2)^2 \equiv 18^2 = 324 \equiv 2 \pmod{23}.$$
 
Không cần tính $8^4 = 4096$ rồi mới chia.
 
### A2. Nghịch đảo modulo
 
$b^{-1} \pmod p$ là số $x$ sao cho $b \cdot x \equiv 1 \pmod p$. Nghịch đảo modulo của một số là số mà khi nhân với nó, kết quả chia cho modulo sẽ dư 1.

Một số \(a\) có nghịch đảo modulo \(m\) khi và chỉ khi \(a\) và \(m\) phải nguyên tố cùng nhau.
 
- $12^{-1} \pmod{23} = 2$, vì $12 \cdot 2 = 24 \equiv 1$.
Với $p$ nguyên tố, mọi số khác 0 đều có nghịch đảo.
 
### A3. Nhóm cyclic, phần tử sinh, cấp
 
Lũy thừa của $g = 2$ theo modulo $23$ (mỗi ô = ô trước × 2, rồi mod 23):
 
| $k$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| $2^k \bmod 23$ | 1 | 2 | 4 | 8 | 16 | 9 | 18 | 13 | 3 | 6 | 12 | **1** |
 
Đến $k = 11$ thì quay về 1, rồi lặp lại y hệt. Ta có:
 
- $g = 2$ là **phần tử sinh** (generator).
- $G = \{1, 2, 4, 8, 16, 9, 18, 13, 3, 6, 12\}$ là **nhóm cyclic** sinh bởi $g$.
- **Cấp** (order) của nhóm là $n = 11$: số phần tử, cũng là số mũ nhỏ nhất để $g^n = 1$.
Phân biệt hai con số:
 
- $p = 23$: modulo để tính **giá trị** $g^k$.
- $n = 11$: modulo để tính **số mũ** $k$.
### A4. Quy tắc lũy thừa
 
$$g^{x} \cdot g^{y} = g^{x+y}, \qquad (g^{x})^{y} = g^{xy}, \qquad g^{-x} = (g^{x})^{-1}.$$
 
### A5. Vì sao số mũ được rút gọn mod $n$
 
Vì $g^{11} \bmod 23$  = 1$:
 
$$g^{k + 11} = g^k \cdot g^{11} = g^k \cdot 1 = g^k.$$
 
Cộng thêm 11 vào số mũ không đổi kết quả, nên **số mũ chỉ cần xét theo mod $n = 11$**.
 
Ví dụ: $2^{21} = 2^{21 \bmod 11} = 2^{10} = 12$ (tra bảng).
 
Đây là lý do trong Schnorr công thức $z = y + ca$ lấy "$\bmod n$" chứ không phải "$\bmod p$".
 
### A6. Bài toán logarit rời rạc
 
- Biết $a$, tính $A = g^a$: **dễ**.
- Biết $A$, tìm $a$: **khó** khi $n$ lớn (cỡ $2^{256}$). Ở bảng trên ta "tra" được vì $n$ chỉ bằng 11.
### A7. Hàm băm và ký hiệu $\Vert$
 
- $M \Vert w$: **nối string** (concatenation), ghép bit của $M$ rồi đến bit của $w$.
- $H(\cdot)$: hàm băm, nhận chuỗi bất kỳ, trả về chuỗi bit cố định. Trong Schnorr, đọc chuỗi bit đó như một số trong $[0, n-1]$.
---
 
## Phần B: Chữ ký Schnorr, từng dòng
 
//Các kiến thức B1, B2, B3 đã được em tổng hợp ở phần 3 thuật toán cơ bản trong "1 số khái niệm căn bản.md"
### B1. Sinh khóa(key)
 
$$a \in_R [1, n-1], \qquad A = g^a.$$
 
- $\in_R$: "chọn ngẫu nhiên đều trong".
- $a$: **khóa bí mật**. $A$: **khóa công khai**.
- An toàn vì tìm $a$ từ $A$ là bài toán logarit rời rạc (A6).
**Số:** $a = 3$. Tra bảng: $A = 2^3 = 8$.
 
### B2. Ký thông điệp $M$
 
**Dòng 1, commitment (cam kết):**
 
$$y \in_R [1, n-1], \qquad w = g^y.$$
 
$y$ là số bí mật **dùng một lần**. Công khai $w$ không lộ $y$ (lại là logarit rời rạc).
 
**Số:** $y = 5$, tra bảng $w = 2^5 = 9$.
 
**Dòng 2, challenge (thách thức):**
 
$$c = H(M \Vert w).$$
 
$c$ phụ thuộc cả $M$ lẫn $w$. Người ký **không chọn được** $c$, nó do hash quyết định.
 
**Ví dụ số:** giả sử $H$ cho ra $c = 7$.
 
**Dòng 3, response (trả lời):**
 
$$z = y + c \cdot a \bmod n.$$
 
**Ví dụ số:** $z = 5 + 7 \cdot 3 = 26$, và $26 \bmod 11 = 4$. Vậy $z = 4$.
 
**Chữ ký:** $\sigma = (c, z) = (7, 4)$. Người ký gửi $(M, \sigma)$. $y$ và $w$ **không** gửi kèm.
 
### B3. Kiểm tra(Verify)
 
Huy Binh có $A$, $M$, $(c, z)$. Huy Binh tính
 
$$w' = g^z \cdot A^{-c} \pmod p$$
 
rồi kiểm tra $c \stackrel{?}{=} H(M \Vert w')$.
 
**Vì sao $w' = w$ :**
 
$$
\begin{aligned}
g^z \cdot A^{-c}
&= g^{y + ca} \cdot (g^a)^{-c} && \text{(thay } z = y + ca,\ A = g^a\text{)} \\
&= g^{y + ca} \cdot g^{-ca} && \text{(quy tắc } (g^x)^y = g^{xy}\text{)} \\
&= g^{y + ca - ca} && \text{(quy tắc } g^x g^y = g^{x+y}\text{)} \\
&= g^{y} = w.
\end{aligned}
$$
 
(Việc $z$ đã lấy mod $n$ không ảnh hưởng.)
 
Nếu $w' = w$ thì $H(M \Vert w') = c$, chữ ký được chấp nhận.
 
** Ví dụ số, cách 1 (tra bảng nhờ A5):**
 
- $g^z = 2^4 = 16$.
- $A^{-c} = 8^{-7} = (2^3)^{-7} = 2^{-21}$. Số mũ mod 11: $-21 \equiv -21 + 22 = 1$, nên $A^{-c} = 2^1 = 2$.
- $w' = 16 \cdot 2 = 32 \equiv 9 = w$ ✓.
**Số, cách 2 (tính trực tiếp):**
 
- $8^7 = 8^4 \cdot 8^2 \cdot 8 \equiv 2 \cdot 18 \cdot 8 = 288 \equiv 12 \pmod{23}$.
- $12^{-1} \equiv 2$ (A2).
- $w' = 16 \cdot 2 \equiv 9$ ✓.
Hai cách cùng kết quả, minh chứng cho quy tắc A5.
 
### B4. Vì sao kẻ giả mạo khó làm được
 
Kẻ giả mạo không có $a$, muốn tạo $(c, z)$ hợp lệ cho $M^*$ mới:
 
- Muốn tính $w' = g^z A^{-c}$ thì cần $c$ trước, nhưng $c = H(M^* \Vert w')$ lại phụ thuộc $w'$. **Vòng tròn phụ thuộc.**
- Hàm băm "ngẫu nhiên" nên không đoán được. Cách duy nhất là biết $a$.

 
### B5. Vì sao $z$ không lộ $a$
 
$z = y + ca \bmod n$, với $y$ ngẫu nhiên đều trong $\{0, \dots, 10\}$.
 
Cố định $c = 7$, $a = 3$: $ca \bmod 11 = 21 \bmod 11 = 10$. Khi $y$ chạy qua mọi giá trị:
 
| $y$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| $z = y + 10 \bmod 11$ | 10 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
 
$z$ chạy qua **mọi giá trị, mỗi giá trị đúng một lần**: kẻ tấn công thấy một số ngẫu nhiên đều, **bất kể** $a$ là bao nhiêu. Tức dù a có bằng 4, 5 thì z vẫn sẽ chạy ngẫu nhiên đều, có chăng chỉ đổi thứ tự
 
> Điều kiện để che được: **$y$ ngẫu nhiên đều trên toàn bộ $\mathbb{Z}_n$** và phép cộng **quay vòng** (mod $n$). Lattice mất cả hai điều này (Phần D6).
 
---
 
## Phần C: Dùng lại $y$ thì lộ khóa
 
Ký hai thông điệp $M_1, M_2$ với **cùng** $y$: $w$ giống nhau, nhưng $c$ khác:
 
$$z_1 = y + c_1 a \pmod n, \qquad z_2 = y + c_2 a \pmod n.$$
 
Trừ hai phương trình, $y$ triệt tiêu:
 
$$z_1 - z_2 \equiv (c_1 - c_2)\,a \pmod n.$$
 
Nhân hai vế với nghịch đảo của $(c_1 - c_2)$:
 
$$a \equiv (z_1 - z_2) \cdot (c_1 - c_2)^{-1} \pmod n.$$
 
Mọi đại lượng bên phải đều công khai trong chữ ký → kẻ tấn công tính ra $a$. Giải thích sâu hơn: Do w công khai, M là thông điệp cũng thế, nên phép nối chuỗi ra c cũng sẽ cho c công khai.
 

 
---
 
## Phần D: Phiên bản lattice (Dilithium "toy")
 
### D1. Đọc từng ký hiệu
 
| Ký hiệu | Là gì | Trong ML-DSA-44 |
|---|---|---|
| $R_q$ | vành đa thức $\mathbb{Z}_q[x]/(x^n+1)$ | $n = 256$, $q = 8380417$ |
| $q$ | Tập các modulo | Nghĩa là các hệ số của đa thức được tính toán theo modulo |
| $A \in R_q^{k \times \ell}$ | ma trận $k \times \ell$, **mỗi ô là một đa thức** | $4 \times 4$ |
| $s_1 \in S_\eta^{\ell}$ | vector $\ell$ đa thức, mọi hệ số của đa thức trong khoảng $[-\eta, \eta]$ | hệ số trong $[-2, 2]$ |
| $s_2 \in S_\eta^{k}$ | vector $k$ đa thức | như trên |
| $t = A s_1 + s_2$ | khóa công khai, vector $k$ đa thức | |
| $y$ | vector $\ell$ đa thức, hệ số trong $(-\gamma_1, \gamma_1]$ | $\gamma_1 = 2^{17}$ |
| $c \in B_\tau$ | **một** đa thức, đúng $\tau$ hệ số $\pm 1$, còn lại = 0 | $\tau = 39$ (Trên tổng số n = 256 hệ số) |
 
$c$ là **một** đa thức (không phải vector). Nhân $c$ với vector $s_1$ nghĩa là nhân $c$ vào **từng** đa thức của $s_1$.
 
### D2. Các phép toán
 
Nhân ma trận đa thức với vector đa thức giống nhân ma trận số, chỉ khác "nhân hai ô" là **nhân đa thức trong $R_q$** (nhân rồi thay $x^n = -1$):
 
$$(A s_1)_i = A_{i1} s_{1,1} + A_{i2} s_{1,2} + \cdots + A_{i\ell} s_{1,\ell}.$$
 
Hai tính chất dùng ở bước kiểm tra:
 
1. **Phân phối:** $A(u + v) = Au + Av$.
2. **Giao hoán với $c$:** $A(c\,u) = c\,(Au)$, vì nhân đa thức trong $R_q$ có giao hoán.
### D3. Bảng liên hệ Schnorr sang Dilithium toy
 
| | Schnorr | Dilithium toy |
|---|---|---|
| Khóa bí mật | $a$ | $(s_1, s_2)$, **nhỏ** |
| Khóa công khai | $A = g^a$ | $t = A s_1 + s_2$ |
| Commitment | $w = g^y$ | $w = A y$ |
| Challenge | $c = H(M \Vert w)$ | $c = H(M \Vert w) \in B_\tau$ |
| Response | $z = y + ca$ | $z = y + c s_1$ |
| Kiểm tra | $g^z A^{-c} = w$ | $Az - ct = w - c s_2$ |
 
 
### D4. Chứng minh phép kiểm tra, từng bước
 
$$
\begin{aligned}
Az - ct
&= A(y + c s_1) - c\,(A s_1 + s_2) && \text{(thay } z \text{ và } t\text{)} \\
&= Ay + A(c s_1) - c A s_1 - c s_2 && \text{(phân phối, D2.1)} \\
&= Ay + c A s_1 - c A s_1 - c s_2 && \text{(giao hoán với } c\text{, D2.2)} \\
&= Ay - c s_2 \\
&= w - c s_2.
\end{aligned}
$$
 
Khác Schnorr ($g^z A^{-c} = w$ **chính xác**): Huy Binh nhận được $w$ **bị lệch** một lượng $-c s_2$, và không trừ ngược được vì không biết $s_2$.
 
Nhưng lượng lệch **nhỏ**: $c$ có $\tau$ hệ số $\pm 1$, $s_2$ có hệ số trong $[-\eta, \eta]$, nên
 
$$\lVert c\,s_2 \rVert_\infty \le \beta = \tau\eta = 39 \cdot 2 = 78 \quad \ll \quad q \approx 8 \times 10^6.$$
 
### D5. Ví dụ số siêu siêu siêu nhỏ (Thay vì hiểu công thức thì nhìn vô số)
 
Trường hợp **suy biến**: mọi "đa thức" chỉ là **một số** ($n = 1$), $k = \ell = 1$, $q = 97$. (Không an toàn, chỉ minh họa.)
 
**Khóa:**
 
- $A = 30$, $s_1 = 2$, $s_2 = -1$.
- $t = A s_1 + s_2 = 60 - 1 = 59$.
**Ký:**
 
- $y = 5$. $w = A y = 150 \bmod 97 = 53$.
- Giả sử $c = 1$.
- $z = y + c s_1 = 5 + 2 = 7$.
**Kiểm tra:**
 
- $Az - ct = 30 \cdot 7 - 1 \cdot 59 = 210 - 59 = 151 \bmod 97 = 54$.
- Đối chiếu: $w - c s_2 = 53 - (1)(-1) = 54$ ✓.
Huy Bình có **54**, còn $w$ thật là **53**: lệch đúng $-cs_2 = 1$. Đây là **Vấn đề A**.
 
**=> Ý tưởng HighBits**: chia mỗi số theo bước $\alpha$, chỉ giữ "phần cao". Với $\alpha = 10$:
 
$$\mathrm{HighBits}(53) = 5, \qquad \mathrm{HighBits}(54) = 5.$$
 
Lệch một chút nhưng **cùng phần cao**. Nếu cả người ký và Bob đều băm $\mathrm{HighBits}(w)$ thay cho $w$, Bob vẫn ra đúng $c$.
 
Rủi ro: nếu $w$ nằm sát ranh giới, ví dụ $59$ bị lệch thành $60$, phần cao đổi từ 5 sang 6. Vì vậy người ký **kiểm tra trước** $w - cs_2$ không sát ranh giới, nếu sát thì ký lại. Đây là điều kiện "LowBits đủ nhỏ" trong Algorithm 7.1.
 
### D6. Vấn đề B: $z$ lộ thông tin về $s_1$
 
Ở Schnorr (B5), $y$ đều trên **toàn bộ** $\mathbb{Z}_n$ và phép cộng quay vòng, nên $z$ che hoàn hảo.
 
Ở lattice, $y$ **bắt buộc nhỏ** (hệ số trong $(-\gamma_1, \gamma_1]$): nếu $y$ lớn tùy ý thì $z$ lớn tùy ý, bài toán giả mạo trở nên dễ, mất an toàn MSIS. Mà $y$ nhỏ thì $z = y + cs_1$ **không quay vòng**, phân bố của $z$ bị "dịch" theo $cs_1$.
 
**Ví dụ một chiều:** $y$ đều trong $\{-3, \dots, 3\}$, một hệ số của $cs_1$ là $u$.
 
- Nếu $u = +1$: $z$ đều trong $\{-2, \dots, 4\}$.
- Nếu $u = -1$: $z$ đều trong $\{-4, \dots, 2\}$.
Thấy $z = 4$ → biết chắc $u = +1$. Thấy $z = -4$ → biết chắc $u = -1$. Thu nhiều chữ ký sẽ ghép ra $s_1$.
 
**Cách sửa:** chỉ công bố $z$ khi nằm trong vùng an toàn $\{-2, \dots, 2\}$ (vùng mà cả hai trường hợp đều sinh ra với cùng xác suất). Rơi ra ngoài thì bỏ, ký lại với $y$ mới. Đó là **rejection sampling** 
 