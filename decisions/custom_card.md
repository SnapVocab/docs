# Đặc tả Kiến trúc: Schema-Template Studio & Thẻ Flashcard Tùy chỉnh (Custom Card)

Tài liệu này mô tả chi tiết kiến trúc **Schema-Template System** (theo phong cách Frappe DocType & Layout) — giải pháp cho phép hệ thống và người học định nghĩa cấu trúc dữ liệu thuộc tính động (`Schema`) và tùy biến đa dạng cách thức hiển thị thẻ flashcard (`Template`) khi ôn tập các mục từ vựng (`TopicItem`) trong từng chủ đề (`Topic`), kết hợp các vai trò ngữ nghĩa (`SemanticRole`).

---

## 1. Tổng quan

### 1.1. Vấn đề cần giải quyết

Hệ thống flashcard dạng truyền thống thường áp dụng cấu trúc cứng nhắc: mặt trước luôn là từ vựng, mặt sau luôn là định nghĩa cố định. Cách tiếp cận này bộc lộ những hạn chế lớn:

1. **Không thích ứng với thuộc tính động**: Mỗi chủ đề học tập (`Topic`) có thể có bộ thuộc tính riêng (phiên âm, giải nghĩa tiếng Việt, câu ví dụ, ngữ cảnh, hình ảnh minh họa, file phát âm...). Cấu trúc cứng không thể phản ánh đầy đủ mô hình dữ liệu EAV của hệ thống.
2. **Thiếu linh hoạt trong trải nghiệm học (Đa chế độ học)**: Một bộ từ vựng cần hỗ trợ nhiều cách học khác nhau (học nhận diện chữ - nghĩa truyền thống, học luyện nghe qua phát âm, học đảo chiều đoán từ từ nghĩa) mà không được nhân bản dữ liệu từ vựng hay làm phân mảnh tiến trình ôn tập SRS.
3. **Phân tán cấu hình**: Nếu tách rời cấu hình Schema (trường dữ liệu) và Template (giao diện hiển thị) sang hai màn hình/entity độc lập không có ràng buộc chặt chẽ, người dùng phải chuyển qua lại nhiều bước, dễ gây lỗi ánh xạ (template trỏ tới trường không tồn tại).

### 1.2. Giải pháp: Frappe-style Unified Schema-Template Studio

SnapVocab áp dụng mô hình **Unified Studio** (tương tự kiến trúc DocType & Form/Print Format của Frappe Framework):

- **Template gắn liền với Schema (`schemas (1) --- (N) templates`)**: Một `Schema` định nghĩa cấu trúc dữ liệu (Data Contract gồm các nhóm và thuộc tính). Mỗi `Schema` sở hữu một hoặc nhiều `Template` quy định các góc nhìn hiển thị (View/Layout) khác nhau của chính bộ thuộc tính đó.
- **Bảo toàn toàn vẹn dữ liệu 100%**: Vì `Template` thuộc về `Schema`, các phần tử `TemplateField` chỉ được phép trỏ tới các thuộc tính `SchemaAttribute` thuộc cùng Schema đó. Hoàn toàn không bao giờ xảy ra tình trạng "trỏ nhầm thuộc tính của schema khác".
- **Hỗ trợ Đa chế độ học (Multi-mode Learning)**: Một Schema có thể có nhiều template ứng với các chế độ học khác nhau:
  - `STANDARD`: Thẻ chuẩn (Mặt trước: Từ vựng + Phiên âm + Audio; Mặt sau: Nghĩa + Ví dụ + Ảnh).
  - `LISTENING`: Luyện nghe (Mặt trước: Audio; Mặt sau: Từ vựng + Phiên âm + Nghĩa + Ví dụ).
  - `REVERSE`: Đảo chiều (Mặt trước: Nghĩa + Ảnh; Mặt sau: Từ vựng + Phiên âm + Audio).
- **Topic chọn Chế độ học (`topic.active_template_id`)**: `Topic` liên kết với `Schema` (qua `TopicSchema`), và trỏ tới một `Template` hiện hành (`activeTemplate`) của Schema đó. Chuyển đổi chế độ học chỉ đơn giản là đổi `active_template_id` trên Topic, dữ liệu từ vựng EAV (`topic_items`) và tiến trình FSRS (`fsrs_records`) được giữ nguyên 100%.
- **Nhận diện hệ thống bằng Mã chuẩn (`code`)**: Các Schema và Template chuẩn của hệ thống được xác định bằng `code` chuỗi bất biến (ví dụ: `DEFAULT_ENGLISH`, `STANDARD`, `LISTENING`, `REVERSE`), tuyệt đối không phụ thuộc vào ID tự tăng của database.
- **Copy-on-Write (Fork) tự động**: Khi người dùng muốn tùy biến sâu cấu trúc thuộc tính hoặc layout thẻ cho riêng Topic của mình, hệ thống thực hiện nhân bản (fork) đồng thời cả Schema và toàn bộ Template của nó thành một bản sao độc lập cho Topic.

| Khía cạnh | Mô hình cũ (Cố định) | Mô hình Schema-Template hiện tại |
| :--- | :--- | :--- |
| Phạm vi áp dụng | Toàn bộ thẻ chung một khuôn | Schema quản lý data contract + danh sách Templates hiển thị |
| Quan hệ Template | Gắn cứng vào Topic | Gắn vào Schema (`schemas (1) --- (N) templates`), Topic chọn `active_template_id` |
| Chế độ học | Chỉ có 1 cách hiển thị duy nhất | Đa chế độ (Standard, Listening, Reverse) trên cùng 1 bộ từ vựng |
| Định danh hệ thống | ID số tự tăng (dễ lệch giữa các env) | Mã chuẩn hóa bất biến (`code`: `DEFAULT_ENGLISH`, `STANDARD`...) |
| Quản lý tiến độ | Bảng Card riêng lẻ | `FsrsRecord` gắn trực tiếp cặp `(user_id, topic_item_id)` độc lập với view |

### 1.3. Vị trí trong hệ thống

Chức năng này thuộc phân hệ **Learning Engine**, cung cấp cấu hình hiển thị thẻ cho module Flashcard và thuật toán lặp lại ngắt quãng FSRS (`FsrsRecord`).

---

## 2. Khái niệm cốt lõi

### 2.1. Lược đồ Thuộc tính (`Schema`, `TopicSchema` & `SchemaAttribute`)

Cấu trúc thuộc tính được tổ chức theo mô hình độc lập và tái sử dụng:
- **`Schema` độc lập**: Định nghĩa cấu trúc khung gồm các nhóm thuộc tính (`SchemaAttributeGroup`), thuộc tính (`SchemaAttribute`) và các mẫu hiển thị (`Template`). Một `Schema` có thể được dùng chung cho hàng ngàn chủ đề (`Topic`).
  - `code`: Mã định danh chuẩn cho các schema hệ thống (ví dụ: `DEFAULT_ENGLISH`). Với schema người dùng tạo, trường này có thể là `null`.
  - `is_system`: Cờ đánh dấu schema mặc định của hệ thống (`true`/`false`).
- **`TopicSchema` trung gian**: Mỗi `Topic` liên kết 1-1 với một bản ghi `TopicSchema`, bản ghi này trỏ khóa ngoại `schema_id` tới `Schema` (Quan hệ `Topic (1) --- (1) TopicSchema (N) --- (1) Schema`).
- Mỗi nhóm (`SchemaAttributeGroup`) chứa các thuộc tính (`SchemaAttribute`) xác định tên thuộc tính, nhãn hiển thị (`label`), kiểu dữ liệu (`dataType`), thứ tự (`position`) và trạng thái bắt buộc (`required`).
- Các mục từ trong chủ đề (`TopicItem`) lưu giá trị thực tế tương ứng trong bảng `topic_item_attribute_values`.

### 2.2. Mẫu hiển thị Thẻ (`Template`, `TemplateElement` & `TemplateField`)

Mỗi `Template` liên kết trực tiếp với `Schema` qua trường `schema_id`:
- **Định danh Template**:
  - `name`: Tên hiển thị (ví dụ: "Thẻ Tiêu chuẩn", "Luyện nghe", "Đảo chiều").
  - `code`: Mã nhận diện chuẩn (ví dụ: `STANDARD`, `LISTENING`, `REVERSE`).
  - `is_default`: Đánh dấu template mặc định sẽ được chọn khi Topic mới được tạo.
- **`TemplateElement`**: Khối phần tử layout trên thẻ:
  - `position`: Thứ tự hiển thị tăng dần từ trên xuống dưới.
  - `type`: Phân loại phần tử gồm:
    - `FIELD`: Trường dữ liệu hiển thị (liên kết 1-1 với `TemplateField`).
    - `SECTION_BREAK`: Phân tách giữa các phần (ví dụ: phân cách Mặt trước / Mặt sau).
    - `COLUMN_BREAK`: Phân chia cột hiển thị linh hoạt (theo chuẩn Frappe layout).
- **`TemplateField`**: Cấu hình chi tiết cho phần tử kiểu `FIELD`:
  - `schema_attribute_id`: Khóa ngoại tham chiếu trực tiếp đến `SchemaAttribute` của cùng Schema.
  - `semantic_role`: Vai trò ngữ nghĩa hiển thị trên thẻ.
  - `field_label`: Nhãn tùy chỉnh hiển thị trước giá trị.
  - `hide_if_empty`: Nếu giá trị rỗng thì tự động ẩn khỏi thẻ.
  - `audio_action`: Nhấn vào trường này sẽ phát âm thanh.
  - `font_size`, `alignment` (`LEFT`, `CENTER`, `RIGHT`), `color`: Thuộc tính định dạng trực quan.

### 2.3. Vai trò Ngữ nghĩa (`SemanticRole`)

Để ứng dụng di động hiểu được ý nghĩa hiển thị mà không cần hardcode tên trường, mỗi `TemplateField` được gán một `SemanticRole`:

| SemanticRole | Ý nghĩa | Ứng dụng hiển thị trên thẻ |
| :--- | :--- | :--- |
| `TARGET_WORD` / `FRONT` | Từ vựng mục tiêu / câu hỏi chính | Từ vựng, thuật ngữ chính cần học ghi nhớ |
| `DEFINITION` / `BACK` | Định nghĩa / giải nghĩa chính | Giải nghĩa từ vựng |
| `NATIVE_TRANSLATION` | Bản dịch nghĩa tiếng mẹ đẻ | Nghĩa tiếng Việt bổ trợ |
| `EXAMPLE_SENTENCE` | Câu ví dụ ngữ cảnh | Câu ví dụ minh họa |
| `AUDIO` | Dữ liệu âm thanh phát âm | Nút nghe hoặc tự động phát âm khi lật thẻ |
| `IMAGE` | Hình ảnh minh họa | Ảnh minh họa ở vị trí trực quan của thẻ |
| `PHONETIC` | Phiên âm quốc tế | Ký hiệu ngữ âm IPA |
| `HINT` | Gợi ý khi cần | Ẩn mặc định, mở khi bấm trợ giúp |
| `TAG` | Thẻ phân loại hoặc cấp độ | Phân loại từ (noun, verb), cấp độ (A1, B2) |
| `EXTRA` | Thông tin bổ sung | Từ đồng nghĩa, trái nghĩa, ghi chú cá nhân |

---

## 3. Cơ chế hoạt động

### 3.1. Luồng Unified Studio (Thiết kế Schema & Template hợp nhất)

```
                       [ Unified Studio ]
          ┌────────────────────────────────────────┐
          │  1. Định nghĩa cấu trúc Schema:        │
          │     - Thuộc tính: word, ipa, meaning.. │
          │                                        │
          │  2. Thiết kế các Templates hiển thị:   │
          │     - STANDARD (Default)               │
          │     - LISTENING                        │
          │     - REVERSE                          │
          └──────────────────┬─────────────────────┘
                             │
                             ▼
         Lưu trữ vào CSDL (schemas & templates gắn kết)
                             │
                             ▼
         Người dùng tạo Topic mới:
         - Gán Schema (mặc định: DEFAULT_ENGLISH)
         - Gán active_template_id (mặc định: STANDARD)
                             │
                             ▼
         Nhập từ vựng vào Topic (TopicItem & EAV Values)
                             │
                             ▼
         Khi học: Đổi chế độ học (Standard / Listening / Reverse)
         => Chỉ cần cập nhật topic.active_template_id!
```

### 3.2. Luồng render thẻ trong phiên học Flashcard

```
Learner mở phiên ôn tập cho Topic
                 │
                 ▼
Backend truy vấn FSRS Records đến hạn ôn
(WHERE user_id = :userId AND topic_item_id IN (...) AND due <= NOW())
                 │
                 ▼
Backend xác định active_template của Topic (hoặc template theo mode yêu cầu)
(Tải danh sách TemplateElement và TemplateField theo position ASC)
                 │
                 ▼
Backend map dữ liệu EAV của từng TopicItem vào các trường của Template
                 │
                 ▼
Mobile Client render Flashcard theo SemanticRole:
├─ Mặt trước (Front): Các element trước SECTION_BREAK (hoặc TARGET_WORD, AUDIO...)
├─ Mặt sau (Back): Các element sau SECTION_BREAK (hoặc DEFINITION, TRANSLATION, EXAMPLE...)
                 │
                 ▼
Learner đánh giá độ nhớ (Again, Hard, Good, Easy) -> Cập nhật FsrsRecord
```

### 3.3. Xử lý dữ liệu khuyết thiếu & Thay đổi thuộc tính

1. **Ẩn trường trống (`hide_if_empty = true`)**: Khi một `TopicItem` không có giá trị cho một thuộc tính tùy chọn (ví dụ không có ví dụ hay hình ảnh), ứng dụng tự động bỏ qua khối element đó, giao diện thẻ tự động co giãn tự nhiên.
2. **Thay đổi cấu hình Template**: Khi cập nhật Template của Topic (thay đổi thứ tự `position`, đổi vai trò `semantic_role` hoặc màu sắc, kích cỡ chữ), toàn bộ các `TopicItem` thuộc Topic lập tức được áp dụng giao diện mới trong phiên học tiếp theo mà không cần cập nhật dữ liệu từng item.
3. **Tiến độ FSRS độc lập**: Trạng thái học tập của từng từ (`fsrs_records`) hoàn toàn độc lập với việc thay đổi giao diện thẻ, bảo đảm dữ liệu ghi nhớ không bị ảnh hưởng khi tinh chỉnh layout.

---

## 4. Mô hình Dữ liệu

### 4.1. Chi tiết các bảng liên quan

#### Bảng `schemas`

Lưu lược đồ thuộc tính độc lập và danh mục template của lược đồ.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh lược đồ |
| `code` | `varchar(50)` | UNIQUE, NULLABLE | Mã định danh chuẩn cho schema hệ thống (`DEFAULT_ENGLISH`...) |
| `name` | `varchar(255)` | NOT NULL | Tên lược đồ (ví dụ: "Tiếng Anh Chuẩn", "Kanji Nhật") |
| `description` | `text` | NULLABLE | Mô tả chi tiết về lược đồ |
| `is_system` | `bit(1)` | NOT NULL, DEFAULT 0 | Đánh dấu lược đồ mẫu mặc định của hệ thống |
| `created_at` | `datetime(6)` | NOT NULL | Thời điểm tạo |
| `updated_at` | `datetime(6)` | NOT NULL | Thời điểm cập nhật |

#### Bảng `templates`

Lưu cấu hình template hiển thị của Schema (hỗ trợ nhiều mode học khác nhau).

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh template |
| `schema_id` | `bigint(20)` | FK -> `schemas(id)`, NOT NULL | Lược đồ sở hữu template này |
| `code` | `varchar(50)` | NULLABLE | Mã nhận diện mode học (`STANDARD`, `LISTENING`, `REVERSE`...) |
| `name` | `varchar(255)` | NOT NULL | Tên template ("Thẻ Chuẩn", "Luyện nghe", "Đảo chiều") |
| `is_default` | `bit(1)` | NOT NULL, DEFAULT 0 | Đánh dấu template mặc định được chọn khi tạo Topic |
| `created_at` | `datetime(6)` | NOT NULL | Thời điểm tạo |
| `updated_at` | `datetime(6)` | NOT NULL | Thời điểm cập nhật |

#### Bảng `topics` (Trích đoạn các trường liên quan)

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh chủ đề |
| `active_template_id` | `bigint(20)` | FK -> `templates(id)`, NULLABLE | Template/Chế độ học đang áp dụng cho Topic |

#### Bảng `template_elements`

Lưu các phần tử thành phần của một template theo thứ tự hiển thị (Layout Frappe style).

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh phần tử |
| `template_id` | `bigint(20)` | FK -> `templates(id)`, NOT NULL | Template chứa phần tử |
| `position` | `int(11)` | NOT NULL | Thứ tự vị trí xuất hiện (0, 1, 2...) |
| `type` | `varchar(50)` | NOT NULL | Kiểu phần tử: `FIELD`, `SECTION_BREAK`, `COLUMN_BREAK` |

> Ràng buộc duy nhất: `uk_template_element_position (template_id, position)`.

#### Bảng `template_fields`

Lưu chi tiết cấu hình hiển thị cho các phần tử kiểu `FIELD`.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh cấu hình field |
| `element_id` | `bigint(20)` | FK -> `template_elements(id)`, NOT NULL, UNIQUE | Phần tử tương ứng (quan hệ 1-1) |
| `schema_attribute_id` | `bigint(20)` | FK -> `schema_attributes(id)`, NOT NULL | Thuộc tính dữ liệu được hiển thị (cùng Schema) |
| `semantic_role` | `varchar(50)` | NULLABLE | Vai trò ngữ nghĩa (`TARGET_WORD`, `DEFINITION`, `AUDIO`, v.v.) |
| `field_label` | `varchar(255)` | NULLABLE | Nhãn tuỳ chỉnh hiển thị trước giá trị |
| `hide_if_empty` | `bit(1)` | NOT NULL, DEFAULT 0 | Ẩn trường nếu giá trị rỗng |
| `audio_action` | `bit(1)` | NOT NULL, DEFAULT 0 | Kích hoạt chức năng phát âm khi nhấn vào trường |
| `font_size` | `int(11)` | NULLABLE | Kích cỡ chữ tương đối (pixel hoặc đơn vị giao diện) |
| `alignment` | `varchar(50)` | NULLABLE | Căn chỉnh văn bản: `LEFT`, `CENTER`, `RIGHT` |
| `color` | `varchar(50)` | NULLABLE | Mã màu văn bản (Hex code hoặc tên màu chuẩn) |

#### Bảng `fsrs_records`

Lưu trữ trạng thái ôn tập FSRS của từng người dùng đối với từng mục trong chủ đề.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh bản ghi ôn tập |
| `user_id` | `bigint(20)` | FK -> `users(id)`, NOT NULL | Người học |
| `topic_item_id` | `bigint(20)` | FK -> `topic_items(id)`, NOT NULL | Mục từ vựng đang học |
| `card_state` | `varchar(50)` | NOT NULL | Trạng thái: `NEW`, `LEARNING`, `REVIEW`, `RELEARNING`, `SUSPENDED` |
| `due` | `datetime(6)` | NOT NULL | Thời điểm đến hạn ôn tiếp theo |
| `stability` | `double` | NOT NULL | Độ bền trí nhớ (S) |
| `difficulty` | `double` | NOT NULL | Độ khó của thẻ (D) |
| `reps` | `int(11)` | NOT NULL | Số lượt ôn tập thành công |
| `lapses` | `int(11)` | NOT NULL | Số lần quên thẻ |
| `last_review` | `datetime(6)` | NULLABLE | Thời điểm ôn tập gần nhất |
| `created_at` | `datetime(6)` | NOT NULL | Thời điểm tạo bản ghi |
| `updated_at` | `datetime(6)` | NOT NULL | Thời điểm cập nhật |

> Ràng buộc duy nhất: `uk_user_topic_item (user_id, topic_item_id)`.

### 4.2. Sơ đồ quan hệ thực thể

```mermaid
erDiagram
    users ||--o{ collections : "owns (type=USER)"
    collections ||--o{ topics : contains
    topics ||--o{ topics : "parent-child"
    topics ||--|| topic_schemas : "has (1-1)"
    schemas ||--o{ topic_schemas : "applies to (1-N)"
    schemas ||--o{ schema_attribute_groups : contains
    schema_attribute_groups ||--o{ schema_attributes : contains
    schemas ||--o{ templates : "defines views (1-N)"
    topics }o--o| templates : "active learning mode (active_template_id)"
    templates ||--o{ template_elements : contains
    template_elements ||--o| template_fields : "specifies (type=FIELD)"
    schema_attributes ||--o{ template_fields : "mapped to"
    topics ||--o{ topic_items : contains
    topic_items ||--o{ topic_item_attribute_groups : has
    topic_item_attribute_groups ||--o{ topic_item_attribute_values : contains
    schema_attributes ||--o{ topic_item_attribute_values : "defines schema for"
    users ||--o{ fsrs_records : reviews
    topic_items ||--o{ fsrs_records : "tracked by"

    schemas {
        bigint id PK
        varchar code UK
        varchar name
        text description
        bit is_system
        datetime created_at
        datetime updated_at
    }

    topics {
        bigint id PK
        bigint collection_id FK
        bigint parent_id FK
        bigint active_template_id FK
        varchar name
        datetime created_at
        datetime updated_at
    }

    topic_schemas {
        bigint id PK
        bigint topic_id FK_UK
        bigint schema_id FK
        datetime created_at
        datetime updated_at
    }

    schema_attribute_groups {
        bigint id PK
        bigint schema_id FK "FK trỏ sang schemas"
        varchar name
        varchar label
        bit multiple
        smallint position
        datetime created_at
        datetime updated_at
    }

    schema_attributes {
        bigint id PK
        bigint group_id FK
        varchar name
        varchar label
        varchar data_type
        bit required
        smallint position
        datetime created_at
        datetime updated_at
    }

    templates {
        bigint id PK
        bigint schema_id FK
        varchar code
        varchar name
        bit is_default
        datetime created_at
        datetime updated_at
    }

    template_elements {
        bigint id PK
        bigint template_id FK
        int position
        varchar type "FIELD, SECTION_BREAK, COLUMN_BREAK"
    }

    template_fields {
        bigint id PK
        bigint element_id FK_UK
        bigint schema_attribute_id FK
        varchar semantic_role "TARGET_WORD, DEFINITION, AUDIO..."
        varchar field_label
        bit hide_if_empty
        bit audio_action
        int font_size
        varchar alignment "LEFT, CENTER, RIGHT"
        varchar color
    }

    fsrs_records {
        bigint id PK
        bigint user_id FK
        bigint topic_item_id FK
        varchar card_state "NEW, LEARNING, REVIEW..."
        datetime due
        double stability
        double difficulty
        int reps
        int lapses
        datetime last_review
        datetime created_at
        datetime updated_at
    }
```

#### Vai trò các bảng trong mô hình Template & Học tập

| Bảng | Vai trò | Ghi chú |
| :--- | :--- | :--- |
| `topics` | Đơn vị tổ chức kiến thức | Liên kết 1-1 với `topic_schemas` và trỏ tới `active_template_id` của Schema. |
| `schemas` | Lược đồ thuộc tính độc lập | Quản lý tập hợp các nhóm, thuộc tính động và danh mục templates hiển thị. |
| `topic_schemas` | Cầu nối Topic - Schema | Entity trung gian gán Topic với Schema tương ứng (1-1 với Topic, N-1 với Schema). |
| `schema_attribute_groups` | Nhóm thuộc tính schema | Gom nhóm các thuộc tính liên quan (ví dụ main, examples), thuộc `schemas`. |
| `schema_attributes` | Định nghĩa thuộc tính | Tên trường, nhãn, kiểu dữ liệu, thứ tự hiển thị cơ bản. |
| `templates` | Mẫu hiển thị thẻ của Schema | Thuộc về `schemas`, đại diện cho các mode học (Standard, Listening, Reverse...). |
| `template_elements` | Khối phần tử trên thẻ | Lưu thứ tự `position` và kiểu (`FIELD`, `SECTION_BREAK`, `COLUMN_BREAK`). |
| `template_fields` | Thiết lập trường hiển thị | Map phần tử với `schema_attribute_id` (cùng Schema), gán `semantic_role` và styling. |
| `topic_items` | Mục từ vựng thực tế | Từng mục kiến thức trong chủ đề, mang các giá trị thuộc tính tương ứng. |
| `fsrs_records` | Trạng thái ghi nhớ FSRS | Theo dõi độ ổn định (stability), độ khó (difficulty) và lịch ôn tập `due` cho từng `(user_id, topic_item_id)`. |

### 4.3. Chiến lược Tiến hóa Lược đồ: Copy-on-Write (Fork) & Additive Evolution

Nhằm đảm bảo tính toàn vẹn dữ liệu khi nhiều Topic cùng chia sẻ một Schema:

1. **Template làm lớp hiển thị đa chế độ (Multi-mode Views)**: 
   - `Template` chính là View Layer, còn `Schema` là Data Contract.
   - Một Schema có sẵn nhiều Template ứng với các chế độ học (Standard, Listening, Reverse). Topic chỉ cần chuyển đổi `active_template_id` là có thể đổi cách học ngay tức thì mà không cần đụng chạm dữ liệu từ vựng.
2. **Tiến hóa mở rộng (Additive Evolution)**:
   - Trên Schema dùng chung, chỉ cho phép **thêm mới** thuộc tính (các thuộc tính mới mặc định là tùy chọn).
   - Hệ thống ngăn chặn việc xóa hoặc đổi kiểu dữ liệu của các thuộc tính đang có dữ liệu (`topic_item_attribute_values`) hoặc đang được tham chiếu bởi `template_fields`.
3. **Copy-on-Write (Fork Schema & Templates)**:
   - Khi người dùng muốn tùy biến sâu cấu trúc thuộc tính hoặc sửa đổi các template cho riêng Topic của mình:
   - Gọi API `POST /api/topics/{topicId}/schema/fork`.
   - Hệ thống thực thi trong một `@Transactional` duy nhất:
     1. Nhân bản (clone) Schema hiện tại thành một Schema độc lập mới (`code = null`, `is_system = false`).
     2. Nhân bản toàn bộ nhóm thuộc tính (`SchemaAttributeGroup`) và thuộc tính con (`SchemaAttribute`), xây dựng bảng ánh xạ ID cũ -> ID mới.
     3. Nhân bản toàn bộ danh sách `Template` của Schema cũ sang Schema mới.
     4. Nhân bản toàn bộ `TemplateElement` và `TemplateField` của từng template, re-map `schema_attribute_id` sang thuộc tính mới tương ứng.
     5. Cập nhật `topic_schemas.schema_id` sang Schema mới.
     6. Cập nhật `topic.active_template_id` sang Template clone tương ứng.
     7. Tự động re-map toàn bộ khóa ngoại trong `topic_item_attribute_values` và `topic_item_attribute_groups` của Topic đó sang các thuộc tính mới vừa clone.
     8. Sau khi fork, Topic sở hữu Schema và bộ Template hoàn toàn riêng biệt, tự do tùy biến mà không ảnh hưởng tới bất kỳ Topic nào khác.

---

## 5. Enumeration trong Mã nguồn Backend

Các enum thuộc package `vn.ptit.snapvocab.domain.enumeration`:

### `SemanticRole`

Định nghĩa vai trò ngữ nghĩa của từng trường dữ liệu khi hiển thị trên thẻ flashcard:

```java
public enum SemanticRole {
    TARGET_WORD,        // Từ vựng mục tiêu / câu hỏi chính
    EXAMPLE_SENTENCE,   // Câu ví dụ minh họa hoặc ngữ cảnh
    NATIVE_TRANSLATION, // Bản dịch tiếng mẹ đẻ (tiếng Việt) bổ trợ
    DEFINITION,         // Định nghĩa / giải nghĩa từ vựng
    AUDIO,              // Dữ liệu âm thanh phát âm
    IMAGE               // Hình ảnh minh họa trực quan
}
```

### `TemplateElementType`

Phân loại phần tử bố cục trong template (chuẩn Frappe layout):

```java
public enum TemplateElementType {
    FIELD,          // Trường dữ liệu hiển thị (liên kết 1-1 với TemplateField)
    COLUMN_BREAK,   // Ngắt cột bố cục (chia layout nhiều cột)
    SECTION_BREAK   // Phân tách khối thẻ (ngăn cách Mặt trước / Mặt sau)
}
```

### `Alignment`

Căn lề văn bản của trường hiển thị:

```java
public enum Alignment {
    LEFT,
    CENTER,
    RIGHT,
    JUSTIFY
}
```

### `CardState`

Trạng thái học tập của thẻ theo thuật toán FSRS:

```java
public enum CardState {
    NEW,            // Thẻ mới chưa học
    LEARNING,       // Đang học lần đầu
    REVIEW,         // Đang trong chu kỳ ôn tập định kỳ
    RELEARNING,     // Bị quên, đang học lại
    SUSPENDED       // Tạm dừng học
}
```

---

## 6. Quy tắc Nghiệp vụ (Business Rules)

### 6.1. Quản lý Schema & Template (Unified Studio)

1. **Gắn kết Template vào Schema**: Mỗi `Template` thuộc về một `Schema` (`schemas (1) --- (N) templates`). Một Schema có thể có nhiều Template tương ứng với các chế độ học khác nhau (`STANDARD`, `LISTENING`, `REVERSE`).
2. **Định danh Hệ thống bằng `code`**:
   - Schema mẫu hệ thống có `code` bất biến (ví dụ: `DEFAULT_ENGLISH`) và `is_system = true`.
   - Template mẫu hệ thống có `code` bất biến (ví dụ: `STANDARD`, `LISTENING`, `REVERSE`) và `is_default` xác định template mặc định ban đầu.
   - Tuyệt đối không hardcode ID số tự tăng trong code logic hay seed data.
3. **Toàn vẹn khóa ngoại của Field**: `schema_attribute_id` trong `template_fields` bắt buộc phải thuộc về chính `Schema` sở hữu Template đó (thông qua `schema_attribute_groups`). Điều này ngăn chặn 100% lỗi template trỏ nhầm sang thuộc tính của schema khác.
4. **Tính toàn vẹn của thứ tự (`position`)**: Trường `position` trong `template_elements` phải là số nguyên không âm và là duy nhất trong phạm vi một template (ràng buộc `uk_template_element_position`). Khi client hiển thị, các phần tử được sắp xếp theo thứ tự `position ASC`.
5. **Quan hệ 1-1 giữa Element và Field**: Mỗi phần tử có kiểu `type = FIELD` bắt buộc phải có đúng một bản ghi `template_fields` tương ứng; các kiểu `SECTION_BREAK` hoặc `COLUMN_BREAK` không chứa `template_fields`.
6. **Xóa tầng (Cascade delete)**: Khi xóa một `Schema`, hệ thống cascade xóa toàn bộ `Template`, `TemplateElement` và `TemplateField` liên quan.

### 6.2. Hiển thị & Đa chế độ học (Multi-mode Learning)

1. **Chọn Chế độ học cho Topic**: Mỗi `Topic` trỏ tới `active_template_id` của Schema tương ứng.
   - Tạo Topic mới: Mặc định gán Schema hệ thống (`DEFAULT_ENGLISH`) và Template mặc định (`is_default = true`, tức `STANDARD`).
   - Đổi chế độ học: Người học có thể đổi chế độ ôn tập (Standard -> Listening -> Reverse) ngay trên Topic settings hoặc trước phiên học. Hệ thống chỉ cập nhật `topic.active_template_id`.
   - Không nhân bản dữ liệu: Dữ liệu từ vựng EAV (`topic_items`) và tiến trình FSRS (`fsrs_records`) được bảo toàn trọn vẹn.
2. **Phân định hai mặt thẻ dựa theo `SECTION_BREAK` & `SemanticRole`**:
   - Mặt trước (Front): Các phần tử xuất hiện trước `SECTION_BREAK` đầu tiên (ví dụ `TARGET_WORD`, `AUDIO`).
   - Mặt sau (Back): Các phần tử xuất hiện sau `SECTION_BREAK` (ví dụ `DEFINITION`, `NATIVE_TRANSLATION`, `EXAMPLE_SENTENCE`, `IMAGE`).
3. **Ẩn trường trống (`hide_if_empty = true`)**: Nếu một mục từ không có dữ liệu cho thuộc tính tương ứng, ứng dụng di động sẽ tự động bỏ qua khối hiển thị đó mà không để lại khoảng trống bất thường.
4. **Hành vi âm thanh (`audio_action = true`)**: Khi người dùng nhấn vào trường có cờ này hoặc trường có vai trò `AUDIO`, ứng dụng sẽ kích hoạt phát file âm thanh phát âm.

### 6.3. Quản lý Ôn tập FSRS

1. **Theo dõi tiến trình trực tiếp**: Trạng thái ôn tập của từng người học được lưu tại bảng `fsrs_records` cho từng cặp `(user_id, topic_item_id)`.
2. **Độc lập giao diện**: Thay đổi chế độ học (`active_template_id`) hoặc sửa đổi layout template chỉ làm thay đổi cách hiển thị thẻ, hoàn toàn không làm gián đoạn hoặc sai lệch các tham số FSRS (`stability`, `difficulty`, `due`, `reps`, `lapses`).

---

## 7. Thiết kế API Endpoints

### 7.1. Quản lý Topic & Template

| Phương thức | Đường dẫn | Mô tả | Quyền truy cập |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/v1/collections/{collectionId}/topics` | Tạo Topic mới (tự động gán Schema & Template mặc định nếu không truyền) | Bearer JWT |
| `PUT` | `/api/v1/topics/{topicId}/active-template/{templateId}` | Thay đổi chế độ học (active template) cho Topic | Bearer JWT |
| `POST` | `/api/v1/topics/{topicId}/schema/fork` | Fork Schema và toàn bộ Templates riêng cho Topic | Bearer JWT |
| `GET` | `/api/v1/schemas/{schemaId}/templates` | Lấy danh sách các templates của một Schema | Bearer JWT |
| `GET` | `/api/v1/templates/{id}` | Lấy chi tiết một Template kèm các Elements và Fields | Bearer JWT |
| `PUT` | `/api/v1/templates/{id}` | Cập nhật layout cho một Template | Bearer JWT |

#### Response mẫu: Cấu hình Template của Topic

```json
{
  "statusCode": 200,
  "data": {
    "id": 1,
    "schemaId": 1,
    "code": "STANDARD",
    "name": "Standard Flashcard",
    "isDefault": true,
    "elements": [
      {
        "id": 101,
        "position": 0,
        "type": "FIELD",
        "field": {
          "id": 201,
          "schemaAttributeId": 1,
          "semanticRole": "TARGET_WORD",
          "fieldLabel": "Từ vựng",
          "hideIfEmpty": false,
          "audioAction": false,
          "fontSize": 24,
          "alignment": "CENTER",
          "color": "#1F2937"
        }
      },
      {
        "id": 102,
        "position": 1,
        "type": "FIELD",
        "field": {
          "id": 202,
          "schemaAttributeId": 4,
          "semanticRole": "AUDIO",
          "fieldLabel": "Phát âm",
          "hideIfEmpty": true,
          "audioAction": true,
          "fontSize": 16,
          "alignment": "CENTER",
          "color": "#6B7280"
        }
      },
      {
        "id": 103,
        "position": 2,
        "type": "SECTION_BREAK",
        "field": null
      },
      {
        "id": 104,
        "position": 3,
        "type": "FIELD",
        "field": {
          "id": 203,
          "schemaAttributeId": 3,
          "semanticRole": "DEFINITION",
          "fieldLabel": "Định nghĩa",
          "hideIfEmpty": false,
          "audioAction": false,
          "fontSize": 20,
          "alignment": "LEFT",
          "color": "#111827"
        }
      },
      {
        "id": 105,
        "position": 4,
        "type": "FIELD",
        "field": {
          "id": 204,
          "schemaAttributeId": 5,
          "semanticRole": "EXAMPLE_SENTENCE",
          "fieldLabel": "Ví dụ",
          "hideIfEmpty": true,
          "audioAction": false,
          "fontSize": 15,
          "alignment": "LEFT",
          "color": "#4B5563"
        }
      }
    ],
    "createdAt": "2026-09-01T08:00:00Z",
    "updatedAt": "2026-09-01T08:00:00Z"
  }
}
```

### 7.2. Phiên Học Flashcard (Study Session)

| Phương thức | Đường dẫn | Mô tả | Quyền truy cập |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/v1/topics/{topicId}/study-session` | Tải phiên ôn tập: template đang active và các mục đến hạn | Bearer JWT |
| `POST` | `/api/v1/fsrs-records/{id}/review` | Gửi kết quả đánh giá thẻ (Again, Hard, Good, Easy) | Bearer JWT |

#### Response mẫu: Dữ liệu phiên học Flashcard

```json
{
  "statusCode": 200,
  "data": {
    "topicId": 45,
    "activeTemplate": {
      "id": 1,
      "code": "STANDARD",
      "name": "Standard Flashcard",
      "elements": [
        {
          "position": 0,
          "type": "FIELD",
          "semanticRole": "TARGET_WORD",
          "attributeId": 1,
          "fontSize": 24,
          "alignment": "CENTER"
        },
        {
          "position": 1,
          "type": "SECTION_BREAK"
        },
        {
          "position": 2,
          "type": "FIELD",
          "semanticRole": "DEFINITION",
          "attributeId": 3,
          "fontSize": 20,
          "alignment": "LEFT"
        }
      ]
    },
    "items": [
      {
        "topicItemId": 801,
        "fsrsRecord": {
          "id": 1501,
          "cardState": "REVIEW",
          "due": "2026-09-11T09:00:00Z",
          "stability": 4.5,
          "difficulty": 5.2,
          "reps": 3,
          "lapses": 0
        },
        "values": [
          { "attributeId": 1, "attributeName": "word", "value": "perseverance" },
          { "attributeId": 2, "attributeName": "phonetic", "value": "/ˌpɜːrsəˈvɪərəns/" },
          { "attributeId": 3, "attributeName": "meaning", "value": "sự kiên trì, bền bỉ" },
          { "attributeId": 5, "attributeName": "example", "value": "Success requires perseverance." }
        ]
      }
    ]
  }
}
```

---

## 8. Dữ liệu Mẫu (Seed Data)

Dưới đây là kịch bản SQL mẫu khởi tạo Schema hệ thống và 3 Template tương ứng:

```sql
-- 1. Khởi tạo Schema hệ thống DEFAULT_ENGLISH
INSERT INTO schemas (id, code, name, description, is_system, created_at, updated_at)
VALUES (1, 'DEFAULT_ENGLISH', 'Tiếng Anh Chuẩn', 'Schema từ vựng tiếng Anh mặc định', 1, NOW(), NOW());

-- 2. Khởi tạo nhóm thuộc tính và các thuộc tính
INSERT INTO schema_attribute_groups (id, schema_id, name, label, multiple, position, created_at, updated_at)
VALUES (1, 1, 'main', 'Thông tin từ vựng', 0, 0, NOW(), NOW());

INSERT INTO schema_attributes (id, group_id, name, label, data_type, required, position, created_at, updated_at) VALUES
(1, 1, 'word', 'Từ vựng', 'TEXT', 1, 0, NOW(), NOW()),
(2, 1, 'phonetic', 'Phiên âm', 'TEXT', 0, 1, NOW(), NOW()),
(3, 1, 'meaning', 'Định nghĩa', 'TEXT', 1, 2, NOW(), NOW()),
(4, 1, 'audio', 'Phát âm', 'AUDIO', 0, 3, NOW(), NOW()),
(5, 1, 'example', 'Ví dụ', 'TEXT', 0, 4, NOW(), NOW()),
(6, 1, 'image', 'Hình ảnh', 'IMAGE', 0, 5, NOW(), NOW());

-- 3. Khởi tạo 3 Template cho Schema (Standard, Listening, Reverse)
INSERT INTO templates (id, schema_id, code, name, is_default, created_at, updated_at) VALUES
(1, 1, 'STANDARD', 'Thẻ Chuẩn', 1, NOW(), NOW()),
(2, 1, 'LISTENING', 'Luyện Nghe', 0, NOW(), NOW()),
(3, 1, 'REVERSE', 'Đảo Chiều (Đoán Từ)', 0, NOW(), NOW());

-- 4. Cấu hình Elements & Fields cho STANDARD Template
INSERT INTO template_elements (id, template_id, position, type) VALUES
(1, 1, 0, 'FIELD'),
(2, 1, 1, 'SECTION_BREAK'),
(3, 1, 2, 'FIELD'),
(4, 1, 3, 'FIELD');

INSERT INTO template_fields (element_id, schema_attribute_id, semantic_role, field_label, hide_if_empty, audio_action, font_size, alignment, color) VALUES
(1, 1, 'TARGET_WORD', 'Từ vựng', 0, 0, 24, 'CENTER', '#111827'),
(3, 3, 'DEFINITION', 'Giải nghĩa', 0, 0, 20, 'LEFT', '#1F2937'),
(4, 5, 'EXAMPLE_SENTENCE', 'Ví dụ', 1, 0, 15, 'LEFT', '#4B5563');
```

---

## 9. Chuyển dịch Kiến trúc & Tương thích

Hệ thống đã hoàn tất tái cấu trúc, hoàn thiện mô hình:

1. **Từ vựng & Thư mục**: `Collection` (phân loại `SYSTEM` hoặc `USER`) và `Topic` (hỗ trợ phân cấp cây cha - con).
2. **Lược đồ & Hiển thị Thẻ**: 
   - `Schema` độc lập đóng vai trò Data Contract.
   - `templates` gắn với `Schema` đóng vai trò View Layer (hỗ trợ nhiều chế độ học cho cùng một bộ từ vựng).
   - `Topic` liên kết với `Schema` qua `topic_schemas` và trỏ tới `active_template_id`.
3. **Nội dung thẻ**: `TopicItem` kết hợp thuộc tính động EAV (`schema_attributes`, `topic_item_attribute_values`).
4. **Theo dõi ôn tập**: `fsrs_records` kết nối trực tiếp `users` và `topic_items`, hoàn toàn độc lập với việc thay đổi chế độ học.

---

## 10. Tương tác với các Phân hệ Khác

| Phân hệ | Mối liên hệ và tương tác |
| :--- | :--- |
| **Thuật toán SRS FSRS** | FSRS tính toán lịch ôn tập và lưu trữ trực tiếp trên bảng `fsrs_records`. Template chỉ quyết định lớp hiển thị của thẻ, không làm thay đổi các biến số tính toán của thuật toán. |
| **Quét từ vựng (Scan-to-Vocabulary)** | Dữ liệu từ vựng nhận diện qua OCR/LLM sau khi xác nhận sẽ được lưu thành `TopicItem` thuộc một `Topic` đã chọn. Mục từ này ngay lập tức hiển thị theo template đang active của Topic đó. |
| **Gamification & Thống kê** | Mỗi lượt gửi kết quả đánh giá FSRS thành công được tính vào chỉ số hoàn thành mục tiêu học tập hàng ngày và tích lũy điểm kinh nghiệm (XP) cho người học. |

---

## 11. Phụ lục: Lịch sử Quyết định Thiết kế

Trong quá trình xây dựng hệ thống flashcard, bài toán quản lý giao diện thẻ học đã được nâng cấp qua các giai đoạn:

1. **Giai đoạn 1 (Cố định cứng)**: Lưu loại thẻ cố định trên từng bản ghi Note/Card. Bị loại bỏ vì không đáp ứng được yêu cầu thuộc tính động EAV.
2. **Giai đoạn 2 (Topic Template riêng lẻ)**: Mỗi Topic sở hữu Template riêng (`templates.topic_id`). Bị nâng cấp vì dẫn đến việc trùng lặp template trên hàng ngàn topic, khó hỗ trợ Đa chế độ học (Multi-mode Learning) và tách rời cấu hình dữ liệu/giao diện.
3. **Giai đoạn 3 (Schema-Template Unified Studio - Frappe style)**:
   - `Template` thuộc về `Schema` (`schemas (1) --- (N) templates`).
   - Một Schema có sẵn nhiều Template (Standard, Listening, Reverse).
   - Topic chỉ cần chọn `active_template_id` để chuyển chế độ học.
   - Khi cần tùy biến sâu cho Topic, hệ thống tự động Fork cả Schema và Templates sang bản sao riêng biệt trong 1 transaction.
   - Định danh hệ thống bằng chuỗi `code` bất biến thay vì ID số tự tăng.

**Lợi ích vượt trội của thiết kế hiện tại:**
- **Đa chế độ học không nhân bản dữ liệu**: Học nghe, học chữ hay đảo chiều đều dùng chung 1 bộ từ vựng EAV và 1 tiến trình FSRS duy nhất.
- **Toàn vẹn quan hệ 100%**: TemplateField chỉ trỏ vào thuộc tính của chính Schema đó, loại bỏ hoàn toàn lỗi orphan references.
- **Trải nghiệm thiết kế Frappe Studio thống nhất**: Tạo/sửa trường dữ liệu và thiết kế thẻ học cùng một nơi.
- **Tối ưu CSDL**: Hàng ngàn Topic chia sẻ chung Schema và Template hệ thống, giảm thiểu dữ liệu trùng lặp tối đa.
