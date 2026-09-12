# Đặc tả Kiến trúc: Topic Template & Thẻ Flashcard Tùy chỉnh (Custom Card)

Tài liệu này mô tả chi tiết kiến trúc **Topic Template** — giải pháp cho phép hệ thống và người học tùy biến cách thức hiển thị thẻ flashcard khi ôn tập các mục từ vựng (`TopicItem`) trong từng chủ đề (`Topic`), dựa trên cấu trúc thuộc tính động kết hợp các vai trò ngữ nghĩa (`SemanticRole`).

---

## 1. Tổng quan

### 1.1. Vấn đề cần giải quyết

Hệ thống flashcard dạng truyền thống thường áp dụng cấu trúc cứng nhắc: mặt trước luôn là từ vựng, mặt sau luôn là định nghĩa cố định. Cách tiếp cận này bộc lộ những hạn chế lớn:

1. **Không thích ứng với thuộc tính động**: Mỗi chủ đề học tập (`Topic`) có thể có bộ thuộc tính riêng (phiên âm, giải nghĩa tiếng Việt, câu ví dụ, ngữ cảnh, hình ảnh minh họa, file phát âm...). Cấu trúc cứng không thể phản ánh đầy đủ mô hình dữ liệu EAV của hệ thống.
2. **Thiếu linh hoạt trong trải nghiệm học**: Người học hoặc chủ đề khác nhau đòi hỏi các kiểu hiển thị khác nhau (học nhận diện mặt chữ, học nghe - phát hiện từ, học đoán nghĩa qua câu ví dụ, học qua hình ảnh).
3. **Phụ thuộc triển khai client**: Nếu không có cơ chế template động từ backend, mỗi khi thay đổi cách bố trí hiển thị lại đòi hỏi cập nhật code ứng dụng di động.

### 1.2. Giải pháp: Topic Template System

SnapVocab áp dụng mô hình template gắn trực tiếp với từng chủ đề (`Topic`):

- **Template theo chủ đề**: Mỗi `Topic` sở hữu một `Template` quy định cách hiển thị flashcard cho toàn bộ các `TopicItem` thuộc chủ đề đó.
- **Phân rã thành phần tử (`TemplateElement`)**: Mỗi template chứa danh sách các phần tử hiển thị theo thứ tự vị trí (`position`), phân loại theo kiểu phần tử (`FIELD`, `DIVIDER`, `BUTTON`).
- **Ánh xạ thuộc tính & vai trò ngữ nghĩa (`TemplateField`)**: Với phần tử kiểu `FIELD`, cấu hình liên kết trực tiếp tới một thuộc tính `SchemaAttribute`, đồng thời gán vai trò ngữ nghĩa `SemanticRole` (`FRONT`, `BACK`, `EXAMPLE`, `AUDIO`, `IMAGE`, `PHONETIC`, `TRANSLATION`, `HINT`, `TAG`, `EXTRA`) kèm định dạng hiển thị (`font_size`, `alignment`, `color`, `audio_action`, `hide_if_empty`).
- **Tối giản cho Learner**: Không yêu cầu viết mã HTML/CSS. Ứng dụng di động dựa vào `semantic_role` và thứ tự `position` để render thẻ trực quan, mượt mà trên màn hình cảm ứng.

| Khía cạnh | Mô hình cũ (Cố định) | Mô hình Topic Template hiện tại |
| :--- | :--- | :--- |
| Phạm vi áp dụng | Toàn bộ thẻ chung một khuôn | Từng `Topic` có template riêng |
| Nguồn dữ liệu | Cột cố định trong bảng note | Thuộc tính động từ `SchemaAttribute` và `TopicItemAttributeValue` |
| Bố cục hiển thị | Cố định 2 mặt trước / sau | Sắp xếp theo `position` với các vai trò ngữ nghĩa `SemanticRole` |
| Quản lý tiến độ | Bảng Card riêng lẻ | `FsrsRecord` gắn với cặp `(user_id, topic_item_id)` |
| Độ phức tạp | Cứng nhắc, khó mở rộng | Động, mở rộng linh hoạt theo dữ liệu EAV |

### 1.3. Vị trí trong hệ thống

Chức năng này thuộc phân hệ **Learning Engine**, cung cấp cấu hình hiển thị cho module Flashcard và thuật toán lặp lại ngắt quãng FSRS (`FsrsRecord`).

---

## 2. Khái niệm cốt lõi

### 2.1. Cấu trúc Lược đồ Thuộc tính của Topic (`TopicSchema` & `SchemaAttribute`)

Mỗi chủ đề (`Topic`) sở hữu 1 Schema (`TopicSchema`) tổ chức dữ liệu theo mô hình động:
- Một `Topic` liên kết 1-1 với một `TopicSchema`.
- Mỗi `TopicSchema` có các nhóm thuộc tính (`SchemaAttributeGroup`).
- Mỗi nhóm chứa các thuộc tính (`SchemaAttribute`) xác định tên thuộc tính, nhãn hiển thị (`label`), kiểu dữ liệu (`dataType`), thứ tự (`position`) và trạng thái bắt buộc (`required`).
- Các mục từ trong chủ đề (`TopicItem`) lưu giá trị thực tế tương ứng trong bảng `topic_item_attribute_values`.

### 2.2. Vai trò Ngữ nghĩa (`SemanticRole`)

Để ứng dụng di động hiểu được ý nghĩa hiển thị mà không cần hardcode tên trường, mỗi `TemplateField` được gán một `SemanticRole`:

| SemanticRole | Ý nghĩa | Ứng dụng hiển thị trên thẻ |
| :--- | :--- | :--- |
| `FRONT` | Nội dung câu hỏi chính ở mặt trước | Từ vựng, cụm từ, thuật ngữ chính cần ghi nhớ |
| `BACK` | Đáp án chính ở mặt sau | Giải nghĩa, định nghĩa từ vựng |
| `EXAMPLE` | Câu ví dụ hoặc ngữ cảnh sử dụng | Câu ví dụ minh họa kèm bản dịch (nếu có) |
| `AUDIO` | Dữ liệu âm thanh / phát âm | Tích hợp nút nghe hoặc tự động phát âm |
| `IMAGE` | Hình ảnh minh họa | Render hình ảnh ở vị trí nổi bật của thẻ |
| `PHONETIC` | Ký âm ngữ âm | Hiển thị phiên âm quốc tế (IPA) |
| `TRANSLATION` | Bản dịch nghĩa tiếng mẹ đẻ | Hiển thị nghĩa tiếng Việt bổ trợ |
| `HINT` | Gợi ý khi người học gặp khó khăn | Hiển thị dạng ẩn, mở khi người học bấm nút gợi ý |
| `TAG` | Thẻ phân loại hoặc cấp độ | Cấp độ CEFR, nhãn ngữ pháp (noun, verb...) |
| `EXTRA` | Thông tin bổ sung | Ghi chú cá nhân, từ đồng nghĩa, trái nghĩa |

### 2.3. Bố cục Template (`Template` & `TemplateElement`)

Một `Template` liên kết với `Topic` qua trường `topic_id`. Template bao gồm:
- **`TemplateElement`**: Đại diện cho một khối phần tử trên giao diện flashcard.
  - `position`: Thứ tự hiển thị tăng dần từ trên xuống dưới.
  - `type`: Phân loại phần tử gồm `FIELD` (trường dữ liệu), `DIVIDER` (đường phân tách giữa các phần), hoặc `BUTTON` (nút tương tác như nút nghe, nút lật thẻ).
- **`TemplateField`**: Cấu hình chi tiết cho phần tử kiểu `FIELD`.
  - `topic_attribute_id`: Khóa ngoại tham chiếu đến thuộc tính cần lấy dữ liệu.
  - `semantic_role`: Vai trò ngữ nghĩa nêu trên.
  - `field_label`: Nhãn tuỳ chỉnh hiển thị trước giá trị (nếu có).
  - `hide_if_empty`: Nếu giá trị của thuộc tính rỗng thì ẩn hoàn toàn phần tử khỏi thẻ.
  - `audio_action`: Kích hoạt tương tác phát âm thanh khi nhấn vào trường này.
  - `font_size`, `alignment` (`LEFT`, `CENTER`, `RIGHT`), `color`: Các thuộc tính định dạng giao diện.

---

## 3. Cơ chế hoạt động

### 3.1. Luồng cấu hình Template cho Topic

```
Quản trị viên / Người dùng tạo Topic
                 │
                 ▼
Khai báo SchemaAttributeGroup & SchemaAttribute
(Định nghĩa schema thuộc tính: từ, ipa, nghĩa, ví dụ, audio)
                 │
                 ▼
Khởi tạo Template cho Topic
(Hệ thống tự động sinh template mặc định hoặc người dùng tùy chỉnh)
                 │
                 ▼
Tạo danh sách TemplateElement & TemplateField
(Gán position, kiểu phần tử, ánh xạ attribute và semantic_role)
                 │
                 ▼
Nhập dữ liệu các TopicItem
(Giá trị thuộc tính được lưu vào topic_item_attribute_values)
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
Backend tải cấu hình Template của Topic
(Kèm danh sách TemplateElement và TemplateField theo position ASC)
                 │
                 ▼
Backend gộp giá trị thuộc tính của TopicItem vào Response
                 │
                 ▼
Mobile Client render Flashcard:
├─ Mặt trước (Front): Các element có semantic_role = FRONT, PHONETIC, AUDIO...
├─ Đường phân cách / nút lật (DIVIDER / BUTTON)
└─ Mặt sau (Back): Các element có semantic_role = BACK, TRANSLATION, EXAMPLE...
                 │
                 ▼
Learner đánh giá độ nhớ (Again, Hard, Good, Easy)
                 │
                 ▼
Backend cập nhật FsrsRecord (stability, difficulty, due, reps, lapses)
```

### 3.3. Xử lý dữ liệu khuyết thiếu & Thay đổi thuộc tính

1. **Ẩn trường trống (`hide_if_empty = true`)**: Khi một `TopicItem` không có giá trị cho một thuộc tính tùy chọn (ví dụ không có ví dụ hay hình ảnh), ứng dụng tự động bỏ qua khối element đó, giao diện thẻ tự động co giãn tự nhiên.
2. **Thay đổi cấu hình Template**: Khi cập nhật Template của Topic (thay đổi thứ tự `position`, đổi vai trò `semantic_role` hoặc màu sắc, kích cỡ chữ), toàn bộ các `TopicItem` thuộc Topic lập tức được áp dụng giao diện mới trong phiên học tiếp theo mà không cần cập nhật dữ liệu từng item.
3. **Tiến độ FSRS độc lập**: Trạng thái học tập của từng từ (`fsrs_records`) hoàn toàn độc lập với việc thay đổi giao diện thẻ, bảo đảm dữ liệu ghi nhớ không bị ảnh hưởng khi tinh chỉnh layout.

---

## 4. Mô hình Dữ liệu

### 4.1. Chi tiết các bảng liên quan

#### Bảng `templates`

Lưu cấu hình template của chủ đề.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh template |
| `topic_id` | `bigint(20)` | FK -> `topics(id)`, NOT NULL | Chủ đề sở hữu template |
| `name` | `varchar(255)` | NOT NULL | Tên template (ví dụ: "Template Từ vựng Cơ bản", "Template Nghe đoán từ") |
| `created_at` | `datetime(6)` | NOT NULL | Thời điểm tạo |
| `updated_at` | `datetime(6)` | NOT NULL | Thời điểm cập nhật |

#### Bảng `template_elements`

Lưu các phần tử thành phần của một template theo thứ tự hiển thị.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh phần tử |
| `template_id` | `bigint(20)` | FK -> `templates(id)`, NOT NULL | Template chứa phần tử |
| `position` | `int(11)` | NOT NULL | Thứ tự vị trí xuất hiện (0, 1, 2...) |
| `type` | `varchar(50)` | NOT NULL | Kiểu phần tử: `FIELD`, `DIVIDER`, `BUTTON` |

> Ràng buộc duy nhất: `uk_template_element_position (template_id, position)`.

#### Bảng `template_fields`

Lưu chi tiết cấu hình hiển thị cho các phần tử kiểu `FIELD`.

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | `bigint(20)` | PK, AUTO_INCREMENT | Định danh cấu hình field |
| `element_id` | `bigint(20)` | FK -> `template_elements(id)`, NOT NULL, UNIQUE | Phần tử tương ứng (quan hệ 1-1) |
| `schema_attribute_id` | `bigint(20)` | FK -> `schema_attributes(id)`, NOT NULL | Thuộc tính dữ liệu được hiển thị |
| `semantic_role` | `varchar(50)` | NULLABLE | Vai trò ngữ nghĩa (`FRONT`, `BACK`, `EXAMPLE`, `AUDIO`, `IMAGE`, v.v.) |
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
    topic_schemas ||--o{ schema_attribute_groups : contains
    schema_attribute_groups ||--o{ schema_attributes : contains
    topics ||--o{ topic_items : contains
    topic_items ||--o{ topic_item_attribute_groups : has
    topic_item_attribute_groups ||--o{ topic_item_attribute_values : contains
    schema_attributes ||--o{ topic_item_attribute_values : "defines schema for"
    topics ||--o{ templates : "configures"
    templates ||--o{ template_elements : contains
    template_elements ||--o| template_fields : "specifies (type=FIELD)"
    schema_attributes ||--o{ template_fields : "mapped to"
    users ||--o{ fsrs_records : reviews
    topic_items ||--o{ fsrs_records : "tracked by"

    topic_schemas {
        bigint id PK
        bigint topic_id FK_UK
        datetime created_at
        datetime updated_at
    }

    schema_attribute_groups {
        bigint id PK
        bigint schema_id FK
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
        bigint topic_id FK
        varchar name
        datetime created_at
        datetime updated_at
    }

    template_elements {
        bigint id PK
        bigint template_id FK
        int position
        varchar type "FIELD, DIVIDER, BUTTON"
    }

    template_fields {
        bigint id PK
        bigint element_id FK_UK
        bigint schema_attribute_id FK
        varchar semantic_role "FRONT, BACK, EXAMPLE, AUDIO..."
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
| `topics` | Đơn vị tổ chức kiến thức | Sở hữu schema thuộc tính riêng (1-1) và liên kết với template thẻ học. |
| `topic_schemas` | Lược đồ thuộc tính của Topic | Quản lý tập hợp các nhóm thuộc tính động của từng chủ đề. |
| `schema_attribute_groups` | Nhóm thuộc tính schema | Gom nhóm các thuộc tính liên quan (ví dụ main, examples). |
| `schema_attributes` | Định nghĩa thuộc tính | Tên trường, nhãn, kiểu dữ liệu, thứ tự hiển thị cơ bản. |
| `topic_items` | Mục từ vựng thực tế | Từng mục kiến thức trong chủ đề, mang các giá trị thuộc tính tương ứng. |
| `templates` | Cấu hình giao diện thẻ của Topic | Mỗi topic có thể có template xác định cách render flashcard cho toàn bộ các item. |
| `template_elements` | Khối phần tử trên thẻ | Lưu thứ tự `position` và phân loại phần tử (`FIELD`, `DIVIDER`, `BUTTON`). |
| `template_fields` | Thiết lập trường hiển thị | Map phần tử với `schema_attribute_id`, gán `semantic_role` và các thuộc tính styling. |
| `fsrs_records` | Trạng thái ghi nhớ FSRS | Theo dõi độ ổn định (stability), độ khó (difficulty) và lịch ôn tập `due` cho từng `(user_id, topic_item_id)`. |

---

## 5. Enumeration trong Mã nguồn Backend

### `SemanticRole`

Định nghĩa vai trò ngữ nghĩa của từng trường dữ liệu khi hiển thị trên thẻ flashcard:

```java
public enum SemanticRole {
    FRONT,          // Mặt trước thẻ (từ khóa chính, câu hỏi)
    BACK,           // Mặt sau thẻ (giải nghĩa chính, câu trả lời)
    EXAMPLE,        // Câu ví dụ hoặc ngữ cảnh
    AUDIO,          // Âm thanh phát âm
    IMAGE,          // Ảnh minh họa
    PHONETIC,       // Phiên âm ngữ âm (IPA)
    TRANSLATION,    // Bản dịch nghĩa tiếng Việt bổ trợ
    HINT,           // Gợi ý khi cần
    TAG,            // Nhãn phân loại hoặc cấp độ
    EXTRA           // Thông tin phụ hoặc ghi chú
}
```

### `TemplateElementType`

Phân loại phần tử bố cục trong template:

```java
public enum TemplateElementType {
    FIELD,          // Trường dữ liệu hiển thị (liên kết 1-1 với TemplateField)
    DIVIDER,        // Đường kẻ phân tách bố cục (ví dụ ngăn cách Front và Back)
    BUTTON          // Nút tương tác (nút lật thẻ, nút nghe âm thanh)
}
```

### `Alignment`

Căn lề văn bản của trường hiển thị:

```java
public enum Alignment {
    LEFT,
    CENTER,
    RIGHT
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

### 6.1. Quản lý Template & Bố cục

1. **Gắn kết theo Topic**: Mỗi `Topic` có một `Template` quy định layout flashcard cho toàn bộ các `TopicItem` thuộc chủ đề đó.
2. **Tính toàn vẹn của thứ tự (`position`)**: Trường `position` trong `template_elements` phải là số nguyên không âm và là duy nhất trong phạm vi một template (ràng buộc `uk_template_element_position`). Khi client hiển thị, các phần tử được sắp xếp theo thứ tự `position ASC`.
3. **Quan hệ 1-1 giữa Element và Field**: Mỗi phần tử có kiểu `type = FIELD` bắt buộc phải có đúng một bản ghi `template_fields` tương ứng; các kiểu `DIVIDER` hoặc `BUTTON` không chứa `template_fields`.
4. **Tính hợp lệ của thuộc tính**: `topic_attribute_id` trong `template_fields` phải thuộc về danh mục thuộc tính của chính `Topic` đó (thông qua `topic_attribute_groups`).
5. **Xóa tầng (Cascade delete)**: Khi xóa một `Topic`, hệ thống cascade xóa `Template`, toàn bộ `TemplateElement`, `TemplateField` và `TopicItem` liên quan.

### 6.2. Hiển thị & Rendering trên Ứng dụng Di động

1. **Phân định hai mặt thẻ dựa theo `SemanticRole`**:
   - Mặt trước (Front): Hiển thị các trường có `semantic_role` là `FRONT`, kèm theo các trường hỗ trợ như `PHONETIC`, `AUDIO` (nếu có).
   - Mặt sau (Back): Hiển thị các trường có `semantic_role` là `BACK`, `TRANSLATION`, `EXAMPLE`, `EXTRA`, `HINT`.
2. **Ẩn trường trống (`hide_if_empty = true`)**: Nếu một mục từ không có dữ liệu cho thuộc tính tương ứng, ứng dụng di động sẽ tự động bỏ qua khối hiển thị đó mà không để lại khoảng trống bất thường.
3. **Hành vi âm thanh (`audio_action = true`)**: Khi người dùng nhấn vào trường có cờ này hoặc trường có vai trò `AUDIO`, ứng dụng sẽ kích hoạt phát file âm thanh phát âm.
4. **Định dạng linh hoạt**: Các thuộc tính `font_size`, `alignment`, `color` trên `template_fields` cho phép giao diện ứng dụng tự động áp dụng styling mà không cần can thiệp mã nguồn ứng dụng di động.

### 6.3. Quản lý Ôn tập FSRS

1. **Theo dõi tiến trình trực tiếp**: Trạng thái ôn tập của từng người học được lưu tại bảng `fsrs_records` cho từng cặp `(user_id, topic_item_id)`.
2. **Độc lập giao diện**: Thay đổi cấu hình template của Topic chỉ làm thay đổi cách hiển thị thẻ, hoàn toàn không làm gián đoạn hoặc sai lệch các tham số FSRS (`stability`, `difficulty`, `due`, `reps`, `lapses`).

---

## 7. Thiết kế API Endpoints

### 7.1. Quản lý Topic Template

| Phương thức | Đường dẫn | Mô tả | Quyền truy cập |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/v1/topics/{topicId}/template` | Lấy thông tin template và danh sách elements, fields của chủ đề | Bearer JWT |
| `PUT` | `/api/v1/topics/{topicId}/template` | Cập nhật cấu hình template cho chủ đề | Bearer JWT |

#### Response mẫu: Cấu hình Template của Topic

```json
{
  "statusCode": 200,
  "data": {
    "id": 12,
    "topicId": 45,
    "name": "Template Từ vựng Tiếng Anh Chuẩn",
    "elements": [
      {
        "id": 101,
        "position": 0,
        "type": "FIELD",
        "field": {
          "id": 201,
          "topicAttributeId": 5,
          "semanticRole": "FRONT",
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
          "topicAttributeId": 6,
          "semanticRole": "PHONETIC",
          "fieldLabel": "Phiên âm",
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
        "type": "DIVIDER",
        "field": null
      },
      {
        "id": 104,
        "position": 3,
        "type": "FIELD",
        "field": {
          "id": 203,
          "topicAttributeId": 7,
          "semanticRole": "BACK",
          "fieldLabel": "Nghĩa",
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
          "topicAttributeId": 8,
          "semanticRole": "EXAMPLE",
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
| `GET` | `/api/v1/topics/{topicId}/study-session` | Tải phiên ôn tập: bao gồm template và các mục đến hạn kèm dữ liệu thuộc tính | Bearer JWT |
| `POST` | `/api/v1/fsrs-records/{id}/review` | Gửi kết quả đánh giá thẻ (Again, Hard, Good, Easy) | Bearer JWT |

#### Response mẫu: Dữ liệu phiên học Flashcard

```json
{
  "statusCode": 200,
  "data": {
    "topicId": 45,
    "template": {
      "id": 12,
      "name": "Template Từ vựng Tiếng Anh Chuẩn",
      "elements": [
        {
          "position": 0,
          "type": "FIELD",
          "semanticRole": "FRONT",
          "attributeId": 5,
          "fontSize": 24,
          "alignment": "CENTER"
        },
        {
          "position": 1,
          "type": "FIELD",
          "semanticRole": "BACK",
          "attributeId": 7,
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
          { "attributeId": 5, "attributeName": "word", "value": "perseverance" },
          { "attributeId": 6, "attributeName": "ipa", "value": "/ˌpɜːrsəˈvɪərəns/" },
          { "attributeId": 7, "attributeName": "meaning", "value": "sự kiên trì, bền bỉ" },
          { "attributeId": 8, "attributeName": "example", "value": "Success requires perseverance." }
        ]
      }
    ]
  }
}
```

---

## 8. Dữ liệu Mẫu (Seed Data)

Dưới đây là kịch bản SQL mẫu khởi tạo template cơ bản cho một chủ đề từ vựng:

```sql
-- 1. Khởi tạo template cho topic_id = 1
INSERT INTO templates (id, topic_id, name, created_at, updated_at)
VALUES (1, 1, 'Mẫu Thẻ Từ Vựng Cơ Bản', NOW(), NOW());

-- 2. Khởi tạo các phần tử TemplateElement
INSERT INTO template_elements (id, template_id, position, type) VALUES
(1, 1, 0, 'FIELD'),
(2, 1, 1, 'FIELD'),
(3, 1, 2, 'DIVIDER'),
(4, 1, 3, 'FIELD'),
(5, 1, 4, 'FIELD');

-- 3. Cấu hình chi tiết TemplateField
INSERT INTO template_fields (element_id, topic_attribute_id, semantic_role, field_label, hide_if_empty, audio_action, font_size, alignment, color) VALUES
(1, 1, 'FRONT', 'Từ vựng', 0, 0, 24, 'CENTER', '#111827'),
(2, 2, 'PHONETIC', 'Phiên âm', 1, 1, 16, 'CENTER', '#6B7280'),
(4, 3, 'BACK', 'Giải nghĩa', 0, 0, 20, 'LEFT', '#1F2937'),
(5, 4, 'EXAMPLE', 'Ví dụ', 1, 0, 15, 'LEFT', '#4B5563');
```

---

## 9. Chuyển dịch Kiến trúc & Tương thích

Hệ thống đã hoàn tất tái cấu trúc, thay thế hoàn toàn mô hình thực thể cũ (`decks`, `notes`, `cards`, `card_templates`) sang mô hình mới:

1. **Từ vựng & Thư mục**: Thay thế `Deck` bằng `Collection` (hỗ trợ phân loại `SYSTEM` hoặc `USER`) và `Topic` (hỗ trợ quan hệ phân cấp cha - con).
2. **Nội dung thẻ**: Thay thế bảng `Note` cứng bằng `TopicItem` kết hợp thuộc tính động EAV (`topic_attributes`, `topic_item_attribute_values`).
3. **Mẫu hiển thị**: Thay thế `CardTemplate` cũ bằng bộ ba `templates`, `template_elements`, `template_fields` gắn liền với `Topic`.
4. **Theo dõi ôn tập**: Thay thế bảng `Card` bằng `fsrs_records` kết nối trực tiếp `users` và `topic_items`.

---

## 10. Tương tác với các Phân hệ Khác

| Phân hệ | Mối liên hệ và tương tác |
| :--- | :--- |
| **Thuật toán SRS FSRS** | FSRS tính toán lịch ôn tập và lưu trữ trực tiếp trên bảng `fsrs_records`. Template chỉ quyết định lớp hiển thị của thẻ, không làm thay đổi các biến số tính toán của thuật toán. |
| **Quét từ vựng (Scan-to-Vocabulary)** | Dữ liệu từ vựng nhận diện qua OCR/LLM sau khi xác nhận sẽ được lưu thành `TopicItem` thuộc một `Topic` đã chọn. Mục từ này ngay lập tức thừa hưởng template hiển thị của Topic đó. |
| **Gamification & Thống kê** | Mỗi lượt gửi kết quả đánh giá FSRS thành công được tính vào chỉ số hoàn thành mục tiêu học tập hàng ngày và tích lũy điểm kinh nghiệm (XP) cho người học. |

---

## 11. Phụ lục: Lịch sử Quyết định Thiết kế

Trong quá trình xây dựng hệ thống flashcard, bài toán quản lý giao diện thẻ học đã được cân nhắc qua các phương án:

1. **Phương án cấu hình cố định trên từng thẻ**: Lưu loại thẻ cố định trên từng bản ghi. Bị loại bỏ vì không đáp ứng được yêu cầu mở rộng thuộc tính linh hoạt theo chủ đề.
2. **Phương án cấu hình template độc lập tự do**: Cho phép người dùng tạo template rời và gán nhiều template vào một danh mục từ. Bị loại bỏ do gây phức tạp hóa trải nghiệm trên ứng dụng di động và làm phát sinh bài toán trùng lặp thẻ anh em (sibling cards).
3. **Phương án Topic Template gắn thuộc tính ngữ nghĩa (`SemanticRole`)**: Mỗi `Topic` quản lý một bộ thuộc tính (`TopicAttribute`) và có một `Template` định nghĩa bố cục cùng vai trò ngữ nghĩa của các thuộc tính đó. Đây là **phương án được phê duyệt chính thức**.

**Lợi ích của thiết kế hiện tại:**
- **Nhất quán mô hình dữ liệu**: Tương thích hoàn toàn với mô hình EAV của `TopicItem`, dữ liệu không bị nhân bản thừa thãi.
- **Tách bạch giao diện và thuật toán**: Bố cục thẻ (`Template`) độc lập với trạng thái ghi nhớ của người học (`fsrs_records`).
- **Tối ưu trải nghiệm di động**: Ứng dụng di động chỉ cần đọc cấu hình `position` và `semantic_role` để hiển thị giao diện mượt mà, không yêu cầu phân tích cú pháp HTML/CSS phức tạp.
