# SmartFarm: Kiến trúc không gian cộng tác
## 1/ Cấu trúc thư mục
```
SmartFarm/                          ← Cloud Workspace (nguồn dữ liệu chính)

- 01_Code/                        ← Kỹ sư sửa; người khác chỉ xem
- 02_Data/
    + Raw/YYYY/MM/DD/             ← Dữ liệu thô theo ngày, giữ 30 ngày
    + Processed/                  ← Dữ liệu tổng hợp, giữ lâu dài
    + Archive/                    ← File .zip theo tháng
- 03_Docs/                        ← Hướng dẫn cho người nông dân
- 04_Reports/                     ← Báo cáo
- 05_Admin/                       ← CHỈ Chủ dự án: bản xuất Audit Log, biên bản rà soát quyền
```

## 2/ Chức năng 1 (Lưu trữ): không làm tràn dung lượng Cloud
50 MB/ngày × 30 ngày ≈ **1,5 GB/tháng**, tức khoảng 17,8 GB/năm.

| Dữ liệu | Quy tắc |
|---|---|
| Raw (0 đến 30 ngày) | Lưu theo ngày trong `Raw` |
| Raw (sau 30 ngày) | Nén thành 1 file .zip/tháng vào `Archive`, xóa file thô |
| Processed | Giữ lâu dài vì rất nhỏ |
| Gateway | Giữ bộ đệm 7 ngày, tự upload bù khi mất mạng |

- **Files On-Demand:** trên máy người dùng, đặt `Raw` và `Archive` ở chế độ **Online-only** (chỉ tải khi mở); chỉ chọn **Always keep on this device** cho `01_Code`, `03_Docs`, `04_Reports`.
- **Lưu ý:** Files On-Demand chỉ tiết kiệm **ổ cứng máy cá nhân**, không giảm dung lượng **Cloud**. Dung lượng Cloud được giữ bằng quy tắc nén và lưu trữ ở trên.
- Cảnh báo khi Cloud đạt 80% hạn mức.

## 3/ Chức năng 2 (Cộng tác): phân quyền
**Chú thích:** M = Manager, E = Editor (xem, sửa), V = Viewer (chỉ xem), W = chỉ ghi, – = không thấy thư mục.

| Vai trò (nhóm) | Source code | Dữ liệu cảm biến | Tài liệu hướng dẫn | Báo cáo |
|---|---|---|---|---|
| **Chủ dự án** | **M** | **M** | **M** | **M** |
| **Kỹ sư lập trình** | **E** | V | **E** | V |
| **Nông dân** | – | – | V | V |
| **Thành viên gia đình** | – | – | – | V |
| **Thiết bị IoT** (tài khoản dịch vụ) | – | W (chỉ vào `Raw`) | – | – |

- **Viewer xem báo cáo:** Kỹ sư, Nông dân, Thành viên gia đình.
- **Editor sửa code:** chỉ Kỹ sư, qua nhánh và Pull Request; nhánh chính cần ít nhất 1 người duyệt.
- Cấp quyền theo nhóm, không cấp từng cá nhân; không dùng link chia sẻ công khai.

## 4/ Chức năng 3 (Giao tiếp): tin nhắn báo lỗi → Task có Deadline
1. Gateway gửi tin báo lỗi gồm: mã cảm biến, mã lỗi, mức độ, thời gian.
2. Hệ thống tự động tạo Task trong Enterprise Workspace, gán người phụ trách và đặt Deadline theo mức độ.
3. Cùng một cảm biến và cùng mã lỗi mà Task còn mở thì cập nhật Task cũ, không tạo Task mới.

| Mức độ | Ví dụ | Deadline |
|---|---|---|
| Nghiêm trọng | Máy bơm không phản hồi | 4 giờ |
| Cao | Cảm biến mất tín hiệu | 24 giờ |
| Thường | Dữ liệu lệch bất thường | 3 ngày |

*(Các mốc Deadline là đề xuất, đề không quy định.)*

## 5/ Ràng buộc: Hybrid Workflow cho kỹ sư lập trình
1. Clone code về ổ đĩa **local**, không đặt trong thư mục đồng bộ Cloud (tránh xung đột `.git`).
2. Tạo nhánh riêng, viết và kiểm thử code trên máy local bằng trình giả lập (không thử trên máy bơm thật).
3. Push và tạo Pull Request; người khác duyệt rồi mới merge.
4. Triển khai bản đã merge lên gateway.
5. Tài liệu và báo cáo làm trực tiếp trên Cloud để đồng sửa và có lịch sử phiên bản.

## 6/ Ràng buộc: rà soát Access Audit Log hàng tháng
Thực hiện vào ngày làm việc đầu tiên mỗi tháng, Chủ dự án chịu trách nhiệm.

| # | Kiểm tra | Cách làm |
|---|---|---|
| 1 | Ai có quyền gì | Xuất danh sách thành viên, đối chiếu với bảng phân quyền mục 3 |
| 2 | Link chia sẻ | Tìm link công khai hoặc chia sẻ ra ngoài, thu hồi |
| 3 | Truy cập bất thường | Xem nhật ký: tải xuống số lượng lớn, đăng nhập giờ lạ, thiết bị lạ |
| 4 | Tài khoản không dùng | Hạ quyền hoặc xóa |

Lưu biên bản vào `05_Admin/` (tên `Audit_YYYY-MM`).
