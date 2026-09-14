# Thiết Kế Giao Diện Mobile (Figma) - Luồng Quản Lý Bộ Sưu Tập, Chủ Đề & Bố Cục Thẻ Học (Tối Giản & Thân Thiện)

Tài liệu này tổng hợp toàn bộ thiết kế giao diện Mobile (chuẩn `390 x 884`) đã được hoàn thiện trực tiếp trên file Figma **"Đồ án"** (Page: `SnapVocab Learn`, File Key: `unsaved-mu0lggkk-tebb5rd8`). Thiết kế tuân thủ hoàn hảo triết lý sản phẩm SnapVocab: tối giản, loại bỏ hoàn toàn các thuật ngữ kỹ thuật/AI phức tạp, thống nhất kiến trúc **Template & Schema**, tối ưu trải nghiệm chạm trên màn hình di động nhỏ và áp dụng bảng màu nhận diện thương hiệu tím đặc trưng (`#3525cd`).

---

## 1. Trực Quan Toàn Bộ Luồng 7 Màn Hình

````carousel
![Bước 1: Danh sách Bộ sưu tập](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_1_collections.png)
<!-- slide -->
![Bước 2: Modal tạo bộ sưu tập (Tối giản)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_2_create_modal.png)
<!-- slide -->
![Bước 3: Chi tiết bộ sưu tập trống (Linh vật Cáo)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_3_empty_state.png)
<!-- slide -->
![Bước 4: Thêm chủ đề mới (Thuần form điền)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_4_create_topic.png)
<!-- slide -->
![Bước 5: Bố cục thẻ từ vựng (Template Builder tối ưu Mobile)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_5_card_layout.png)
<!-- slide -->
![Bước 6: Tùy chỉnh chi tiết trường dữ liệu (Field Settings Bottom Sheet)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_6_field_settings.png)
<!-- slide -->
![Bước 7: Cài đặt phần hiển thị (Section Settings Bottom Sheet)](C:\Users\Thanh\.gemini\antigravity-cli\brain\d42d488a-599a-4519-b060-4202049f3a8f\screen_7_section_settings.png)
````

---

## 2. Chi Tiết Tinh Chỉnh Từng Màn Hình

### Bước 1: `Collections - List` (Node ID: `202:3` | `x = 2750`)
* **Giao diện:** Tab "Của tôi" & "Hệ thống", các thẻ Bộ sưu tập với thanh tiến độ ghi nhớ từ vựng (SRS).
* **Nút bấm:** Nút nổi `+ Tạo bộ sưu tập` lơ lửng ngay phía trên thanh điều hướng.
* **Navigation:** `BottomNavBar` nguyên bản của SnapVocab với nút Camera tròn tím nổi bật.

---

### Bước 2: `Create Collection - Modal` (Node ID: `202:5` | `x = 3200`)
* **Tinh chỉnh thân thiện người dùng:**
  * **Bỏ bảng màu & biểu tượng:** Giảm tải lựa chọn không cần thiết khi người dùng chỉ muốn nhanh chóng tạo bộ sưu tập.
  * **Bỏ thuật ngữ kỹ thuật (Schema / Template):** Thay thế bằng ghi chú ngắn gọn, tự nhiên:
    > *"✨ Tự động có sẵn phát âm, phiên âm và ví dụ cho mỗi từ."*
  * **Form tinh gọn:** Chỉ gồm 2 ô điền: *Tên bộ sưu tập* và *Mô tả / Ghi chú*.
  * **Nút hành động:** `Tạo bộ sưu tập` (Primary `#3525cd`) và `Hủy bỏ`.

---

### Bước 3: `Collection Detail - Empty State` (Node ID: `202:6` | `x = 3650`)
* **Giao diện:** 
  * Header với nút quay lại `←` và thông tin số lượng `0 chủ đề · 0 từ vựng`.
  * Thẻ trạng thái trống tích hợp **Linh vật Cáo vẫy tay chào (`fox-wave 1`)** thân thiện, ấm áp.
  * Lời mời hành động trực tiếp: `+ Tạo chủ đề đầu tiên` và `📷 Quét từ vựng bằng Camera`.

---

### Bước 4: `Create Topic` (Node ID: `202:176` | `x = 4100`)
* **Tinh chỉnh theo yêu cầu:**
  * **Thuần điền form, không có select phức tạp:** Bỏ hoàn toàn các nhóm chọn cấp độ A1-C2 và phương thức nhập từ rườm rà.
  * **Các ô nhập trực tiếp:**
    1. *Tên chủ đề \** (`Education & Technology`)
    2. *Bản dịch \** (`Giáo dục & Công nghệ hiện đại`)
    3. *Mô tả (Không bắt buộc)* (`Từ vựng chủ đề công nghệ giáo dục...`)
  * **Khu vực cấu hình thẻ học tối giản:** 
    Một dòng thẻ trắng trang nhã: `🗂️ Bố cục thẻ: Tiếng Anh chuẩn` đi kèm nút bấm tinh gọn `Tùy chỉnh ⚙️` để mở ngay trình chỉnh sửa bố cục (Bước 5).
  * **Nút bấm:** `Tạo chủ đề`.

---

### Bước 5: `Bố cục thẻ từ vựng (Template Builder)` (Node ID: `202:177` | `x = 4550`)
Được phục hồi chuẩn mực theo mô hình thẻ học nhiều phần, nhiều cột (hỗ trợ column/section break) nhưng loại bỏ hoàn toàn các emoji/icon rườm rà tạo cảm giác AI:
* **Top Bar chuẩn iOS & Nút Xem trước góc trên:**
  * Đỉnh máy có Dynamic Island và đồng hồ `9:41`.
  * Nút quay lại `←`, Tiêu đề `Bố cục thẻ từ vựng`, Phụ đề `Chủ đề: Du lịch & Khách sạn`.
  * Góc trên bên phải là nút text **`Xem trước`** dạng pill tím nhạt (`#ede9fe`) viền mềm, chữ tím đậm `#3525cd` (loại bỏ icon con mắt).
* **Thanh 2 Tab Mặt trước / Mặt sau lớn (Full-width Segmented Bar):**
  * Chiếm trọn bề ngang màn hình (`width: 358px`, `height: 46px`).
  * Duy nhất 2 tab: **`Mặt trước (Front)`** (thẻ trắng nổi bật đổ bóng) và **`Mặt sau (Back)`**. Vùng chạm ngón tay lớn, dễ thao tác trên mobile.
* **Khối Section 1: "Thông tin chung" (Bố cục 2 Cột tối giản, đồng điệu):**
  * Nút kéo `⠿`, tên phần `Thông tin chung`, huy hiệu `2 Cột`.
  * **Nút `Cài đặt` ngay trên header phần:** Mở Section Settings Bottom Sheet (Bước 7).
  * **Cột 1 (50%):** Thẻ `Từ vựng` [Bắt buộc] dạng 1 hàng nhỏ gọn, tinh tế (đã bỏ khung placeholder "Từ vựng chính"), nút `+ Thêm trường` viền nét liền.
  * **Cột 2 (50%):** Thẻ `Phiên âm` [Phát âm] dạng 1 hàng đồng bộ (đã bỏ khung placeholder "Ký tự phiên âm IPA"), nút `+ Thêm trường` viền nét liền.
  * **Nút thêm cột trải rộng (Full-width action):**
    * `+ Thêm cột (2/3)` trải rộng toàn bộ chiều ngang khối cột (viền nét liền xám nhạt `#e2e8f0`, nền `#f8fafc`). Tùy chọn đổi kiểu cột tập trung hoàn toàn trong nút **Cài đặt** của phần, giúp giao diện không bị trùng lặp tính năng.
* **Khối Section 2: "Nghĩa & Ví dụ" (Danh sách lặp lại 1-n):**
  * Nút kéo `⠿`, tên phần `Nghĩa & Ví dụ`, huy hiệu xanh `Lặp lại (1-n)`.
  * Nút `Cài đặt` mở Bottom Sheet thiết lập cho Section 2.
  * Các thẻ trường: `Bản dịch / Nghĩa` [Bắt buộc] và `Câu ví dụ (Sentence)` [Tùy chọn].
  * Nút: `+ Thêm trường vào nhóm lặp lại` (Viền nét liền `#e2e8f0`).
* **Nút tạo Section:** `+ Thêm phần mới` viền nét liền xám nhạt `#cbd5e1`, nền trắng thanh lịch.
* **Sticky Bottom Bar với nút Lưu toàn màn hình:**
  * Nút Primary CTA duy nhất trải rộng toàn dòng: **`Lưu bố cục thẻ học`** (Nền tím SnapVocab `#3525cd`, chữ trắng đậm, đổ bóng `0 4px 14px rgba(53,37,205,0.3)`).

---

### Bước 6: `Field Settings - Bottom Sheet` (Node ID: `202:372` | `x = 5000`)
Được thiết kế chuẩn mực theo mô hình **Hợp nhất Template & Schema**, loại bỏ hoàn toàn khái niệm kỹ thuật *"Bind to Schema Attribute"*, đem lại trải nghiệm thân thiện, sắc nét:
* **Khung viền thiết bị & Nền mờ phía sau (Backdrop Dimmer):**
  * Hiển thị Dynamic Island, trạng thái pin, sóng `5G` và giao diện Canvas Builder mờ nhẹ phía sau.
  * Thanh kéo xám `Sheet Handle` và nút đóng nhanh `✕` góc trên bên phải.
* **Header thông tin trường:**
  * Tiêu đề: `Tùy chỉnh trường: ` kèm tên trường nổi bật màu tím: `Nghĩa từ`.
  * Phụ đề: `Điều chỉnh cách hiển thị và kiểu dáng trên thẻ học`.
* **Ô nhập tên nhãn hiển thị (Field Label):**
  * Tiêu đề: `TÊN NHÃN HIỂN THỊ` kèm ô điền `Nghĩa tiếng Việt`.
* **Khối Vai trò nội dung / Loại dữ liệu (Content Role Pills):**
  * `Bản dịch / Nghĩa ✓` (Đang chọn - Nền tím nhạt `#ede9fe`, viền tím `#ddd6fe`, chữ tím `#635bff`).
  * `Từ vựng chính` và `Phiên âm`.
* **Khối Quy tắc hiển thị (Display Rules):**
  * **Bắt buộc điền** (Switch ON).
  * **Tự động ẩn ô này nếu để trống** (Switch ON).
* **Khối Tương tác phát âm (Audio Action):**
  * **🔊 Phát âm thanh khi chạm** (Switch ON).
  * **Nguồn phát âm thanh (Audio Source):** Bộ chuyển 2 nút riêng biệt: `[ Máy đọc tự động (TTS) ]` và `[ File âm thanh đính kèm ]`.
* **Khối Định dạng & Màu sắc (Style & Typography):**
  * **Cỡ chữ (Font Size):** 4 nút: `S`, `M` *(đang chọn)*, `L`, `XL`.
  * **Căn lề (Alignment):** 3 nút: `Trái` *(đang chọn)*, `Giữa`, `Phải`.
  * **Kiểu chữ (Font Style):** `B Đậm` *(đang chọn)* và `I Nghiêng`.
  * **Bảng màu chữ (Text Color):** 7 nút màu swatch tròn: Đen than mặc định `#0f172a` *(đang chọn)*, Tím `#635bff`, Xám, Xanh ngọc, Vàng cam, Đỏ hồng, Tím nhạt.
* **Thanh hành động cuối bảng (Bottom Actions):**
  * Nút xóa màu đỏ mềm mại: `Xóa trường này`.
  * Nút lưu áp dụng màu tím đổ bóng: `Lưu tùy chỉnh`.

---

### Bước 7: `Section Settings - Bottom Sheet` (Node ID: `202:745` | `x = 5450`)
Màn hình Bottom Sheet chuyên biệt dành riêng cho việc cài đặt phần hiển thị (Section):
* **Lớp phủ nền làm mờ (Dimmed Backdrop Overlay):** Nền đen mờ 45% phía sau giữ vững ngữ cảnh làm việc của người dùng.
* **Container Bottom Sheet:** Bo góc trên mềm mại `24px` với thanh kéo `Drag Handle` và nút đóng `✕`.
* **Tiêu đề Sheet:** `Cài đặt phần hiển thị`, phụ đề `Chỉnh sửa tên và cấu trúc hiển thị của phần này`.
* **Khối 1: Tên phần hiển thị (Section Label):**
  * Tiêu đề: `TÊN PHẦN HIỂN THỊ`.
  * Ô nhập trực tiếp: `Thông tin chung` (Trạng thái active viền tím `#3525cd` nổi bật kèm nút xóa nhanh `✕`).
  * Ghi chú hướng dẫn: *Hiển thị làm tiêu đề nhóm các ô thông tin trên thẻ từ vựng*.
* **Khối 2: Chế độ lặp lại (Repeatable 1-n):**
  * Thẻ viền mềm sang trọng chứa tiêu đề: `Cho phép lặp lại (Danh sách 1-n)`.
  * Mô tả: *Phù hợp nhóm mục như: nhiều nghĩa, ví dụ, cụm từ...*
  * Công tắc chuyển đổi (iOS Switch Toggle): Màu tím `#3525cd` bật sáng với nút trượt trắng đổ bóng.
* **Khối 3: Tùy chọn số cột hiển thị (Columns Layout):**
  * Tiêu đề: `SỐ CỘT HIỂN THỊ TRÊN MÀN HÌNH`.
  * 3 thẻ lựa chọn trực quan có hình minh họa thanh cột:
    1. `1 Cột` (Tràn rộng 100%)
    2. `2 Cột` (Chia đôi 50% - Đang chọn: viền tím `#3525cd` 2px, nền tím nhạt `#f5f3ff`)
    3. `3 Cột` (Chia 3 đều)
* **Khối 4: Hành động dưới đáy (Bottom Actions):**
  * Nút đỏ cảnh báo: `Xóa phần này (kèm các ô bên trong)` (Nền `#fef2f2`, viền `#fee2e2`, chữ `#dc2626`).
  * Nút Primary CTA: `Lưu cài đặt phần` (Nền tím SnapVocab `#3525cd`, chữ trắng, bo góc 12px, shadow tím).

---

## 3. Tổng Kết Vị Trí Canvas Trên Figma

Tất cả 7 màn hình hiện đang nằm liên tiếp theo luồng tương tác tự nhiên từ trái sang phải tại tọa độ `y = 6963` trên trang `SnapVocab Learn` (File Key: `unsaved-mu0lggkk-tebb5rd8`):
* `x = 2750`: **Danh sách BST** (`202:3`)
* `x = 3200`: **Tạo BST mới (Modal tối giản)** (`202:5`)
* `x = 3650`: **Chi tiết BST (Empty State + Mascot Cáo)** (`202:6`)
* `x = 4100`: **Thêm chủ đề (Thuần form điền + Nút Tùy chỉnh)** (`202:176`)
* `x = 4550`: **Bố cục thẻ học (Template Builder tối ưu Mobile)** (`202:177`)
* `x = 5000`: **Cài đặt chi tiết ô dữ liệu (Field Settings Bottom Sheet)** (`202:372`)
* `x = 5450`: **Cài đặt phần hiển thị (Section Settings Bottom Sheet)** (`202:745`)
