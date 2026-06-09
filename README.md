# Privacy-Preserving Machine Learning with Fully Homomorphic Encryption (FHE)

> Đồ án môn **Mã hóa và Thám mã** — Chương trình Thạc sĩ Công nghệ Thông tin,
> Trường Đại học Công nghệ TP.HCM (HUTECH), 08/2025.

Tìm hiểu và demo **Privacy-Preserving Machine Learning (PPML)** dựa trên **Fully
Homomorphic Encryption (FHE)** — kỹ thuật cho phép **tính toán trực tiếp trên dữ liệu đã
mã hóa** mà không cần giải mã. Tập trung vào **inference trên ciphertext** bằng thư viện
**Concrete-ML** (Zama).

> Bài toán: *"Có thể dự đoán/huấn luyện mà không tiết lộ dữ liệu gốc?"* — quan trọng với dữ
> liệu nhạy cảm (y tế, tài chính, quốc phòng) và tuân thủ GDPR/HIPAA.

## Cơ sở lý thuyết

**Các kỹ thuật PPML:**

| Kỹ thuật | Mã hóa? | Đặc điểm |
|----------|---------|----------|
| Remote Execution | có thể | Chạy model từ xa |
| Federated Learning | thường không | Huấn luyện phân tán, dữ liệu ở tại chỗ (rủi ro rò rỉ gradient) |
| **Secure Computation (FHE/MPC)** | **có** | Tính toán không thấy dữ liệu — bảo mật cao, hiệu năng thấp |
| Differential Privacy | không bắt buộc | Thêm nhiễu — nhanh nhưng giảm độ chính xác |

**Fully Homomorphic Encryption (FHE):**
- Dựa trên **homomorphism** — phép toán trên plaintext được bảo toàn trên ciphertext:
  `Enc(a) ⊕ Enc(b) = Enc(a+b)`, `Enc(a) ⊗ Enc(b) = Enc(a·b)`.
- Phân cấp: **PHE** (1 phép toán) → **SHE** (giới hạn số phép toán do noise) → **FHE** (cộng & nhân không giới hạn).
- Mốc: Gentry (2009) — lược đồ FHE đầu tiên; thế hệ sau **BGV, BFV, CKKS, TFHE**.
- Mô hình public-key: `Dec_sk( Eval_evk(f, Enc_pk(x)) ) = f(x)` — server chỉ giữ evaluation key, **không bao giờ thấy dữ liệu gốc**.
- **FHE vs E2EE:** E2EE bảo vệ dữ liệu *khi truyền*; FHE bảo vệ dữ liệu *khi xử lý*.

## Tổng quan nghiên cứu (literature)

- **Fang & Qian (2021)** — PFMLP: HE (Paillier) + Federated Learning cho training, mã hóa gradient.
- **Lee et al. (2022)** — FHE (RNS-CKKS + bootstrapping, SEAL) cho inference DNN (ResNet-20), accuracy ~92%.

## Công cụ: Concrete-ML (Zama)

Thư viện mã nguồn mở: train trên cleartext, **inference trên FHE**, không cần kiến thức mật mã
sâu. Tích hợp scikit-learn / PyTorch / TensorFlow / ONNX với API quen thuộc (`fit`, `predict`,
`pipeline`, `gridsearch`). FHE chỉ làm việc trên **số nguyên** → cần **quantization**.

```
Data → (Quantize) Integers → (Encrypt) Ciphertext
     → (Inference in FHE) Encrypted result → (Decrypt) Output
```

## Demo

| # | Demo | Mô hình | Kết quả |
|---|------|---------|---------|
| 1 | **Iris Classification** | MLP (`NeuralNetClassifier`, 2×16 hidden, softmax 3 lớp) | accuracy ≈ **96.6%** |
| 2 | **Sentiment Analysis** (Twitter airline) | XGBoost (`Concrete-ML XGBClassifier` + GridSearch) | accuracy ≈ **84–85%**; compile ~9.3s; FHE inference ~4.4s/tweet |
| 3 | **Image Filtering** | Bộ lọc ảnh chạy trên ciphertext | client mã hóa → server lọc (FHE) → client giải mã |

Mỗi demo theo pipeline **client–server**: client tạo khóa + mã hóa input → server `Eval` trên
ciphertext → client giải mã kết quả. Kết quả FHE **tương đương** inference trên dữ liệu rõ.

## Cài đặt & chạy

```bash
pip install concrete-ml scikit-learn xgboost
# Chạy từng demo (Iris / Sentiment / Image Filtering) — xem thư mục demos/
python demos/iris_mlp_fhe.py
```

> Concrete-ML yêu cầu Linux/macOS; cập nhật tên file demo cho khớp repo.

## Kết luận

- FHE cho phép tính toán trên dữ liệu mã hóa → bảo mật tối đa trong môi trường zero-trust.
- Phù hợp nhất cho **inference-as-a-service** trên dữ liệu nhạy cảm; Concrete-ML giúp triển khai dễ dàng.
- **Hạn chế:** chậm, tốn bộ nhớ, khó cho deep learning phức tạp hoặc training trực tiếp trên ciphertext.

### Hướng phát triển
Tăng tốc bằng GPU/FPGA · kết hợp **FHE + Federated Learning / Differential Privacy** ·
lượng tử hóa & xấp xỉ hàm phi tuyến tốt hơn cho deep learning.

## Tài liệu tham khảo

- Zama — **Concrete-ML** (github.com/zama-ai/concrete-ml).
- L. Maggio — *PPML: Machine Learning on Data You Cannot See*.
- Fang & Qian (2021), *Future Internet* 13(4):94.
- Lee, J.-W. et al. (2022), *IEEE Access* 10:30039–30054.

## Nhóm thực hiện

Nguyễn Ngọc Đỉnh · Hà Anh Dũng · Nguyễn Minh Trung Nghĩa — GVHD: TS. Võ Văn Khang
