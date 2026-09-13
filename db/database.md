# Database Schema

Tài liệu này mô tả kiến trúc cơ sở dữ liệu của hệ thống SnapVocab.

## Quy chuẩn thiết kế (Conventions)

Để đảm bảo tính đồng nhất trên mọi môi trường và công cụ ORM (Hibernate), dự án áp dụng các quy ước cơ sở dữ liệu sau:

### 1. Encoding & Collation

- Charset: `utf8mb4` (hỗ trợ đầy đủ Unicode, emoji, đa ngôn ngữ).
- Collation: `utf8mb4_unicode_ci` hoặc `utf8mb4_0900_ai_ci` cho tất cả các bảng.

### 2. Naming Convention

- Bắt buộc sử dụng `snake_case` số nhiều cho tên bảng (ví dụ: `users`, `collections`, `topics`, `topic_items`, `templates`, `fsrs_records`).
- Bắt buộc sử dụng `snake_case` cho tên cột (`user_id`, `created_at`, `topic_item_id`).
- Khóa ngoại có tiền tố tên bảng tham chiếu kèm `_id` (ví dụ: `collection_id`, `topic_id`, `item_id`).

### 3. Auditing & Base Entities

Các bảng trong hệ thống áp dụng cơ chế kế thừa Auditing thông qua Spring Data JPA:
- `BaseTimeEntity`: Cung cấp 2 trường `created_at` (không cho phép update) và `updated_at` (tự động cập nhật khi sửa đổi).
- `BaseCreatedAtEntity`: Cung cấp trường `created_at` cho các bảng dữ liệu bất biến (append-only hoặc log/mapping).

### 4. Primary Key (PK) Strategy

- Các bảng dữ liệu chính sử dụng kiểu `BIGINT` (Long trong Java), khóa chính tự tăng (IDENTITY / AUTO_INCREMENT) để tối ưu hiệu năng join và đánh index.
- Bảng `authorities` sử dụng trực tiếp tên quyền dạng `VARCHAR(50)` làm khóa chính (ROLE_USER, ROLE_ADMIN).

### 5. Phạm vi chức năng

- Chức năng Community (bài đăng, nhóm học, chia sẻ xã hội) đã được lược bỏ hoàn toàn khỏi phạm vi sản phẩm và cơ sở dữ liệu.
- Hệ thống tập trung hoàn toàn vào luồng học cá nhân: Tra cứu/Quét ảnh -> Bộ sưu tập & Chủ đề (Collections/Topics) -> Thẻ học theo Template -> Lịch ôn tập Spaced Repetition (FSRS) -> Gamification cá nhân (Level, Shop, Inventory).

---

## 1. Core Domain (Authentication & Dictionary)

Quản lý người dùng và hệ thống từ điển nền tảng.

### Bảng `users`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID người dùng |
| `password_hash` | VARCHAR(60) | NOT NULL | Mật khẩu mã hóa BCrypt |
| `first_name` | VARCHAR(50) | NOT NULL | Tên |
| `last_name` | VARCHAR(50) | NULL | Họ và tên đệm |
| `email` | VARCHAR(254) | UNIQUE, NOT NULL | Địa chỉ email đăng nhập |
| `avatar_url` | VARCHAR(2048) | NULL | Đường dẫn ảnh đại diện |
| `native_language` | VARCHAR(10) | NULL | Ngôn ngữ mẹ đẻ (mặc định vi) |
| `learning_language` | VARCHAR(10) | NULL | Ngôn ngữ đang học (mặc định en) |
| `exp` | BIGINT | DEFAULT 0 | Điểm kinh nghiệm tích lũy |
| `coin` | BIGINT | DEFAULT 0 | Tiền tệ trong ứng dụng |
| `streak_days` | INT | DEFAULT 0 | Chuỗi ngày học liên tục |
| `last_studied_at` | DATETIME(6) | NULL | Thời điểm học gần nhất |
| `activated` | BOOLEAN | DEFAULT FALSE, NOT NULL | Trạng thái kích hoạt tài khoản |
| `bio` | TEXT | NULL | Giới thiệu ngắn |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `authorities`

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `name` | VARCHAR(50) | Khóa chính (PK) | Tên quyền (ROLE_USER, ROLE_ADMIN) |

### Bảng `user_authority` (Bảng liên kết N-N)

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `user_id` | BIGINT | FK -> `users(id)`, NOT NULL | ID người dùng |
| `authority_name` | VARCHAR(50) | FK -> `authorities(name)`, NOT NULL | Tên quyền |

Khóa chính ghép: `(user_id, authority_name)`.

### Bảng `words`
Kế thừa `BaseCreatedAtEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID từ vựng |
| `word` | VARCHAR(255) | NOT NULL | Nội dung từ vựng |
| `lang_code` | VARCHAR(20) | NOT NULL | Mã ngôn ngữ (en, vi...) |
| `is_deleted` | BOOLEAN | DEFAULT FALSE, NOT NULL | Cờ xóa mềm (soft-delete) |
| `deleted_at` | DATETIME(6) | NULL | Thời điểm xóa |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `definitions`
Kế thừa `BaseCreatedAtEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID định nghĩa |
| `definition` | LONGTEXT | NOT NULL | Nội dung định nghĩa |
| `pos` | VARCHAR(50) | NULL | Loại từ (Part of Speech) |
| `sub_pos` | VARCHAR(50) | NULL | Phân loại phụ của loại từ |
| `definition_lang` | VARCHAR(10) | NULL | Ngôn ngữ của định nghĩa |
| `links` | VARCHAR(2048) | NULL | Các liên kết mở rộng |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `word_definitions`
Kế thừa `BaseCreatedAtEntity`. Liên kết từ với định nghĩa kèm ví dụ minh họa.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID liên kết từ - định nghĩa |
| `word_id` | BIGINT | FK -> `words(id)`, NOT NULL | Tham chiếu đến từ vựng |
| `definition_id` | BIGINT | FK -> `definitions(id)`, NOT NULL | Tham chiếu đến định nghĩa |
| `example` | LONGTEXT | NULL | Ví dụ minh họa sử dụng từ |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `translations`
Kế thừa `BaseCreatedAtEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bản dịch |
| `word_id` | BIGINT | FK -> `words(id)`, NOT NULL | Tham chiếu đến từ gốc |
| `lang_code` | VARCHAR(20) | NOT NULL | Mã ngôn ngữ đích |
| `translation` | VARCHAR(1024) | NULL | Nội dung dịch nghĩa |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `pronunciations`
Kế thừa `BaseCreatedAtEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID phát âm |
| `word_id` | BIGINT | FK -> `words(id)`, NOT NULL | Tham chiếu đến từ gốc |
| `ipa` | VARCHAR(512) | NULL | Phiên âm quốc tế IPA |
| `region` | VARCHAR(100) | NULL | Vùng phát âm (UK, US...) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `word_relations`
Kế thừa `BaseCreatedAtEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID liên kết quan hệ từ |
| `word_id` | BIGINT | FK -> `words(id)`, NOT NULL | Tham chiếu đến từ gốc |
| `related_word` | VARCHAR(255) | NULL | Từ liên quan (dạng text) |
| `relation_type` | VARCHAR(50) | NULL | Loại quan hệ (synonym, antonym...) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Ràng buộc & Indexes (Core Domain)
- `users`: Unique index trên `email`.
- `words`: Unique constraint ghép trên `(word, lang_code)`. Kèm theo Index độc lập trên `word` (Partial Index `WHERE is_deleted = false`) để tra cứu nhanh.
- `user_authority`: Khóa chính ghép `(user_id, authority_name)`.
- `word_definitions`: Unique constraint ghép trên `(word_id, definition_id)` để tránh trùng lặp liên kết.
- Khóa ngoại (`word_id`, `definition_id`) đều có index và thiết lập `FetchType.LAZY`.

## 2. Collections, Topics & EAV Data Engine

Cấu trúc thu thập, tổ chức và quản lý dữ liệu từ vựng theo cấu trúc phân cấp linh hoạt: Collection -> Topic -> TopicSchema -> TopicItem. Dữ liệu chi tiết của từng từ vựng học tập được lưu theo mô hình EAV (Entity-Attribute-Value) để đáp ứng cấu trúc đa dạng của từng chủ đề. Mỗi Topic sở hữu đúng một TopicSchema để quản lý định nghĩa các thuộc tính.

### Bảng `collections`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bộ sưu tập |
| `name` | VARCHAR(255) | NOT NULL | Tên bộ sưu tập |
| `translation` | VARCHAR(255) | NULL | Tên dịch nghĩa |
| `type` | VARCHAR(20) | ENUM('SYSTEM', 'USER'), NOT NULL | Phân loại bộ sưu tập (hệ thống hoặc cá nhân) |
| `owner_id` | BIGINT | FK -> `users(id)`, NULLABLE | Người tạo (nếu type = USER) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `topics`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID chủ đề |
| `collection_id` | BIGINT | FK -> `collections(id)`, NOT NULL | Thuộc bộ sưu tập nào |
| `parent_id` | BIGINT | FK -> `topics(id)`, NULLABLE | Chủ đề cha (hỗ trợ phân cấp cây chủ đề) |
| `active_template_id` | BIGINT | FK -> `templates(id)`, NULLABLE | Template / Chế độ học đang kích hoạt cho Topic |
| `name` | VARCHAR(255) | NOT NULL | Tên chủ đề |
| `translation` | VARCHAR(255) | NULL | Dịch nghĩa chủ đề |
| `description` | TEXT | NULL | Mô tả chi tiết |
| `description_translation` | TEXT | NULL | Dịch nghĩa mô tả |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `schemas`
Kế thừa `BaseTimeEntity`. Định nghĩa lược đồ cấu trúc thuộc tính độc lập (1 Schema có thể được nhiều Topic sử dụng lại, sở hữu nhiều Template hiển thị).

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID lược đồ |
| `code` | VARCHAR(50) | UNIQUE, NULLABLE | Mã định danh chuẩn cho schema hệ thống (`DEFAULT_ENGLISH`...) |
| `name` | VARCHAR(255) | NOT NULL | Tên lược đồ (vd: Standard English, Kanji...) |
| `description` | TEXT | NULL | Mô tả chi tiết về lược đồ |
| `is_system` | BOOLEAN | DEFAULT FALSE, NOT NULL | Đánh dấu lược đồ mẫu mặc định của hệ thống |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `topic_schemas`
Kế thừa `BaseTimeEntity`. Đóng vai trò entity trung gian liên kết giữa Topic và Schema (Topic 1-1 TopicSchema N-1 Schema).

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bản ghi liên kết topic - schema |
| `topic_id` | BIGINT | FK -> `topics(id)`, UNIQUE, NOT NULL | Thuộc chủ đề nào (Quan hệ 1-1 với Topic) |
| `schema_id` | BIGINT | FK -> `schemas(id)`, NOT NULL | Áp dụng lược đồ nào (N-1 với Schema) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm gán |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `schema_attribute_groups`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID nhóm thuộc tính |
| `schema_id` | BIGINT | FK -> `schemas(id)`, NOT NULL | Thuộc schema độc lập nào |
| `name` | VARCHAR(255) | NOT NULL | Tên kỹ thuật của nhóm (main, examples...) |
| `label` | VARCHAR(255) | NULL | Nhãn hiển thị của nhóm |
| `multiple` | BOOLEAN | NOT NULL | Cho phép nhiều bản ghi lặp lại hay không |
| `position` | SMALLINT | NOT NULL | Vị trí hiển thị |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `schema_attributes`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID thuộc tính |
| `group_id` | BIGINT | FK -> `schema_attribute_groups(id)`, NOT NULL | Thuộc nhóm thuộc tính nào |
| `name` | VARCHAR(255) | NOT NULL | Tên kỹ thuật của thuộc tính |
| `label` | VARCHAR(255) | NULL | Nhãn hiển thị |
| `data_type` | VARCHAR(50) | NOT NULL | Kiểu dữ liệu (TEXT, AUDIO, IMAGE...) |
| `required` | BOOLEAN | NOT NULL | Bắt buộc hay không |
| `position` | SMALLINT | NOT NULL | Vị trí hiển thị trong nhóm |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `topic_items`
Kế thừa `BaseCreatedAtEntity`. Mỗi item đại diện cho một mục từ vựng học tập trong chủ đề.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID mục từ vựng |
| `topic_id` | BIGINT | FK -> `topics(id)`, NOT NULL | Thuộc chủ đề nào |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `topic_item_attribute_groups`

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID instance của nhóm cho 1 item |
| `topic_item_id` | BIGINT | FK -> `topic_items(id)`, NOT NULL | Item tương ứng |
| `group_definition_id` | BIGINT | FK -> `schema_attribute_groups(id)`, NOT NULL | Nhóm định nghĩa gốc |
| `position` | SMALLINT | NOT NULL | Thứ tự bản ghi (khi multiple = true) |

### Bảng `topic_item_attribute_values`
Kế thừa `BaseTimeEntity`.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID giá trị thuộc tính |
| `group_instance_id` | BIGINT | FK -> `topic_item_attribute_groups(id)`, NOT NULL | Instance nhóm tương ứng |
| `schema_attribute_id` | BIGINT | FK -> `schema_attributes(id)`, NOT NULL | Thuộc tính schema gốc |
| `value` | LONGTEXT | NOT NULL | Giá trị thực tế được lưu |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Ràng buộc & Indexes (Collections, Topics & EAV)
- `collections`: Index trên `owner_id` và `type`.
- `topics`: Index trên `collection_id`, `parent_id` và `active_template_id`.
- `schemas`: Unique index trên `code`.
- `topic_schemas`: Unique index trên `topic_id`, Index trên `schema_id`.
- `schema_attribute_groups`: Unique index ghép trên `(schema_id, name)`.
- `schema_attributes`: Unique index ghép trên `(group_id, name)`.
- `topic_items`: Index trên `topic_id` để phân trang và load item theo chủ đề.
- `topic_item_attribute_groups`: Index trên `topic_item_id`.
- `topic_item_attribute_values`: Index trên `group_instance_id` và `schema_attribute_id`.

### Cơ chế Quản lý Lược đồ: Copy-on-Write (Fork) & Additive Evolution
1. **Schema độc lập & Tích hợp Templates**: 1 Schema định nghĩa cấu trúc dữ liệu (`schema_attributes`) và chứa danh mục các mẫu hiển thị (`templates`) cho nhiều chế độ học (Standard, Listening, Reverse...).
2. **Topic chọn Template kích hoạt**: Topic liên kết với Schema qua `topic_schemas` và chọn một `active_template_id` để hiển thị trong các buổi học. Chuyển đổi chế độ học chỉ cập nhật `active_template_id`, không làm nhân bản dữ liệu EAV hay lịch ôn FSRS.
3. **Tiến hóa mở rộng (Additive Evolution)**: Đối với Schema dùng chung, chỉ cho phép **thêm mới** các nhóm/thuộc tính (thuộc tính mới là tùy chọn). Không cho phép xóa thuộc tính nếu thuộc tính đó đang được liên kết bởi `topic_item_attribute_values` hoặc `template_fields`.
4. **Copy-on-Write (Fork Schema & Templates)**: Khi người dùng muốn tùy biến sâu cấu trúc thuộc tính hoặc sửa layout template cho riêng một Topic, hệ thống hỗ trợ **Fork** Schema dùng chung thành một Schema riêng biệt cho Topic đó (`POST /api/topics/{topicId}/schema/fork`). Hệ thống nhân bản đồng thời Schema, các Groups, Attributes, toàn bộ Templates, Elements và Fields (re-map attribute ID tương ứng), đồng thời cập nhật `topic.active_template_id` và re-map dữ liệu EAV trong 1 transaction duy nhất.

## 3. Flashcard Templates & Spaced Repetition (SRS)

Mô hình **Frappe-style Unified Studio**: `Template` thuộc về `Schema` (`schemas (1) --- (N) templates`). Mỗi template đại diện cho một cách thức hiển thị thẻ học (chế độ học Standard, Listening, Reverse...) của bộ thuộc tính Schema đó. Mỗi `Topic` trỏ tới `active_template_id` của Schema tương ứng để quyết định giao diện hiển thị khi ôn tập.

Tiến trình ôn tập ngắt quãng Spaced Repetition được quản lý thông qua thuật toán FSRS trực tiếp trên từng mục từ vựng học tập của người dùng qua bảng `fsrs_records`.

### Bảng `templates`
Kế thừa `BaseTimeEntity`. Định nghĩa mẫu hiển thị thẻ học cho Schema.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID mẫu thẻ học |
| `schema_id` | BIGINT | FK -> `schemas(id)`, NOT NULL | Thuộc lược đồ nào |
| `code` | VARCHAR(50) | NULLABLE | Mã định danh chế độ học (`STANDARD`, `LISTENING`, `REVERSE`...) |
| `name` | VARCHAR(255) | NOT NULL | Tên mẫu thẻ ("Thẻ Chuẩn", "Luyện Nghe", "Đảo Chiều") |
| `is_default` | BOOLEAN | DEFAULT FALSE, NOT NULL | Đánh dấu template mặc định ban đầu của Schema |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `template_elements`
Các thành phần hiển thị trên template theo kiến trúc bố cục Frappe (Layout elements), sắp xếp theo thứ tự hiển thị.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID thành phần |
| `template_id` | BIGINT | FK -> `templates(id)`, NOT NULL | Thuộc template nào |
| `position` | INT | NOT NULL | Vị trí hiển thị trên thẻ (0, 1, 2...) |
| `type` | VARCHAR(50) | ENUM, NOT NULL | Loại phần tử: `FIELD`, `SECTION_BREAK`, `COLUMN_BREAK` |

Ràng buộc duy nhất: UNIQUE(`template_id`, `position`).

### Bảng `template_fields`
Cấu hình chi tiết cho phần tử dạng FIELD, ánh xạ trực tiếp thuộc tính Schema với vai trò ngữ nghĩa (`SemanticRole`) và styling trên thẻ học.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID cấu hình trường |
| `element_id` | BIGINT | FK -> `template_elements(id)`, UNIQUE, NOT NULL | Phần tử template tương ứng (quan hệ 1-1) |
| `schema_attribute_id` | BIGINT | FK -> `schema_attributes(id)`, NOT NULL | Thuộc tính Schema được lấy dữ liệu (cùng Schema) |
| `semantic_role` | VARCHAR(50) | ENUM, NULLABLE | Vai trò ngữ nghĩa trên thẻ học |
| `field_label` | VARCHAR(255) | NULL | Nhãn trường khi render |
| `hide_if_empty` | BOOLEAN | DEFAULT FALSE, NOT NULL | Tự động ẩn nếu giá trị rỗng |
| `audio_action` | BOOLEAN | DEFAULT FALSE, NOT NULL | Cho phép bấm phát âm thanh |
| `font_size` | INT | NULL | Cỡ chữ tùy chỉnh |
| `alignment` | VARCHAR(20) | ENUM('LEFT', 'CENTER', 'RIGHT', 'JUSTIFY'), NULL | Căn lề |
| `color` | VARCHAR(50) | NULL | Mã màu hex hiển thị |

#### Danh sách SemanticRole (`vn.ptit.snapvocab.domain.enumeration.SemanticRole`):
- `TARGET_WORD`: Từ vựng mục tiêu, câu hỏi chính ở mặt trước thẻ.
- `EXAMPLE_SENTENCE`: Câu ví dụ minh họa hoặc ngữ cảnh sử dụng.
- `NATIVE_TRANSLATION`: Bản dịch nghĩa tiếng mẹ đẻ (tiếng Việt).
- `DEFINITION`: Định nghĩa / giải nghĩa chính của từ vựng.
- `AUDIO`: Dữ liệu âm thanh / phát âm.
- `IMAGE`: Hình ảnh minh họa trực quan.

### Bảng `fsrs_records`
Kế thừa `BaseTimeEntity`. Quản lý tiến trình ôn tập ngắt quãng theo thuật toán FSRS cho từng cặp (user, topic_item).

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bản ghi FSRS |
| `user_id` | BIGINT | FK -> `users(id)`, NOT NULL | Người học |
| `topic_item_id` | BIGINT | FK -> `topic_items(id)`, NOT NULL | Mục từ vựng học tập |
| `state` | TINYINT / ENUM | NOT NULL | Trạng thái: 0:NEW, 1:LEARNING, 2:REVIEW, 3:RELEARNING, 4:SUSPENDED |
| `due` | DATETIME(6) | NOT NULL | Thời điểm đến hạn ôn tập tiếp theo |
| `stability` | FLOAT | NOT NULL | Độ bền trí nhớ (S) tính theo ngày |
| `difficulty` | FLOAT | NOT NULL | Độ khó của thẻ (D) từ 1.0 đến 10.0 |
| `reps` | INT | DEFAULT 0, NOT NULL | Tổng số lần ôn tập thành công |
| `lapses` | INT | DEFAULT 0, NOT NULL | Số lần quên (đánh giá Again trong phase Review) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm bắt đầu học |
| `updated_at` | DATETIME(6) | NOT NULL | Lần cập nhật tham số SRS gần nhất |

### Ràng buộc & Indexes (Templates & SRS)
- `templates`: Index trên `schema_id`, Unique index trên `(schema_id, code)`.
- `template_elements`: Unique constraint ghép `(template_id, position)`.
- `template_fields`: Unique constraint trên `element_id`. Index trên `schema_attribute_id`.
- `fsrs_records`: Unique constraint ghép `(user_id, topic_item_id)` đảm bảo mỗi người học có đúng 1 tiến trình cho mỗi mục từ.
- `fsrs_records`: Index ghép trên `(user_id, due, state)` phục vụ query lấy danh sách thẻ đến hạn ôn tập cực nhanh.

## 4. Daily Mission & Gamification

Thiết kế từ `daily_mission.md` nhằm thúc đẩy duy trì thói quen học hàng ngày.

### Bảng `MissionTemplate`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID Template nhiệm vụ |
| `code` | String | Unique | Mã template |
| `name` | String | | Tên nhiệm vụ hiển thị |
| `description` | String | | Mô tả ngắn |
| `type` | Enum | | DAILY, BONUS, CHEST |
| `category` | Enum | | Nhóm nhiệm vụ (SCAN, VOCABULARY...) |
| `triggerEvent` | String | | Event dùng để cập nhật tiến độ |
| `targetValue` | Number | | Chỉ tiêu số lượng cần đạt |
| `rewardCoin` | Number | | Thưởng xu |
| `rewardXp` | Number | | Thưởng điểm XP |
| `rewardItemCode`| String | Nullable | Vật phẩm thưởng nếu có |
| `weight` | Number | | Trọng số random |
| `eligibilityRule`| JSON | | Rule lọc theo ngữ cảnh |
| `isActive` | Boolean| | Bật/tắt template |
| `createdAt` | DateTime| | Thời điểm tạo |
| `updatedAt` | DateTime| | Thời điểm cập nhật |

### Bảng `UserDailyMission`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID Nhiệm vụ của User |
| `userId` | Long | FK -> `User(id)` | Của người dùng nào |
| `missionTemplateId`| UUID | FK -> `MissionTemplate(id)`| Sử dụng template nào |
| `missionDate` | Date | | Ngày cấp nhiệm vụ |
| `slot` | Number | | Vị trí hiển thị (1-5, hoặc 6) |
| `targetValue` | Number | | Target Snapshot |
| `currentProgress`| Number | | Tiến độ đã đạt được |
| `status` | Enum | | IN_PROGRESS, COMPLETED, CLAIMED, EXPIRED |
| `snapshotData` | JSON | | Dữ liệu snapshot |
| `assignedAt` | DateTime| | Thời điểm cấp mission |
| `completedAt` | DateTime| Nullable | Thời điểm hoàn thành |
| `claimedAt` | DateTime| Nullable | Thời điểm nhận thưởng |
| `updatedAt` | DateTime| | Thời điểm cập nhật |

### Bảng `UserDailyMissionClaimLog`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID Log nhận thưởng |
| `userDailyMissionId`| UUID | FK -> `UserDailyMission(id)`, Unique | Nhận thưởng từ nhiệm vụ nào |
| `userId` | Long | FK -> `User(id)` | Người nhận thưởng |
| `idempotencyKey`| String | Unique | Key chống nhận thưởng nhiều lần |
| `rewardCoin` | Number | | Số xu thực tế được cộng |
| `rewardXp` | Number | | Điểm XP thực tế được cộng |
| `rewardItemCode`| String | Nullable | Vật phẩm đã cộng |
| `claimedBy` | Enum | | Ghi nhận người claim (USER) |
| `createdAt` | DateTime| | Thời điểm claim |

### Bảng `UserDailyChest`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID Rương ngày |
| `userId` | Long | FK -> `User(id)` | Người nhận rương |
| `chestDate` | Date | | Ngày của rương |
| `status` | Enum | LOCKED, UNLOCKED, CLAIMED | Trạng thái rương |
| `claimedAt` | DateTime| Nullable | Thời điểm nhận thưởng |
| `createdAt` | DateTime| | Thời điểm tạo |
| `updatedAt` | DateTime| | Thời điểm cập nhật |
### Bảng `UserWeeklyMilestone`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID Cột mốc tiến trình tuần |
| `userId` | Long | FK -> `User(id)` | Người dùng |
| `weekStartDate` | Date | | Ngày đầu tuần (Thứ Hai) |
| `weekEndDate` | Date | | Ngày cuối tuần (Chủ Nhật) |
| `activityStampCount`| Number | | Số ngày nhận rương Daily Chest trong tuần |
| `bronzeStatus` | Enum | | Rương Đồng (LOCKED, UNLOCKED, CLAIMED) |
| `silverStatus` | Enum | | Rương Bạc |
| `goldStatus` | Enum | | Rương Vàng |
| `updatedAt` | DateTime| | Thời điểm cập nhật |

### Bảng `ExperienceLog`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID log nhận XP |
| `userId` | Long | FK -> `User(id)` | Người nhận XP |
| `amount` | Number | Not Null | Số lượng XP nhận được |
| `sourceType` | Enum | | Nguồn nhận (MISSION, QUIZ, SCAN, REVIEW...) |
| `eventKey` | String | Unique | Khóa chống cộng trùng XP từ 1 event |
| `createdAt` | DateTime| | Thời điểm nhận |

### Bảng `CoinTransaction`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Khóa chính (PK) | ID giao dịch Coin |
| `userId` | Long | FK -> `User(id)` | Người nhận/tiêu Coin |
| `amount` | Number | Not Null | Số lượng Coin (dương = nhận, âm = tiêu) |
| `sourceType` | Enum | | Nguồn (MISSION, WEEKLY_CHEST, SHOP_BUY...) |
| `eventKey` | String | Unique | Khóa chống giao dịch trùng từ 1 event |
| `createdAt` | DateTime| | Thời điểm giao dịch |

### Bảng `levels`
Định nghĩa các mốc cấp độ theo điểm kinh nghiệm (EXP).

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | INT | Khóa chính (PK), AUTO_INCREMENT | ID cấp độ |
| `level_number` | INT | UNIQUE, NOT NULL | Số cấp độ (1, 2, 3...) |
| `required_exp` | BIGINT | NOT NULL | Mốc EXP cần đạt để lên cấp |
| `title` | VARCHAR(100) | NULL | Danh hiệu người học ở cấp độ này |

### Bảng `shop_items`
Kế thừa `BaseTimeEntity`. Quản lý danh mục vật phẩm bày bán trong cửa hàng.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID vật phẩm |
| `name` | VARCHAR(255) | NOT NULL | Tên vật phẩm |
| `type` | VARCHAR(50) | NOT NULL | Phân loại vật phẩm (THEME, AVATAR_FRAME, STREAK_FREEZE...) |
| `price` | BIGINT | NOT NULL | Giá bán (tính bằng Coin) |
| `is_active` | BOOLEAN | DEFAULT TRUE, NOT NULL | Cờ kích hoạt bày bán |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |
| `updated_at` | DATETIME(6) | NOT NULL | Thời điểm cập nhật |

### Bảng `user_inventories`
Quản lý kho đồ, tài sản vật phẩm mà người học sở hữu.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bản ghi kho đồ |
| `user_id` | BIGINT | FK -> `users(id)`, NOT NULL | Người sở hữu |
| `item_id` | BIGINT | FK -> `shop_items(id)`, NOT NULL | Vật phẩm sở hữu |
| `quantity` | INT | DEFAULT 0, NOT NULL | Số lượng sở hữu |
| `acquired_at` | DATETIME(6) | NOT NULL | Thời điểm nhận vật phẩm |

Ràng buộc duy nhất: UNIQUE(`user_id`, `item_id`).

### Ràng buộc & Indexes (Gamification & Shop)
- `levels`: Unique index trên `level_number`.
- `shop_items`: Index trên `is_active` và `type`.
- `user_inventories`: Unique index ghép trên `(user_id, item_id)`.
- Leaderboard: Quản lý qua Redis Sorted Set với điểm số là Weekly XP, nguồn dữ liệu tham chiếu và đồng bộ từ trường `exp` trên bảng `users`.

---

## 5. Media & AI Recognition

Quản lý lưu trữ tệp tin và tiến trình nhận diện hình ảnh.

### Lưu trữ tệp tin (Object Storage)
- Tệp tin avatar, ảnh quét gốc và tài nguyên tĩnh được lưu trữ trực tiếp trên Object Storage (MinIO cho dev/staging, Cloudflare R2 cho production).
- Database lưu trữ trực tiếp URL truy cập hoặc object key (`avatar_url` trên bảng `users`, URL ảnh crop và âm thanh trong bảng giá trị EAV `topic_item_attribute_values`).

---

## 6. Notifications

Quản lý thông báo trong ứng dụng gửi đến người học.

### Bảng `notifications`
Kế thừa `BaseCreatedAtEntity`. Nội dung thông báo hệ thống.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID thông báo |
| `title` | VARCHAR(255) | NOT NULL | Tiêu đề thông báo |
| `content` | LONGTEXT | NOT NULL | Nội dung thông báo |
| `type` | VARCHAR(50) | NULL | Phân loại thông báo (SYSTEM, REMINDER, REWARD...) |
| `created_at` | DATETIME(6) | NOT NULL | Thời điểm tạo |

### Bảng `user_notifications`
Phân phối thông báo tới từng người học.

| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | Khóa chính (PK), AUTO_INCREMENT | ID bản ghi |
| `user_id` | BIGINT | FK -> `users(id)`, NOT NULL | Người nhận |
| `notification_id` | BIGINT | FK -> `notifications(id)`, NOT NULL | Thông báo gốc |
| `is_read` | BOOLEAN | DEFAULT FALSE, NOT NULL | Trạng thái đã đọc |
| `read_at` | DATETIME(6) | NULL | Thời điểm đọc |

Ràng buộc duy nhất: UNIQUE(`user_id`, `notification_id`).
Index: Ghép trên `(user_id, is_read)` và `(user_id, notification_id)`.

---

## 7. Các truy vấn nóng & Chiến lược tối ưu (Hot Queries)

Các truy vấn có tần suất gọi cao được thiết kế index và mô hình tối ưu:

1. **Lấy hàng đợi ôn tập FSRS của người dùng:**
   - Mục đích: Lấy danh sách các mục từ vựng đến hạn ôn tập cho buổi học Spaced Repetition.
   - Query:
     ```sql
     SELECT * FROM fsrs_records 
     WHERE user_id = ? AND due <= ? AND state IN (1, 2, 3) 
     ORDER BY due ASC;
     ```
   - Tối ưu: Index ghép `(user_id, due, state)` trên bảng `fsrs_records`. Truy vấn đọc trực tiếp từ 1 bảng duy nhất, không cần join lồng nhiều tầng.

2. **Lấy dữ liệu phân trang EAV cho Topic (Two-step EAV Pagination):**
   - Thách thức: Mô hình EAV với các nhóm lặp (multiple attribute groups) nếu dùng JOIN 1 câu duy nhất sẽ sinh ra Cartesian product khổng lồ và sai lệch số lượng phân trang.
   - Giải pháp:
     - Bước 1: Query phân trang chỉ lấy danh sách `id` từ `topic_items` theo `topic_id`.
     - Bước 2: Batch query danh sách phẳng các thuộc tính từ `topic_item_attribute_groups` và `topic_item_attribute_values` theo danh sách `topic_item_id` của trang hiện tại, sau đó gom nhóm trực tiếp trên memory backend.

3. **Tra cứu từ vựng từ điển nhanh (Dictionary Lookup):**
   - Query:
     ```sql
     SELECT * FROM words WHERE word = ? AND lang_code = 'en' AND is_deleted = false;
     ```
   - Tối ưu: Unique index ghép trên `(word, lang_code)` trên bảng `words`.

4. **Lấy danh sách bộ sưu tập người dùng (My Collections):**
   - Query:
     ```sql
     SELECT * FROM collections WHERE (type = 'SYSTEM' OR (type = 'USER' AND owner_id = ?)) ORDER BY id ASC;
     ```
   - Tối ưu: Index trên `(type, owner_id)`. Thêm caching ở tầng Spring Cache / Redis.
