# CLEAN-BDG-01 — Kiểm tra tính nhất quán (2026-10-07)

Nguồn đối chiếu: [badges.md](../decisions/badges.md) rev 3, ACTIVE / AUTHORITATIVE, ghi nhận PO duyệt PD-BDG-01…12 và COPY-BDG-01 ngày 2026-10-07. Không thay đổi quyết định sản phẩm.

Kết quả: hoàn tất CLEAN-BDG-01 trong phạm vi §13. Đặc tả đủ để bắt đầu triển khai MVP; backend/mobile Badge chưa được triển khai và chưa nghiệm thu production.

| Phạm vi | Kết quả |
| :--- | :--- |
| FR, backlog, business flow, màn hình và phân hệ | Khớp catalog 12 badge/4 metric; sync qua POST, GET chỉ đọc; nhận một lần, không thu hồi/thưởng/notification; Admin CRUD hoãn |
| Database và SA | Ba bảng `badges`, `user_badges`, `user_badge_word_credits` khớp tên trường, kiểu dữ liệu và ràng buộc §7; không dùng bảng counter cũ |
| Mobile contract dẫn xuất | Progress giới hạn theo ngưỡng, tổng lấy từ API; không rarity, modal không ngắt phiên học; Profile theo `earnedAt` rồi `sortOrder` giảm dần |
| Admin mock | Gỡ lazy import và route trong `App.tsx`, mục menu và import icon trong `Sidebar.tsx`; giữ source prototype. Admin design ghi rõ ranh giới scope |
| Prototype mobile | Không sửa theo §13; không dùng làm contract triển khai |

Các chỉnh sửa bổ sung khi kiểm tra: tách Badge khỏi nhánh evaluation của learning flow trong SA; sửa E2E server thành nhánh sync độc lập; sửa ví dụ JSON để số đếm khớp danh sách; làm rõ `/seen` trả số unseen còn lại, không luôn bằng 0.

Kiểm chứng:

- Kiểm tra link tương đối trong 13 tài liệu liên quan: không có link file bị hỏng tại thời điểm kiểm tra.
- Đối chiếu trường của ba bảng giữa canonical §7 và `database.md`; kiểm tra kiểu dữ liệu, UNIQUE, FK, retention và seed/migration.
- Rà tham chiếu cũ trong tài liệu ACTIVE: không còn contract Badge dùng GET để cấp, `/admin/badges` hay nguồn schema chưa thiết kế. Tham chiếu cũ trong bảng mô tả việc đồng bộ là lịch sử chỉnh sửa.
- `npm.cmd run build` tại `snapAdmin`: TypeScript và Vite production build pass; không có chunk `BadgesPage` và không có chuỗi đặc trưng của Badge mock trong JS bundle.
- Không có import từ `features/badges` hoặc `domains/badges` ngoài source prototype được giữ.

Giới hạn công cụ: `graphify query` và `graphify update .` không chạy được vì launcher trỏ tới Python 3.11 không còn tồn tại. Graph chưa được cập nhật; kiểm tra dùng source trực tiếp. Chưa chạy nghiệm thu trên thiết bị hoặc staging, chưa triển khai Badge MVP.
