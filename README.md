# Đề 7 · Chuyển khoản qua app ngân hàng

> **Mã:** DE07 · **Độ khó:** 7/8 · **Vai trò:** 1 (khách hàng chuyển tiền) · **Nền tảng:** App ngân hàng iOS / Android

Tính năng chuyển khoản bằng số tài khoản hoặc số thẻ trong app ngân hàng, theo đúng các bước người dùng làm: **chọn ngân hàng → nhập số tài khoản → chọn số tiền → xác nhận, xem lại và xác thực chuyển khoản → hoàn thành chuyển khoản**. Đề khó hơn Đề 4 vì có nhiều ngân hàng nhận, tra tên liên ngân hàng, hai hình thức chuyển (nhanh 24/7 và thường), phí, hạn mức, cảnh báo lừa đảo và nhiều trạng thái kết quả (đã tiếp nhận, chưa xác định). Đề gồm **1 tính năng**, là **1 luồng** gồm 5 bước và các màn hình; tài liệu gồm 1 PRD và 1 SRS.

## Tính năng

| **Tính năng** | **Luồng** | **PRD** | **SRS** |
| --- | --- | --- | --- |
| Chuyển khoản qua app ngân hàng | Chọn ngân hàng → Nhập số tài khoản → Chọn số tiền → Xác nhận & xác thực chuyển khoản → Hoàn thành chuyển khoản | [PRD](docs/PRD.md) | [SRS](docs/SRS.md) |

## Các bước và màn hình của luồng

| **Bước** | **Tên bước** | **Màn hình** | **Phạm vi** |
| --- | --- | --- | --- |
| Bước 1 | Chọn ngân hàng | SCR-01, SCR-02 | Màn chuyển tiền, đi tắt từ gần đây, danh bạ, tài khoản của tôi; danh sách ngân hàng, hay dùng, tìm, trạng thái bảo trì |
| Bước 2 | Nhập số tài khoản | SCR-03 | Số tài khoản hoặc số thẻ, dán, tra tên chủ tài khoản, thẻ không khớp ngân hàng, cảnh báo tài khoản |
| Bước 3 | Chọn số tiền | SCR-04 | Số tiền bằng chữ, nội dung, tài khoản nguồn, chuyển nhanh hoặc thường, phí, thời gian tới, hạn mức |
| Bước 4 | Xác nhận & xác thực chuyển khoản | SCR-05, SCR-06 | Bấm Xác nhận, xem lại, cảnh báo lừa đảo và chuyển trùng, PIN / sinh trắc / khuôn mặt, gửi lệnh |
| Bước 5 | Hoàn thành chuyển khoản | SCR-07, SCR-08 | Đang xử lý, thành công, đã tiếp nhận, thất bại, chưa xác định, huỷ lệnh chuyển thường, biên lai, lưu danh bạ, chi tiết |

Luồng: Bước 1 Chọn ngân hàng → Bước 2 Nhập số tài khoản → Bước 3 Chọn số tiền → Bước 4 Xác nhận & xác thực chuyển khoản → Bước 5 Hoàn thành chuyển khoản.

## Luồng chuyển khoản

```mermaid
flowchart LR
  A["Trang chủ, Chuyển tiền"] --> B["Chọn ngân hàng, Bước 1"]
  B --> C["Nhập số tài khoản, tra tên, Bước 2"]
  C --> D["Chọn số tiền, Bước 3"]
  D --> E["Bấm Xác nhận, xem lại giao dịch, Bước 4"]
  E --> F["Xác thực PIN, sinh trắc, khuôn mặt, Bước 4"]
  F --> G["Hoàn thành, Bước 5"]
  A -->|Gần đây, danh bạ| D
```

Người nhận quen (Gần đây, Danh bạ) và tài khoản của chính khách đi tắt từ màn chuyển tiền tới bước chọn số tiền (Bước 1 BR-07, BR-08).

## Vòng đời giao dịch (dùng chung cả đề)

```mermaid
stateDiagram-v2
  [*] --> NHAP : Người nhận đã xác minh, Bước 1, Bước 2
  NHAP --> NHAP : Chọn số tiền, Bước 3
  NHAP --> DA_BO : Thoát, dừng ở cảnh báo, Bước 3, Bước 4
  NHAP --> BI_TU_CHOI : Rủi ro bị chặn, Bước 4
  NHAP --> CHO_XAC_THUC : Bấm Chuyển tiền ở màn xem lại, Bước 4
  CHO_XAC_THUC --> DA_BO : Hết 3 phút
  CHO_XAC_THUC --> BI_TU_CHOI : Sai xác thực quá số lần
  CHO_XAC_THUC --> DANG_XU_LY : Xác thực, gửi lệnh nhanh hoặc nội bộ, Bước 4
  CHO_XAC_THUC --> DA_TIEP_NHAN : Xác thực, gửi lệnh chuyển thường, Bước 4
  DA_TIEP_NHAN --> DANG_XU_LY : Tới khung giờ xử lý, Bước 5
  DA_TIEP_NHAN --> DA_HUY : Khách huỷ lệnh, Bước 5
  DANG_XU_LY --> THANH_CONG : Ngân hàng nhận ghi có, Bước 5
  DANG_XU_LY --> THAT_BAI : Bị từ chối, hoàn tiền nếu đã trừ, Bước 5
  DANG_XU_LY --> CHUA_XAC_DINH : Quá 60 giây, Bước 5
  CHUA_XAC_DINH --> THANH_CONG : Đối soát
  CHUA_XAC_DINH --> THAT_BAI : Đối soát, hoàn tiền
  THANH_CONG --> [*]
  THAT_BAI --> [*]
  DA_HUY --> [*]
  DA_BO --> [*]
  BI_TU_CHOI --> [*]
```

## Bản đồ màn hình

```mermaid
flowchart LR
  S01["SCR-01 Chuyển tiền"] --> S02["SCR-02 Chọn ngân hàng"]
  S02 --> S03["SCR-03 Nhập số tài khoản"]
  S03 -->|Đổi ngân hàng| S02
  S03 --> S04["SCR-04 Chọn số tiền"]
  S01 -->|Gần đây, danh bạ| S04
  S04 --> S05["SCR-05 Xem lại giao dịch"]
  S05 -->|Sửa| S04
  S05 --> S06["SCR-06 Xác thực"]
  S06 --> S07["SCR-07 Kết quả & biên lai"]
  S07 -->|Giao dịch mới| S01
  S07 --> S08["SCR-08 Chi tiết giao dịch"]
  S08 -->|Chuyển lại| S04
```

## Bối cảnh chung

Chuyển khoản bằng số tài khoản dùng cho khoản lớn và quan trọng: tiền nhà, học phí, tiền hàng, tiền cọc. Khách cần chọn đúng ngân hàng, nhập đúng số tài khoản, thấy tên người nhận thật, nhập đúng số tiền, biết phí và **biết khi nào tiền tới**. Đây cũng là kênh chính của lừa đảo, khi nạn nhân tự tay xác thực theo hướng dẫn của kẻ gian. Bài toán thiết kế chính là giữ mỗi bước rõ ràng, khó nhầm, cảnh báo đúng lúc, và tuân thủ quy định xác thực của Ngân hàng Nhà nước.

| **Nhóm người dùng** | **Nhu cầu chính** |
| --- | --- |
| Người chuyển định kỳ (tiền nhà, học phí) | Đi tắt người nhận quen, biên lai gửi người nhận |
| Chủ hộ kinh doanh | Chuyển khoản lớn, biết hình thức, phí, thời gian tới |
| Người nhận số tài khoản qua tin nhắn | Dán số, thấy tên người nhận ngay |
| Người lớn tuổi, dễ bị lừa | Chữ to, số tiền bằng chữ, cảnh báo rõ |

Ngoài phạm vi cả đề: mở tài khoản, đăng nhập và cài Smart OTP, đăng ký sinh trắc khuôn mặt (chỉ dẫn link), chuyển khoản QR (xem Đề 4), ví điện tử, số điện thoại, ngoại tệ và quốc tế, hẹn lịch và định kỳ, chuyển theo lô, lịch sử giao dịch đầy đủ, tra soát, quản lý danh bạ, đổi hạn mức.

## Quy ước dùng chung

- Tiền tệ VND, không có số lẻ; định dạng `12.500.000 ₫`; số tiền luôn kèm cách đọc bằng chữ ở màn nhập và xác nhận.
- Ngày làm việc theo lịch ngân hàng (thứ Hai – thứ Sáu, trừ ngày lễ); mọi giờ là giờ Việt Nam.
- Ngưỡng xác thực, hạn mức, phí là **cấu hình**; giá trị trong tài liệu là giá trị hiện hành (ngưỡng xác thực theo Quyết định 2345/QĐ-NHNN).
- Số tổng đài hiển thị dạng `{số}`.
- Mã BR, MSG, UC đánh số liên tục trong cả đề: Bước 1 dùng BR-01–08, MSG-01–05, UC-01–02; Bước 2 dùng BR-09–17, MSG-06–12, UC-03; Bước 3 dùng BR-18–28, MSG-13–20, UC-04; Bước 4 dùng BR-29–38, MSG-21–30, UC-05–06; Bước 5 dùng BR-39–48, MSG-31–39, UC-07–08.

## Danh sách màn hình

| **Mã** | **Màn hình** | **Bước** | **Mô tả chi tiết ở** |
| --- | --- | --- | --- |
| SCR-01 | Chuyển tiền (bắt đầu) | Chọn ngân hàng | Bước 1 |
| SCR-02 | Chọn ngân hàng | Chọn ngân hàng | Bước 1 |
| SCR-03 | Nhập số tài khoản | Nhập số tài khoản | Bước 2 |
| SCR-04 | Chọn số tiền | Chọn số tiền | Bước 3 |
| SCR-05 | Xem lại giao dịch (gồm màn cảnh báo) | Xác nhận & xác thực | Bước 4 |
| SCR-06 | Xác thực giao dịch | Xác nhận & xác thực | Bước 4 |
| SCR-07 | Kết quả & biên lai | Hoàn thành | Bước 5 |
| SCR-08 | Chi tiết giao dịch | Hoàn thành | Bước 5 |

## Cấu trúc repo

```text
de07-chuyen-khoan-ngan-hang/
├── README.md
└── docs/
    ├── PRD.md    Bối cảnh, phạm vi, luồng, use case theo từng bước
    └── SRS.md    Quy tắc, trạng thái, use case, màn hình, API, thông báo theo từng bước
```

## Cách dùng bộ tài liệu

1. Đọc README để nắm luồng 5 bước và vòng đời giao dịch.
2. Đọc PRD để nắm toàn bộ tính năng và luồng, rồi đọc SRS theo từng bước của luồng. Thiết kế đủ trạng thái lỗi của từng bước (ngân hàng bảo trì, không tìm thấy tài khoản, thiếu số dư, chỉ chuyển thường, cảnh báo, sai PIN, đã tiếp nhận, chưa xác định).
3. Ghép các màn thành một prototype liền mạch từ **SCR-01** tới **SCR-07**, cả luồng người nhận mới và luồng đi tắt.
4. Thiết kế màn hình theo đúng mã SCR, mã MSG trong SRS để dễ đối chiếu khi review.
5. Sơ đồ viết bằng Mermaid, xem trực tiếp trên GitHub hoặc trình xem Markdown có hỗ trợ Mermaid.
