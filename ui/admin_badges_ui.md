# Admin Huy hiệu — đặc tả UI

> **Trạng thái:** ACTIVE / DERIVED. Phạm vi và rule thuộc [badges.md](../decisions/badges.md), PD-BDG-13a–d (PO duyệt 2026-10-08). File này ghi phần UI đã thống nhất và hành vi P1; không mở thêm phạm vi P2/P3.
>
> **Hiện trạng:** P1 dùng API thật, chỉ đọc; module mặc định ẩn sau `VITE_ADMIN_BADGES_ENABLED`. Phát hành ngoài dev vẫn cần nghiệm thu trên staging theo badges.md §10. O1/O2 về mật độ còn mở (§6).

## 1. Phạm vi và điều hướng

- Sidebar: LiveOps & Economy → Huy hiệu, sau Nhiệm vụ hằng ngày.
- Breadcrumb: LiveOps / Huy hiệu.
- Catalog và chi tiết toàn trang trong cùng module. Admin hiện dùng `activeNav`, chưa có URL riêng cho từng huy hiệu.
- P1 chỉ có Xem, tìm kiếm, lọc trạng thái, Làm mới và quay lại danh sách. Không có nút tạo, sửa, bật/tắt.
- P2 và P3 theo badges.md §8.4, §10; chưa triển khai trong P1.
- `VITE_ADMIN_BADGES_ENABLED=true` mở P1 ở dev hoặc bản build được cấu hình rõ ràng. Không mặc định bật module production chỉ vì PO đã duyệt roadmap.

## 2. Catalog theo chuỗi

### Dữ liệu và thứ tự

- `GET /api/admin/badges`: toàn catalog kể cả huy hiệu đang tắt; không dùng API `/me`.
- Nhóm theo `metric`; thứ tự nhóm tính trên toàn catalog bằng `min(sortOrder)`, trước khi lọc.
- Trong nhóm: `threshold`, `sortOrder`, `id` tăng dần, theo UI-BDG-07.
- Tìm theo tên/mã, không phân biệt hoa thường hoặc dấu tiếng Việt. Trạng thái: Tất cả / Đang bật / Đang tắt.
- Lọc chỉ ẩn dòng và nhóm không còn dòng; không đổi vị trí nhóm. Số mốc và số đang bật ở đầu nhóm tính trên toàn catalog.

### Bố cục

- Header: tiêu đề Huy hiệu, mô tả ngắn và dòng tổng số huy hiệu / đang bật / chuỗi.
- Toolbar: tìm tên/mã, lọc trạng thái, Làm mới.
- Bảng có hàng tiêu đề cho từng chuỗi: tên chuỗi, enum nhỏ, số mốc và cách đếm.
- Các cột: Huy hiệu (artwork 32 px + tên + mã), Điều kiện, Mô tả hiển thị, Đã đạt, Trạng thái, Xem.
- Huy hiệu tắt vẫn dùng artwork dạng đã đạt; chip và dòng lý do thể hiện trạng thái cấp. Phôi xám chỉ thể hiện người học chưa đạt trong preview.
- `earnerCount = null`: thiếu dữ liệu, không coi là 0. Toàn catalog không có số người đạt thì ẩn cột; khi chỉ một số dòng thiếu thì dùng “—” cho những dòng đó.
- `earnerCount = 0`: số thực, vẫn hiện 0. Huy hiệu tắt và chưa ai đạt: “Người học không thấy huy hiệu này”.
- Huy hiệu tắt có người đạt: “Ngừng cấp · {N} người đã đạt vẫn thấy”. Khi chưa có số liệu: “Người đã đạt vẫn thấy; người chưa đạt không thấy”.
- Chỉ số chưa map có tên “Chỉ số chưa hỗ trợ”, enum và ghi chú. P1 vẫn xem được dữ liệu.

## 3. Chi tiết và preview

- Mở bằng tên hoặc nút Xem; đọc mới bằng `GET /api/admin/badges/{id}`.
- Giữ catalog mounted để bảo toàn từ khóa và bộ lọc. Lưu vị trí cuộn trước khi mở; quay lại khôi phục vị trí và focus về nút đã mở.
- Header dính: quay lại, tên, trạng thái; chỉ hiện phiên bản khi `version !== null`.
- Khối Điều kiện đạt chỉ đọc: câu điều kiện, chuỗi, ngưỡng, mã có nút sao chép và cách đếm.
- Khối Hiển thị cho người học: tên, mô tả, biểu tượng.
- Thứ tự nằm trong Cấu hình nâng cao; giải thích ảnh hưởng tới thứ tự kệ và thứ tự phụ trên Hồ sơ.
- Preview có huy chương đã đạt / chưa đạt, kệ trong chuỗi và chữ người học đọc. Không mô phỏng khung điện thoại.
- Kệ preview lấy các mốc đang bật trong toàn catalog cộng huy hiệu đang xem. Không phụ thuộc bộ lọc danh sách. Khi GET chi tiết mới hơn list, dùng dữ liệu chi tiết mới cho huy hiệu đó.
- Huy hiệu đang tắt: “Người học chưa đạt hiện không thấy huy hiệu này; kệ trên là khi huy hiệu được bật.”
- Kệ từ 4 mốc: nêu app chuyển sang cuộn ngang. Kệ thực tế có thể khác giữa người học đã đạt và chưa đạt huy hiệu đang tắt (badges.md §10).
- `iconKey` lạ: artwork mặc định `award`, kèm cảnh báo rõ ràng. App hỗ trợ `flame`, `clipboard-check`, `target`, `book-open`, `award`.
- Ngày tạo/cập nhật hiển thị theo múi giờ Việt Nam.

## 4. Tải, lỗi và làm mới

| Tình huống | Hành vi P1 |
| --- | --- |
| Tải lần đầu | Skeleton có `role=status`; chưa hiện catalog giả hoặc số liệu giả |
| GET list lỗi, chưa có dữ liệu | Banner lỗi và Thử lại; không rơi về fixture |
| Làm mới list lỗi | Giữ dữ liệu/bộ lọc đã tải; báo “Đang hiển thị dữ liệu đã tải trước đó” |
| API chưa có (404 ngoài `BADGE_NOT_FOUND`) | “API quản trị huy hiệu chưa sẵn sàng trên máy chủ này” |
| 403 | “Bạn không có quyền xem catalog huy hiệu” |
| Catalog rỗng | Trạng thái rỗng, không thay bằng seed phía client |
| Không có kết quả lọc | Câu không có kết quả và nút Xóa bộ lọc |
| Đang tải chi tiết | Skeleton và Quay lại danh sách; không lấy row cũ làm bản chi tiết mới |
| GET chi tiết lỗi | Banner + Thử lại, vẫn rời trang được; nếu đã tải chi tiết thì giữ bản đó với cảnh báo |
| `BADGE_NOT_FOUND` | Quay lại danh sách, thông báo không tìm thấy và refetch list |
| Làm mới trong chi tiết | Đọc lại list và detail để cập nhật cả kệ và nội dung; báo khi một phần dữ liệu chưa tải lại được |

## 5. Accessibility và responsive

- Tên và Xem là button thật, điều khiển được bằng bàn phím; không bắt cả dòng làm vùng bấm.
- Focus vào tiêu đề chi tiết khi mở, trở về nút nguồn khi đóng.
- Nhãn rõ cho tìm kiếm, lọc trạng thái, nút sao chép; trạng thái luôn có chữ.
- Mỗi chuỗi là một `tbody`, tiêu đề dùng `th scope=rowgroup`; các cột dùng `scope=col`.
- Artwork là trang trí (`aria-hidden`) vì đã có tên cạnh hình.
- ≥ 1280 px: đủ cột; 1024–1279 px: ẩn Mô tả và Đã đạt.
- < 1024 px: preview xuống dưới khối thông tin; bảng cuộn ngang trong khung khi cần. Không bổ sung hỗ trợ điện thoại ngoài khả năng shell Admin hiện tại.

## 6. Mật độ và hai quan sát còn mở

- **O1:** giữ hàng nhóm hai dòng có cách đếm. Chưa chuyển cách đếm lên cùng dòng ở màn rộng.
- **O2:** giữ artwork 32 px trên catalog. Chưa tăng lên 40 px.
- Số đo rev 2.1 thuộc prototype fixture, không tự coi là nghiệm thu dữ liệu thật. P1 bỏ dải chọn fixture.
- Kiểm AC-DEN trên 1366×768, 1536×864, 1920×1080, 1536×674; kiểm hai cột ẩn ở 1024 px. Mục tiêu tạm: ở 1366×768 nhìn thấy đầu ít nhất hai chuỗi, không cuộn ngang ở ≥1280 px.
- Đánh giá O1/O2 sau khi xem P1 bằng API và kiểm với người vận hành; không làm chúng thành điều kiện mở rộng scope.

## 7. Nghiệm thu và giới hạn

- Rule và contract: badges.md §8.4, §12.5.
- Test tự động cho nhóm/thứ tự, lọc, preview, icon lạ, nullable version/count; DOM cho tải/lỗi, giữ dữ liệu khi refetch lỗi, điều hướng và focus. HTTP test cho quyền Admin, aggregate counts, 404 và GET không ghi dữ liệu.
- Trước phát hành vẫn cần staging, kiểm mật độ ở viewport thật, bàn phím và screen reader. Test local không thay cho các bước này.
- P1 không thêm cột `version`, không cấp huy hiệu, không sửa catalog và không thay đổi sync.
