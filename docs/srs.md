# SRS rút gọn – Smart CRM Mekong Mobile
## Luồng L8: Khảo sát hài lòng sau bảo hành

- **Họ tên / MSSV:** …………………
- **Track:** DA (Data requirement specification) — *giả định; đổi sang SE nếu cần, xem Phụ lục C*
- **Phiên bản:** 0.1 · Buổi 4 · HK1 2026–2027

**Phạm vi một câu:** Thu thập phản hồi khảo sát hài lòng (điểm 1–5 kèm nhận xét) của khách hàng sau khi phiếu bảo hành được đóng, và tổng hợp chỉ số hài lòng theo trung tâm bảo hành, theo kỹ thuật viên và theo thời gian.

---

## 1. Giới thiệu và bảng thuật ngữ

### 1.1 Mục đích
Tài liệu đặc tả yêu cầu cho luồng L8, giải quyết vấn đề **V7** của Mekong Mobile: sau bảo hành không có kênh thu thập phản hồi, khiếu nại chỉ được biết khi khách đăng lên mạng xã hội.

### 1.2 Bảng thuật ngữ (dùng đúng MỘT tên trong SRS, sơ đồ và wireframe)

| Thuật ngữ | Định nghĩa | Tên kỹ thuật |
|---|---|---|
| Khách hàng | Cá nhân đã mua sản phẩm hoặc dùng dịch vụ của Mekong Mobile. | customer |
| Phiếu bảo hành | Yêu cầu bảo hành/sửa chữa có mã duy nhất và vòng đời trạng thái. | ticket |
| Trạng thái phiếu | Vị trí của phiếu trong vòng đời: Mới → Đã phân công → Đang xử lý → Chờ linh kiện → Hoàn tất → Đã đóng (hoặc Đã hủy). | ticket_status |
| Kỹ thuật viên | Nhân viên thực hiện sửa chữa. | technician |
| Trung tâm bảo hành | Đơn vị tiếp nhận và xử lý phiếu bảo hành (6 trung tâm). | service_center |
| Khảo sát hài lòng | Hoạt động hỏi khách mức hài lòng sau khi phiếu được đóng; thang điểm 1–5 kèm nhận xét. | survey |
| Lời mời khảo sát | Tin nhắn gửi cho khách, chứa liên kết duy nhất để trả lời khảo sát hài lòng của một phiếu. | survey_invitation |
| Phản hồi | Một lần trả lời khảo sát hài lòng của khách cho một phiếu (điểm + nhận xét). Chỉ dùng từ "phản hồi" cho khái niệm này. | survey_response |
| Điểm hài lòng | Số nguyên 1–5 mà khách chọn (1 = rất không hài lòng, 5 = rất hài lòng). | score |
| Tỉ lệ hài lòng | Số phản hồi có điểm ≥ 4 chia tổng số phản hồi trong kỳ (%). | csat_rate |
| Tỉ lệ phản hồi | Số phản hồi nhận được chia số lời mời khảo sát đã gửi trong kỳ (%). | response_rate |
| Phản hồi điểm thấp | Phản hồi có điểm ≤ 2. | low_score |

> Các dòng "Trung tâm bảo hành", "Lời mời khảo sát", "Phản hồi", "Điểm hài lòng", "Tỉ lệ hài lòng", "Tỉ lệ phản hồi", "Phản hồi điểm thấp" là thuật ngữ **bổ sung** của luồng L8, ngoài Bảng 3.1 của case study.

---

## 2. Mô tả tổng quan

### 2.1 Phạm vi
**Trong phạm vi:** gửi lời mời khảo sát khi phiếu ở trạng thái Đã đóng; khách trả lời; lưu phản hồi; tổng hợp và xem chỉ số theo trung tâm bảo hành, kỹ thuật viên, thời gian; xem phản hồi điểm thấp.
**Ngoài phạm vi:** tạo/đóng phiếu (luồng L2, L4), phân tích nội dung nhận xét bằng AI, chiến dịch marketing (L9), tính NPS (xem US9 – WON'T).

### 2.2 Tác nhân (actor)

| Actor | Loại | Vai trò trong luồng |
|---|---|---|
| Khách hàng | Người | Trả lời khảo sát hài lòng. |
| Marketing | Người | Theo dõi xu hướng, xem phản hồi điểm thấp để chăm sóc khách. |
| Quản lý trung tâm bảo hành | Người | Xem chỉ số của trung tâm bảo hành mình phụ trách và của kỹ thuật viên. |
| Ban giám đốc | Người | Xem bức tranh tổng hợp toàn công ty. |
| Bộ định thời | Thời gian | Kích hoạt gửi lời mời khi phiếu đóng và nhắc khách chưa trả lời. |
| Cổng nhắn tin | Hệ thống ngoài | Chuyển lời mời/nhắc tới số điện thoại khách (SMS/Zalo — *giả định kênh*). |

### 2.3 Giả định và ràng buộc
- **G1** Luồng L2/L4 đã tồn tại; L8 chỉ đọc phiếu và nhận sự kiện "phiếu chuyển sang Đã đóng". Dữ liệu thử dùng `tickets_history.csv`, `ticket_status_log.csv`, `survey_responses.csv`.
- **G2 (cần xác nhận)** Kênh gửi là tin nhắn tới số điện thoại đã chuẩn hóa (QT-02).
- **G3 (đề xuất)** Liên kết khảo sát hết hạn sau 7 ngày; nhắc khách một lần sau 3 ngày chưa trả lời.
- **G4** Thang điểm chỉ 1–5 (theo từ điển dữ liệu `survey_response`), nên chỉ số chính là điểm trung bình và tỉ lệ hài lòng; không tính NPS (thang 0–10).
- **R1** Ràng buộc dự án (Phó TGĐ): làm từng phần, phần nào xong dùng phần đó → L8 phải triển khai độc lập được.

---

## 3. Yêu cầu chức năng

| Mã | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-01 | Khi phiếu chuyển sang Đã đóng, hệ thống tạo lời mời khảo sát và gửi tới số điện thoại của khách, kèm liên kết duy nhất. | MUST |
| FR-02 | Hệ thống chỉ gửi lời mời cho phiếu ở trạng thái Đã đóng và chưa có phản hồi (QT-10); không gửi cho phiếu Đã hủy. | MUST |
| FR-03 | Hệ thống ghi trạng thái gửi (đã gửi / thất bại + lý do); gửi lỗi thì thử lại tối đa 3 lần. | MUST |
| FR-04 | Hệ thống hiển thị form khảo sát gồm mã phiếu, tên thiết bị, chọn điểm 1–5 (bắt buộc), nhận xét tùy chọn tối đa 1.000 ký tự. | MUST |
| FR-05 | Hệ thống kiểm tra hợp lệ rồi lưu phản hồi (ticket_id, score, comment, responded_at); mỗi phiếu chỉ một phản hồi. | MUST |
| FR-06 | Hệ thống tính và hiển thị theo trung tâm bảo hành, theo tháng: điểm trung bình, tỉ lệ hài lòng, số phản hồi, tỉ lệ phản hồi. | MUST |
| FR-07 | Hệ thống hiển thị điểm trung bình và tỉ lệ hài lòng theo kỹ thuật viên; chỉ hiển thị khi kỹ thuật viên có ≥ 5 phản hồi trong kỳ. | SHOULD |
| FR-08 | Hệ thống liệt kê phản hồi điểm thấp (điểm ≤ 2), lọc theo trung tâm bảo hành và khoảng ngày, kèm mã phiếu và nhận xét. | SHOULD |
| FR-09 | Hệ thống hiển thị xu hướng điểm trung bình và tỉ lệ hài lòng theo tháng toàn công ty, so sánh giữa các trung tâm bảo hành. | SHOULD |
| FR-10 | Hệ thống nhắc khách một lần nếu sau 3 ngày chưa trả lời. | COULD |
| FR-11 | Hệ thống xuất báo cáo hài lòng ra Excel/PDF theo bộ lọc đang chọn. | COULD |
| FR-12 | Hệ thống tính chỉ số NPS. | WON'T |

*(Chi tiết User Story, tiêu chí chấp nhận và đặc tả use case ở Phụ lục A, B.)*

---

## 4. Yêu cầu phi chức năng (có ngưỡng số)

| Mã | Nhóm | Yêu cầu có ngưỡng đo được | Áp dụng cho |
|---|---|---|---|
| NFR-01 | Hiệu năng | Trang khảo sát tải xong ≤ 3 giây và lưu phản hồi ≤ 2 giây ở 95% yêu cầu, trên mạng 4G. | FR-04, FR-05 |
| NFR-02 | Hiệu năng | Màn hình chỉ số trả kết quả ≤ 5 giây ở 95% yêu cầu với dữ liệu tới 50.000 phản hồi. | FR-06 → FR-09 |
| NFR-03 | Độ tin cậy | ≥ 98% lời mời được gửi thành công trong ≤ 15 phút kể từ khi phiếu đóng. | FR-01, FR-03 |
| NFR-04 | Bảo mật | Liên kết khảo sát dùng mã ngẫu nhiên ≥ 128 bit, hết hạn sau 7 ngày, chỉ dùng cho đúng một phiếu. | FR-04, FR-05 |
| NFR-05 | Toàn vẹn dữ liệu | 0 phiếu có hơn 1 phản hồi; 0 bản ghi có điểm ngoài khoảng 1–5 (ràng buộc UNIQUE và CHECK ở CSDL). | FR-05 |
| NFR-06 | Phân quyền / riêng tư | 100% truy cập báo cáo tuân QT-14; số điện thoại hiển thị dạng che với mọi vai trò trừ Quản lý và Ban giám đốc (QT-15). | FR-06 → FR-11 |

---

## 5. Quy tắc nghiệp vụ và ràng buộc

### 5.1 Quy tắc của case study áp dụng cho L8

| Mã | Nội dung | Áp dụng vào |
|---|---|---|
| QT-02 | Số điện thoại chuẩn hóa về 10 chữ số bắt đầu bằng 0 trước khi dùng. | FR-01 |
| QT-06 | Phiếu chỉ chuyển trạng thái theo vòng đời; mọi lần chuyển được ghi vào ticket_status_log. Thời điểm "Đã đóng" lấy từ log. | FR-01, FR-02 |
| QT-10 | Chỉ gửi khảo sát cho phiếu Đã đóng; mỗi phiếu khảo sát một lần. | FR-02, FR-05 |
| QT-13 | Không xóa vật lý phiếu hay phản hồi; chỉ đánh dấu ngừng sử dụng. | FR-05 |
| QT-14 | Nhân viên chỉ xem dữ liệu đơn vị mình; quản lý xem đơn vị mình phụ trách; Ban giám đốc xem toàn công ty. | FR-06 → FR-11 |
| QT-15 | Số điện thoại hiển thị dạng che trừ Quản lý và Ban giám đốc. | FR-08 |

### 5.2 Quy tắc bổ sung của luồng (đề xuất, cần giảng viên/khách hàng xác nhận)

| Mã | Nội dung |
|---|---|
| QT-L8-01 | Tỉ lệ hài lòng = số phản hồi có điểm ≥ 4 / tổng số phản hồi trong kỳ. |
| QT-L8-02 | Tỉ lệ phản hồi = số phản hồi / số lời mời đã gửi thành công trong kỳ. |
| QT-L8-03 | Phản hồi điểm thấp là phản hồi có điểm ≤ 2. |
| QT-L8-04 | Phản hồi được tính cho kỹ thuật viên đang được gán tại thời điểm phiếu Đã đóng (QT-07 cho phép đổi kỹ thuật viên giữa chừng). |
| QT-L8-05 | Chỉ số theo kỹ thuật viên chỉ hiển thị khi có ≥ 5 phản hồi trong kỳ, tránh kết luận từ mẫu quá nhỏ. |

---

## 6. Bảng truy vết FR ↔ US ↔ Use Case ↔ MoSCoW

| FR | User Story | Use Case | MoSCoW |
|---|---|---|---|
| FR-01 | US1 | UC1 Gửi lời mời khảo sát hài lòng | MUST |
| FR-02 | US1 | UC1 Gửi lời mời khảo sát hài lòng | MUST |
| FR-03 | US1 | UC1 Gửi lời mời khảo sát hài lòng | MUST |
| FR-04 | US2 | UC2 Trả lời khảo sát hài lòng | MUST |
| FR-05 | US2 | UC2 Trả lời khảo sát hài lòng | MUST |
| FR-06 | US3 | UC4 Xem chỉ số hài lòng theo trung tâm bảo hành | MUST |
| FR-07 | US4 | UC5 Xem chỉ số hài lòng theo kỹ thuật viên | SHOULD |
| FR-08 | US5 | UC7 Xem danh sách phản hồi điểm thấp | SHOULD |
| FR-09 | US6 | UC6 Xem xu hướng hài lòng theo thời gian | SHOULD |
| FR-10 | US7 | UC3 Nhắc khách chưa trả lời khảo sát | COULD |
| FR-11 | US8 | UC8 Xuất báo cáo hài lòng | COULD |
| FR-12 | US9 | Không vẽ (WON'T, ngoài phạm vi đợt này) | WON'T |

---

## Phụ lục A. User Story

### A.1 Danh sách, kiểm INVEST và MoSCoW

Mỗi story đã qua 6 bước: kiểm khuôn → soi 4 lỗi → INVEST → MoSCoW → GWT (với MUST) → gán mã. Giá trị ("để…") nối về vấn đề **V7**.

| Mã | User Story | MoSCoW | I | N | V | E | S | T |
|---|---|---|---|---|---|---|---|---|
| US1 | Là **Marketing**, tôi muốn hệ thống tự gửi lời mời khảo sát tới khách ngay khi phiếu được đóng, để thu được phản hồi mà không phải gọi từng khách. | MUST | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US2 | Là **khách hàng**, tôi muốn chọn điểm 1–5 và nhập nhận xét ngắn sau khi bảo hành xong, để phản ánh mức hài lòng trực tiếp tới công ty thay vì đăng lên mạng xã hội. | MUST | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US3 | Là **quản lý trung tâm bảo hành**, tôi muốn xem điểm trung bình và tỉ lệ hài lòng của trung tâm mình theo tháng, để biết chất lượng dịch vụ có đi xuống hay không. | MUST | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US4 | Là **quản lý trung tâm bảo hành**, tôi muốn xem điểm hài lòng theo từng kỹ thuật viên, để nhận biết người cần hỗ trợ và ghi nhận người làm tốt. | SHOULD | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US5 | Là **Marketing**, tôi muốn xem danh sách phản hồi điểm thấp kèm nhận xét, để gọi chăm sóc khách trước khi họ đăng lên mạng xã hội. | SHOULD | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US6 | Là **thành viên Ban giám đốc**, tôi muốn xem xu hướng hài lòng theo tháng và so sánh giữa các trung tâm bảo hành, để đánh giá chất lượng dịch vụ toàn công ty. | SHOULD | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US7 | Là **Marketing**, tôi muốn hệ thống nhắc khách chưa trả lời sau 3 ngày, để tăng tỉ lệ phản hồi. | COULD | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US8 | Là **quản lý trung tâm bảo hành**, tôi muốn xuất báo cáo hài lòng ra Excel, để dùng cho buổi họp giao ban mà không phải chép tay. | COULD | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| US9 | Là **thành viên Ban giám đốc**, tôi muốn xem chỉ số NPS, để so sánh với chuẩn ngành. | WON'T | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

*Ghi chú INVEST:* US3–US6 phụ thuộc dữ liệu từ US2 nhưng có thể phát triển song song bằng dữ liệu mẫu (`survey_responses.csv`) nên vẫn đạt "I". US9 là WON'T vì thang điểm hiện là 1–5, không đủ cơ sở tính NPS (thang 0–10); cần đổi thang khảo sát trước.

### A.2 Tiêu chí chấp nhận Given–When–Then (story MUST)

**US1 – Gửi lời mời khảo sát**
- **AC1.1 (chính):** *Given* phiếu BH000123/2026 vừa chuyển sang Đã đóng và số điện thoại khách hợp lệ; *When* hệ thống ghi nhận lần chuyển trạng thái; *Then* lời mời kèm liên kết duy nhất được gửi trong ≤ 15 phút và thời điểm gửi được ghi lại.
- **AC1.2 (ngoại lệ):** *Given* phiếu đã có phản hồi hoặc đang ở trạng thái Đã hủy; *When* có yêu cầu gửi lời mời; *Then* hệ thống không gửi và ghi lý do bỏ qua.
- **AC1.3 (ngoại lệ):** *Given* cổng nhắn tin trả lỗi hoặc số điện thoại thiếu/sai định dạng; *When* hệ thống gửi lời mời; *Then* hệ thống thử lại tối đa 3 lần, rồi đánh dấu "thất bại" kèm lý do.

**US2 – Trả lời khảo sát**
- **AC2.1 (chính):** *Given* khách mở liên kết hợp lệ của phiếu Đã đóng chưa có phản hồi; *When* khách chọn 4 điểm, nhập nhận xét và bấm Gửi; *Then* hệ thống lưu phản hồi với score = 4, responded_at là thời điểm hiện tại và hiển thị lời cảm ơn.
- **AC2.2 (ngoại lệ):** *Given* khách chưa chọn điểm; *When* bấm Gửi; *Then* hệ thống báo "Vui lòng chọn điểm 1–5", giữ nguyên nhận xét đã nhập và không lưu gì.
- **AC2.3 (ngoại lệ):** *Given* phiếu đã có phản hồi; *When* khách mở lại liên kết; *Then* hệ thống thông báo đã phản hồi và không cho gửi lần hai (QT-10).

**US3 – Xem chỉ số theo trung tâm bảo hành**
- **AC3.1 (chính):** *Given* tháng 09/2026 trung tâm bảo hành của chị Trâm có 30 phản hồi, trong đó 24 phản hồi điểm ≥ 4; *When* chị chọn tháng 09/2026; *Then* màn hình hiển thị tỉ lệ hài lòng 80%, kèm điểm trung bình (1 chữ số thập phân), số phản hồi và tỉ lệ phản hồi.
- **AC3.2 (ngoại lệ):** *Given* tháng được chọn chưa có phản hồi nào; *When* xem chỉ số; *Then* hệ thống hiển thị "Chưa có phản hồi" thay vì 0% hoặc lỗi chia cho 0.
- **AC3.3 (ngoại lệ – phân quyền):** *Given* người dùng là Quản lý của trung tâm bảo hành A; *When* yêu cầu xem dữ liệu trung tâm bảo hành B; *Then* hệ thống từ chối và không trả dữ liệu của B (QT-14).

---

## Phụ lục B. Đặc tả Use Case

### B.1 Danh sách use case (xem sơ đồ `docs/usecase-l8.drawio`)

| Mã | Tên | Actor chính | Actor phụ | MoSCoW |
|---|---|---|---|---|
| UC1 | Gửi lời mời khảo sát hài lòng | Bộ định thời | Cổng nhắn tin | MUST |
| UC2 | Trả lời khảo sát hài lòng | Khách hàng | — | MUST |
| UC3 | Nhắc khách chưa trả lời khảo sát | Bộ định thời | Cổng nhắn tin | COULD |
| UC4 | Xem chỉ số hài lòng theo trung tâm bảo hành | Quản lý trung tâm bảo hành | Ban giám đốc | MUST |
| UC5 | Xem chỉ số hài lòng theo kỹ thuật viên | Quản lý trung tâm bảo hành | — | SHOULD |
| UC6 | Xem xu hướng hài lòng theo thời gian | Ban giám đốc | Marketing, Quản lý trung tâm bảo hành | SHOULD |
| UC7 | Xem danh sách phản hồi điểm thấp | Marketing | Quản lý trung tâm bảo hành | SHOULD |
| UC8 | Xuất báo cáo hài lòng | Quản lý trung tâm bảo hành | Marketing, Ban giám đốc | COULD |

Quan hệ: UC3 **extend** UC1 (chỉ xảy ra khi khách chưa trả lời sau 3 ngày).

### B.2 Đặc tả chi tiết UC2 – Trả lời khảo sát hài lòng (use case quan trọng nhất)

*Lý do chọn:* UC2 là nơi duy nhất tạo ra dữ liệu phản hồi; mọi báo cáo UC4–UC8 phụ thuộc vào nó.

| Mục | Nội dung |
|---|---|
| Actor chính | Khách hàng |
| Mục tiêu | Khách gửi điểm hài lòng và nhận xét cho phiếu bảo hành đã đóng. |
| Tiền điều kiện | Phiếu ở trạng thái Đã đóng; khách đã nhận lời mời khảo sát (UC1); phiếu chưa có phản hồi; liên kết còn hạn (≤ 7 ngày). |
| Hậu điều kiện (thành công) | Một phản hồi (ticket_id, score, comment, responded_at) được lưu; phiếu được đánh dấu đã khảo sát; chỉ số được tính vào lần tổng hợp kế tiếp. |
| Hậu điều kiện (thất bại) | Không có bản ghi phản hồi nào được tạo. |
| Quy tắc liên quan | QT-10, QT-13, NFR-01, NFR-04, NFR-05 |

**Luồng chính**
1. Khách mở liên kết trong lời mời khảo sát.
2. Hệ thống kiểm tra liên kết hợp lệ, còn hạn, phiếu ở trạng thái Đã đóng và chưa có phản hồi.
3. Hệ thống hiển thị form: mã phiếu, tên thiết bị, thang điểm 1–5, ô nhận xét (tùy chọn).
4. Khách chọn điểm và (tùy chọn) nhập nhận xét.
5. Khách bấm Gửi.
6. Hệ thống kiểm tra điểm thuộc 1–5 và nhận xét ≤ 1.000 ký tự.
7. Hệ thống lưu phản hồi và ghi responded_at.
8. Hệ thống hiển thị lời cảm ơn. Use case kết thúc.

**Luồng ngoại lệ (đánh số theo bước)**
- **2a.** Liên kết không hợp lệ hoặc đã hết hạn → hệ thống hiển thị "Liên kết đã hết hạn", không hiển thị form. Use case kết thúc (thất bại).
- **2b.** Phiếu đã có phản hồi (QT-10) → hệ thống hiển thị "Bạn đã gửi phản hồi cho phiếu này", không cho sửa. Use case kết thúc.
- **2c.** Phiếu không ở trạng thái Đã đóng → hệ thống từ chối hiển thị form. Use case kết thúc (thất bại).
- **4a.** Khách đóng trình duyệt giữa chừng → không lưu gì; liên kết vẫn dùng được đến khi hết hạn.
- **6a.** Chưa chọn điểm hoặc điểm ngoài 1–5 → hệ thống báo lỗi ở ô điểm, giữ nhận xét đã nhập, quay lại bước 4.
- **6b.** Nhận xét vượt 1.000 ký tự → hệ thống báo lỗi, giữ nội dung, quay lại bước 4.
- **7a.** Lỗi lưu dữ liệu → hệ thống không tạo bản ghi dở dang, báo "Có lỗi, vui lòng thử lại", quay lại bước 5 (nội dung form được giữ).
- **7b.** Khách bấm Gửi hai lần liên tiếp (vi phạm UNIQUE ticket_id) → hệ thống chỉ giữ một bản ghi và chuyển sang bước 8.

---

## Phụ lục C. Đặc tả dữ liệu theo track DA (Data requirement specification)

### C.1 Nguồn dữ liệu

| Nguồn | Hệ thống nguồn / Định dạng | Tần suất cập nhật | Khối lượng ước tính |
|---|---|---|---|
| survey_responses | CRM (bảng survey_response) / `survey_responses.csv` | Theo thời gian thực; nạp vào kho hằng đêm | ~2.600 dòng (mẫu); ~35–60 dòng/tháng mỗi trung tâm bảo hành *(ước tính từ 260 phiếu/tháng toàn công ty × tỉ lệ phản hồi ~33%)* |
| ticket | CRM / `tickets_history.csv` | Hằng đêm | ~7.800 dòng (30 tháng) |
| ticket_status_log | CRM / `ticket_status_log.csv` | Hằng đêm | ~31.000 dòng |
| technician | CRM / `technicians.csv` | Khi có thay đổi nhân sự | 38 dòng |
| service_center | CRM / `service_centers.csv` | Hiếm khi thay đổi | 6 dòng |

Tỉ lệ phản hồi ước tính thô ~33% (2.600 / 7.800), chưa loại phiếu chưa đóng; cần đo lại sau khi lọc phiếu Đã đóng.

### C.2 Từ điển dữ liệu nguồn (bảng survey_response)

Cột lấy theo từ điển tham chiếu (Mục 8 case study). **Cột "Tỉ lệ thiếu" phải đo trên `survey_responses.csv` thật** — chưa điền số vì chưa chạy trên dữ liệu.

| Cột | Kiểu | Ý nghĩa | Giá trị hợp lệ | Tỉ lệ thiếu |
|---|---|---|---|---|
| response_id | BIGINT | Khóa chính | Duy nhất | *đo* |
| ticket_id | BIGINT | Phiếu được khảo sát | FK → ticket; UNIQUE | *đo* |
| score | SMALLINT | Điểm hài lòng | 1–5 | *đo* |
| comment | TEXT | Nhận xét của khách | Tùy chọn, ≤ 1.000 ký tự | *đo* (thiếu là bình thường) |
| responded_at | TIMESTAMP | Thời điểm trả lời | ≥ closed_at của phiếu | *đo* |

Cột dùng từ bảng khác: `ticket.status`, `ticket.closed_at`, `ticket.center_id`, `ticket.technician_id`, `technician.technician_id`, `service_center.center_id`.

Đoạn lệnh đo nhanh (đổi tên cột nếu tệp thật khác):

```python
import pandas as pd
df = pd.read_csv("survey_responses.csv")
print((df.isna().mean() * 100).round(2))        # tỉ lệ thiếu từng cột
print("trùng ticket_id:", df.duplicated("ticket_id").sum())
print("điểm ngoài 1-5:", (~df["score"].between(1, 5)).sum())
```

### C.3 Quy tắc chất lượng dữ liệu phải đạt

| Mã | Quy tắc | Ngưỡng |
|---|---|---|
| DQ-01 | Completeness của ticket_id, score, responded_at | = 100% |
| DQ-02 | Điểm thuộc khoảng 1–5 | 100% bản ghi |
| DQ-03 | Không trùng theo khóa nghiệp vụ ticket_id | 0 bản ghi trùng |
| DQ-04 | Toàn vẹn tham chiếu: ticket_id tồn tại trong ticket và ticket ở trạng thái Đã đóng | ≥ 99% |
| DQ-05 | responded_at ≥ closed_at của phiếu | ≥ 99% |
| DQ-06 | Completeness của technician_id và center_id trên phiếu được khảo sát | ≥ 95% |
| DQ-07 | Định dạng thời gian thống nhất (ISO 8601) | 100% |

### C.4 Câu hỏi phân tích và mức chi tiết báo cáo

| Mã | Câu hỏi phân tích | Mức chi tiết | Truy vết |
|---|---|---|---|
| Q1 | Điểm hài lòng trung bình của từng trung tâm bảo hành theo tháng là bao nhiêu? | Tháng × trung tâm bảo hành | US3 |
| Q2 | Tỉ lệ hài lòng (điểm ≥ 4) của từng trung tâm bảo hành theo tháng? | Tháng × trung tâm bảo hành | US3 |
| Q3 | Tỉ lệ phản hồi của từng trung tâm bảo hành theo tháng? | Tháng × trung tâm bảo hành | US3 |
| Q4 | Điểm trung bình theo từng kỹ thuật viên (chỉ khi ≥ 5 phản hồi)? | Tháng × kỹ thuật viên | US4 |
| Q5 | Những phiếu nào có điểm ≤ 2 trong khoảng ngày chọn, nhận xét ra sao? | Phiếu (mức chi tiết nhất) | US5 |
| Q6 | Điểm trung bình và tỉ lệ hài lòng toàn công ty thay đổi thế nào qua các tháng? | Tháng (toàn công ty) | US6 |

**Tự kiểm:** mỗi câu hỏi Q1–Q6 truy vết được về một User Story; không có câu hỏi nào không có US.

**Rủi ro dữ liệu cần lưu ý**
- Kỹ thuật viên có thể bị đổi giữa chừng (QT-07) → quy ước QT-L8-04.
- Phản hồi chỉ đến từ ~33% khách (thiên lệch tự chọn: người rất hài lòng hoặc rất bực dễ trả lời hơn) → luôn hiển thị kèm tỉ lệ phản hồi.
