# Ghi chú: Blockchain → Mật mã học → PQC → Nền tảng cho ML-DSA (Dilithium)
 
---
 
## 1. Blockchain
 
### Hình dung chung :p
Cuốn sổ ghi chép các giao dịch mà **mọi người cùng giữ một bản giống hệt nhau**. Không ai nắm sổ gốc. Ghi vào rồi thì khó sửa.
 
### Các thành phần
| Thành phần | Vai trò | Ví dụ liên tưởng cho dễ hiểu xd |
|---|---|---|
| Sổ phân tán | Không ai nắm quyền một mình | Cả làng giữ sổ |
| Hàm băm (hash) | Phát hiện sửa đổi | Dấu vân tay |
| Chuỗi khối | Khối sau chứa hash của khối trước | Niêm phong nối tiếp |
| Đồng thuận | Quyết định ai ghi khối tiếp theo | Thi đấu / bỏ phiếu |
| Chữ ký số | Chứng minh quyền sở hữu | Con dấu cá nhân |
 
### Tại sao khó sửa dữ liệu đã ghi
Sửa giao dịch ở khối $k$ → hash khối $k$ đổi → khối $k+1$ ghi hash cũ nên khớp sai → phải tính lại **tất cả** khối sau, trong khi cả mạng vẫn đang thêm khối mới. Như vậy, khối càng cũ càng an toàn.
 
### Đồng thuận
- **Proof of Work** (Bitcoin): giải câu đố tính toán tốn điện, gian lận rất đắt.
- **Proof of Stake** (Ethereum): đặt cọc tiền, gian lận bị mất cọc.
### Liên hệ lượng tử
- **Shor** phá ECC/RSA → phá **chữ ký số** (từ khóa công khai suy ra khóa riêng). Đây là mối đe dọa lớn nhất với blockchain.
- **Grover** làm giảm độ an toàn hash và tăng tốc đào theo bậc hai, nhưng tác động nhẹ hơn nhiều.
---
 
## 2. Mật mã học (Cryptography) nền tảng
 
### Bốn mục tiêu
| Mục tiêu | Ý nghĩa | Công cụ |
|---|---|---|
| Bảo mật | Kẻ nghe lén không đọc được | Mã hóa |
| Toàn vẹn | Nội dung không bị sửa lén | Hash, MAC |
| Xác thực | Biết đúng người gửi | MAC, chữ ký số |
| Chống chối bỏ | Người ký không thể chối | Chữ ký số |
 
**Nguyên tắc Kerckhoffs:** độ an toàn nằm ở **khóa**, không phải việc giấu thuật toán.
Tức 1 công ty có thể public ra đại chúng thuật toán để mã hoá là gì, bởi độ an toàn nằm ở bản thân cách phá khoá khó.
 
### Đối xứng
Một khóa chung $K$:
 
$$C = \mathrm{Enc}(K, M), \qquad M = \mathrm{Dec}(K, C)$$
 
Ví dụ: AES (khóa 128/256 bit): rất nhanh, an toàn, dùng để mã hóa dữ liệu thực tế.
 
Vấn đề: làm sao hai người chia sẻ khóa $K$ lần đầu qua kênh không an toàn? Và nếu có $n$ người, cần $\dfrac{n(n-1)}{2}$ khóa. Đây là lý do cần mật mã bất đối xứng.
 
### Hash và MAC
- **Hash** (SHA-256, SHA-3), không có khóa. Ba tính chất:
  - kháng tiền ảnh: từ $h$ không tìm lại được $M$ sao cho $H(M) = h$;
  - kháng tiền ảnh thứ hai: cho $M$, không tìm được $M' \neq M$ với $H(M') = H(M)$;
  - kháng va chạm: không tìm được cặp $M \neq M'$ với $H(M) = H(M')$.
- **MAC** (HMAC): hash có khóa chung, $\mathrm{Tag} = \mathrm{MAC}(K, M)$. Người nhận có $K$ tự tính lại và so sánh, nên biết thông điệp đúng và đến từ người có $K$. Cả hai bên đều có $K$ nên không chứng minh được cho bên thứ ba ai là người tạo → không chống chối bỏ.
### Bất đối xứng (khóa công khai)
Mỗi người có một cặp khóa: khóa công khai ($pk$, cho mọi người) và khóa riêng ($sk$, giữ bí mật). Hai khóa liên hệ toán học, nhưng từ $pk$ không suy ra được $sk$.
- **RSA**: nhân $p \cdot q$ dễ, nhưng phân tích $N = p \cdot q$ thành thừa số thì khó.
- **Logarit rời rạc / ECC**: tính $g^x \bmod p$ dễ, tìm $x$ từ $g^x$ thì khó.
**Diffie-Hellman (ví dụ pha màu):** hai bên trộn màu chung với màu bí mật rồi trao đổi, trộn thêm màu bí mật của mình → ra cùng một màu chung. Bên tấn công sẽ không "tách màu" được (tương ứng logarit rời rạc). Dạng số học:
 
$$A = g^a,\quad B = g^b,\qquad K = B^a = A^b = g^{ab} \pmod p$$
 
### Chữ ký số
Ba thuật toán chính:
- $\mathrm{KeyGen} \to (pk, sk)$
- $\mathrm{Sign}(sk, M) \to \sigma$
- $\mathrm{Verify}(pk, M, \sigma) \to$ accept / reject
Thực tế ký hash của $M$: $\sigma = \mathrm{Sign}(sk, H(M))$. Khác MAC: ai cũng kiểm tra được bằng $pk$, chỉ người có $sk$ mới tạo được → **có chống chối bỏ**.
 
**EUF-CMA:**
Tính an toàn EUF-CMA (Existential Unforgeability under Chosen-Message Attack), cụ thể:
- Chosen-message attack: kẻ tấn công được tự chọn nhiều thông điệp $M_1, \dots, M_Q$ và xin người ký ký giúp (signing oracle).
- Existential forgery: sau đó, chỉ cần giả mạo được chữ ký hợp lệ $(M^*, \sigma^*)$ cho một thông điệp $M^* \notin \{M_1, \dots, M_Q\}$ là kẻ tấn công thắng. $M^*$ có vô nghĩa cũng tính.
### Thực tế dùng kết hợp (TLS)
Chữ ký + chứng chỉ (xác thực server) → trao đổi khóa DH/ECDH → AES + MAC mã hóa dữ liệu bằng khóa phiên.
 
### Tác động lượng tử
| Thành phần | Dựa trên | Tác động | Kết luận |
|---|---|---|---|
| RSA, DH, ECC, ECDSA | Thừa số, logarit rời rạc | Shor phá hoàn toàn | Phải thay thế |
| AES | Vét cạn khóa | Grover: còn khoảng $\tfrac12$ số bit | Dùng AES-256 |
| SHA-256 | Tìm tiền ảnh/va chạm | Grover giảm độ an toàn | Tăng độ dài hash |
 
---
 
## 3. Hậu lượng tử (PQC) vs Mật mã lượng tử
 
**Chữ ký hậu lượng tử KHÔNG dùng cơ học lượng tử.** "Hậu lượng tử" nghĩa là *an toàn ngay cả khi kẻ tấn công có máy tính lượng tử*.
 
| | PQC | Mật mã lượng tử (QKD...) |
|---|---|---|
| Chạy trên | Máy tính **cổ điển** | Phần cứng lượng tử (photon, qubit) |
| Dựa trên | Bài toán toán học khó | Định luật vật lý |
| Triển khai | Cập nhật phần mềm | Thiết bị chuyên dụng |
| Ví dụ | ML-DSA, SLH-DSA, ML-KEM | BB84, E91 |
 
Lượng tử liên quan ở phía kẻ tấn công: Shor phá bài toán cũ → chuyển sang bài toán chưa có thuật toán lượng tử hiệu quả (lattice, hash).
 
=> PQC học ở góc nhìn người phòng thủ
 
### Các chuẩn NIST
- **ML-KEM** (Kyber): trao đổi khóa, lattice (FIPS 203).
- **ML-DSA** (Dilithium): chữ ký, lattice (FIPS 204).
- **SLH-DSA** (SPHINCS+): chữ ký, dựa trên hash (FIPS 205).
- **FN-DSA** (Falcon): chữ ký lattice gọn hơn, chuẩn đang hoàn thiện (FIPS 206).
---
 
## 4. Shor và Grover
 
### Shor (phá bài toán có tính tuần hoàn)
Chuỗi quy dẫn: **Phân tích thừa số → Tìm bậc → Ước lượng pha → QFT**
 
1. Chọn $x$ ngẫu nhiên, tìm bậc $r$ nhỏ nhất sao cho
   $$x^r \equiv 1 \pmod N.$$
   Nếu $r$ chẵn và $x^{r/2} \not\equiv -1 \pmod N$ thì $\gcd\left(x^{r/2} \pm 1,\ N\right)$ cho ước số không tầm thường.
2. Phần lượng tử: Hadamard tạo chồng chập, tính $x^a \bmod N$ song song, chu kỳ $r$ được mã hóa vào **pha**, QFT đảo ngược ($\mathrm{QFT}^\dagger$) làm đỉnh xác suất rơi gần $\dfrac{s}{r}$.
3. Phần cổ điển: phân số liên tục khôi phục $r$ từ giá trị đo $\dfrac{\varphi}{2^t} \approx \dfrac{s}{r}$.
**Ví dụ $N=15$, $x=7$:** $7^a \bmod 15 = 7, 4, 13, 1$ → $r=4$. Khi đó $7^2 = 49$:
 
$$\gcd(48, 15) = 3, \qquad \gcd(50, 15) = 5 \;\Rightarrow\; 15 = 3 \times 5.$$
 
Độ phức tạp đa thức (cỡ $O(L^3)$, $L$ là số bit của $N$). Logarit rời rạc cũng tuần hoàn → Shor phá luôn DH, ECC, ECDSA.
 
### Grover (tăng tốc bậc hai)
Tìm $x$ thỏa $f(x)=1$ trong $N$ phần tử không cấu trúc ($M$ nghiệm). Mỗi vòng:
1. **Oracle:** đảo dấu biên độ nghiệm, $O\lvert x \rangle = (-1)^{f(x)}\lvert x \rangle$.
2. **Diffusion:** phản xạ biên độ qua giá trị trung bình, $a_x \mapsto 2\langle a \rangle - a_x$ → nghiệm tăng lên.
Cần
 
$$\approx \frac{\pi}{4}\sqrt{\frac{N}{M}} \text{ vòng.}$$
 
**Lặp quá số vòng tối ưu thì xác suất giảm lại.**
 
### So sánh tác động
| | Shor | Grover |
|---|---|---|
| Nhắm vào | Bài toán có cấu trúc | Vét cạn |
| Với RSA/ECC | Phá hoàn toàn | |
| Với AES, hash | | Giảm khoảng một nửa số bit hiệu dụng |
| Ứng phó | **Thay thuật toán** (PQC) | **Tăng tham số** |
 
---
 
## 5. Nền tảng 4 lớp (tóm tắt)
 
### Lớp 1: Chữ ký số và EUF-CMA
Mỗi chữ ký Dilithium "rò" chút thông tin về khóa bí mật. Rejection sampling, HighBits... tồn tại để xem nhiều chữ ký không giúp kẻ tấn công = bảo đảm EUF-CMA.
 
### Lớp 2: Lattice và SVP
- **Lattice** = mọi tổ hợp tuyến tính **nguyên** của các vector độc lập $b_1, \dots, b_n$ (basis):
  $$L(B) = \{\, c_1 b_1 + \cdots + c_n b_n : c_i \in \mathbb{Z} \,\} = \{\, Bx : x \in \mathbb{Z}^n \,\}$$

  Dễ hiểu: Lattice - mạng tinh thể là một tập hợp các điểm phân bố đều đặn trong không gian nhiều chiều.

  Ví dụ, trong mặt phẳng 2D có 2 vector cơ s(basis) là b1 và b2. Từ 2 thằng này có thể tạo vô hạn điểm v = x1b1 + x2b2. Tập các điểm v là lattice.
- Một lattice có **vô số basis**: $B' = B\,U$, $U$ nguyên với $\det U = \pm 1$ (unimodular, Định lý 2.9).
- **Basis tốt** (ngắn, gần vuông góc) → bài toán dễ. **Basis xấu** → khó. Mật mã lattice khai thác bất đối xứng này.
- $\mathrm{vol}(L) = \lvert \det B \rvert$ bất biến theo basis. $\lambda_1(L)$ = độ dài vector khác 0 ngắn nhất.
- **Gaussian heuristic:**
  $$\lambda_1(L) \approx \sqrt{\frac{n}{2\pi e}}\;\mathrm{vol}(L)^{1/n}$$


- **SVP(Shortest Vector Problem)**: tìm vector ngắn nhất. Hãy tưởng tượng mình có một lattice với hàng triệu điểm. Nhiệm vụ của mình là tìm điểm khác gốc tọa độ gần gốc tọa độ nhất.

   **approx-SVP**: tìm $v \neq 0$ với $\lVert v \rVert \le \gamma\,\lambda_1(L)$. **CVP**: điểm lattice gần $t$ nhất.

   Tại sao SVP khó? Khi nghe mô tả có vẻ dễ, chỉ cần tìm khoảng cách và xem vector nào ngắn nhất. Có một điểm dễ gây nhầm lẫn: Tìm các vector cơ sở không đồng nghĩa với tìm được vector ngắn nhất trong lattice.

   một lattice có thể có basis rất dài nhưng vẫn chứa những vector cực ngắn.
  Trong không gian nhiều chiều, việc xác định vector ngắn nhất trở thành một bài toán tính toán rất khó.

#### SVP liên quan gì mật mã? 
  Ý tưởng cơ bản của lattice-based cryptography là:
1. Xây dựng một cấu trúc toán học liên quan đến lattice.
2. Tạo ra một bài toán mà người dùng hợp lệ có thể giải hiệu quả nhờ thông tin bí mật.
3. Kẻ tấn công chỉ có thông tin công khai và phải giải một bài toán lattice khó để phá hệ thống.

Có thể hiểu theo cách sau:
- EUF-CMA là tiêu chuẩn mà ta muốn hệ thống đạt được.
- Lattice và các bài toán liên quan cung cấp nền tảng toán học để xây dựng hệ thống.
- SVP giúp ta hiểu tại sao việc giải các bài toán lattice có thể rất khó.
- Dilithium là một ứng dụng cụ thể kết hợp những ý tưởng này.
### Lớp 3: SIS và LWE
**SIS:** cho $A \in \mathbb{Z}_q^{n \times m}$ ngẫu nhiên ($m > n$), tìm $z \neq 0$ **ngắn** với
 
$$A z \equiv 0 \pmod q.$$
 
- Tìm *một* nghiệm thì dễ (khử Gauss); cái khó là nghiệm ngắn.
- Các nghiệm tạo lattice $L_A^{\perp} = \{\, z \in \mathbb{Z}^m : Az \equiv 0 \pmod q \,\}$, với $\mathrm{vol}(L_A^{\perp}) = q^n$ (Định lý 3.10) → tìm nghiệm ngắn = (approx-)SVP.
**LWE:** chọn bí mật $s$, ma trận $A$, nhiễu nhỏ $e$. Công bố
 
$$b = A s + e \pmod q,$$
 
tìm $s$.
- $e = 0$: khử Gauss giải ngay. Có nhiễu nhỏ: rất khó.
- **DLWE:** phân biệt $(A,\ As+e)$ với $(A,\ r)$ ngẫu nhiên. Định lý 4.6: LWE $\Leftrightarrow$ DLWE.
- Góc nhìn lattice: $b$ gần điểm lattice $As$ → CVP/BDD → uSVP (primal attack).
- **ss-LWE:** bí mật $s$ cũng nhỏ. Dilithium dùng dạng này.

| | SIS | LWE |
|---|---|---|
| Cần tìm | Vector ngắn $z$, $Az \equiv 0$ | Bí mật $s$ trong $b = As + e$ |
| Vai trò trong Dilithium | An toàn **chữ ký** (chống giả mạo) | An toàn **khóa** |
 
### Lớp 4: Vành $R_q$, MSIS, MLWE
- **Vì sao:** $A$ cỡ $n \times m$ lớn quá, khóa rất nặng. Thêm cấu trúc đại số để mô tả gọn.
- **Vành đa thức:**
  $$R_q = \mathbb{Z}_q[x]/(x^n + 1)$$
  đa thức bậc $< n$, hệ số mod $q$, nhân xong thay $x^n = -1$.
- Nhân với $x$ = **dịch vòng và đổi dấu** một phần tử:
  $$(a_0, a_1, \dots, a_{n-1}) \;\longmapsto\; (-a_{n-1}, a_0, \dots, a_{n-2})$$
  Nhân đa thức = nhân ma trận **phản-vòng** chỉ xác định bởi $n$ số.
- **Kích thước:** $\lVert f \rVert_\infty = \max_i \lvert f_i \rvert$ (biểu diễn đối xứng). **Định lý 5.7:** tích hai đa thức nhỏ vẫn nhỏ: nếu $\lVert f \rVert_\infty \le \eta_1$, $\lVert g \rVert_\infty \le \eta_2$ thì $\lVert fg \rVert_\infty \le n\,\eta_1\eta_2$.
- **Module** $R_q^k$: vector $k$ đa thức. Ma trận $k \times \ell$ chỉ cần $k \cdot \ell \cdot n$ số.
- **MSIS:** cho $A \in R_q^{k \times \ell}$, tìm $z$ nhỏ, $z \neq 0$, với $Az = 0$.
- **MLWE:** cho $A$ và $t = As + e$ ($s, e$ nhỏ), tìm $s$.
- Nếu $z$ là nghiệm MSIS thì $x \cdot z$ cũng là nghiệm (sinh thêm $2n-1$ nghiệm) — hệ quả của cấu trúc.
- **Đánh đổi:** cấu trúc → khóa nhỏ, nhanh (NTT), nhưng bài toán *có thể* dễ hơn LWE không cấu trúc. FrodoKEM (không cấu trúc) là phương án bảo thủ. Nghiên cứu được sách trích (2026) ước tính MLWE có thể yếu hơn LWE khoảng 1 bit.
- **Trong Dilithium:** khóa công khai
  $$t = A s_1 + s_2$$
  là mẫu MLWE; chống giả mạo dựa trên MSIS. ML-DSA dùng $n = 256$, $q = 2^{23} - 2^{13} + 1$.
### Bản đồ nối các lớp
```
EUF-CMA (mục tiêu an toàn)
      ↑ chứng minh bằng reduction
MSIS + MLWE ← thêm cấu trúc vành đa thức ← SIS + LWE
                                               ↑ hiểu như
                                       SVP/CVP trong lattice
```
 