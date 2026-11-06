# Từ của tôi

`UserWord` lưu từ điển đã được user chủ động thêm: `id`, `user_id`, `word_id`,
`created_at`. Unique `(user_id, word_id)` ngăn lưu trùng. Xem chi tiết/lookup
không tự động thêm từ.

Các API đều yêu cầu đăng nhập và lấy user từ authentication, không nhận userId
từ client:

| API | Kết quả |
| --- | --- |
| `PUT /api/words/my/{wordId}` | Thêm từ, 204; gọi lại vẫn 204. Word không tồn tại trả 404. |
| `DELETE /api/words/my/{wordId}` | Xóa khỏi từ của tôi, 204 kể cả khi chưa lưu. |
| `GET /api/words/my?page=0&size=20` | `data: [{id, word}]` và `meta` phân trang; mặc định mới lưu trước. |

`GET /api/words/lookup?q=dog`, `POST /api/words/lookup` và
`GET /api/words/{id}` bổ sung `wordId` và `inMyVocabulary` (boolean).
Luồng scan dùng chung batch lookup, nên `items[].word` cũng có hai trường này;
trạng thái được đọc lại khi lấy kết quả scan. Từ scan không có trong từ điển
vẫn có `word: null` và không thể lưu bằng API này.

Frontend dùng `inMyVocabulary` để hiển thị nút thêm/xóa tại trang chi tiết từ,
gọi API với `wordId`, rồi cập nhật trạng thái sau khi thành công.
Danh sách từ đã lưu cung cấp id để lookup chi tiết và điền dữ liệu vào form
tạo TopicItem theo schema hiện tại. Lưu/xóa UserWord độc lập với TopicItem;
không tự tạo hoặc xóa TopicItem và không thay đổi tiến độ học.

Chạy `migration/migration_user_words.sql` trước khi triển khai ở staging/prod
(`ddl-auto: validate`). Dev dùng `ddl-auto: update` có thể tự tạo bảng.
