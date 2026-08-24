# Database Schema

Tài liệu này mô tả kiến trúc cơ sở dữ liệu của hệ thống SnapVocab, được tổng hợp từ Domain (Backend), schema `crawler.db` và các thiết kế mới trong thư mục `decisions` (`custom_card.md`, `daily_mission.md`).

## 1. Core Domain (Authentication & Dictionary)

Quản lý người dùng và hệ thống từ điển nền tảng.

### Bảng `User`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID người dùng |
| `password_hash` | String | Not Null | Mật khẩu (mã hóa) |
| `first_name` | String | Not Null | Tên (First name) |
| `last_name` | String | | Họ (Last name) |
| `email` | String | Unique | Địa chỉ email |
| `avatar_url` | Text | | Đường dẫn ảnh đại diện |
| `native_language`| String | | Ngôn ngữ mẹ đẻ |
| `learning_language`| String | | Ngôn ngữ đang học |
| `exp` | Long | Default 0 | Điểm kinh nghiệm |
| `coin` | Long | Default 0 | Tiền tệ trong game |
| `streak_days` | Integer| Default 0 | Chuỗi ngày học liên tục |
| `last_studied_at`| Instant| | Lần học gần nhất |
| `activated` | Boolean| Not Null, Default false| Trạng thái kích hoạt |
| `bio` | String | | Tiểu sử |

### Bảng `Authority`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `name` | String | Khóa chính (PK) | Tên quyền (VD: ROLE_USER, ROLE_ADMIN) |

*(Lưu ý: Mối quan hệ giữa User và Authority là N-N qua bảng trung gian `user_authority`)*

### Bảng `Word`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID từ vựng |
| `word` | String | Not Null | Nội dung từ vựng |
| `langCode` | String | Not Null | Mã ngôn ngữ (VD: en, vi) |

### Bảng `Definition`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID định nghĩa |
| `definition` | String | Not Null | Nội dung định nghĩa |
| `pos` | String | | Loại từ (Part of Speech) |
| `subPos` | String | | Phân loại phụ của loại từ |
| `definitionLang`| String | | Ngôn ngữ của định nghĩa |
| `links` | String | | Các liên kết mở rộng |

### Bảng `WordDefinition` (Bảng trung gian)
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID liên kết từ - định nghĩa |
| `word_id` | Long | FK -> `Word(id)` | Tham chiếu đến Word |
| `definition_id`| Long | FK -> `Definition(id)` | Tham chiếu đến Definition |
| `example` | Text | | Ví dụ minh họa cách sử dụng từ theo định nghĩa này |

### Bảng `Translation`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID bản dịch |
| `word_id` | Long | FK -> `Word(id)` | Tham chiếu đến Word |
| `translation` | String | | Nội dung dịch nghĩa |
| `targetLang` | String | | Ngôn ngữ đích |

### Bảng `Pronunciation`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID phát âm |
| `word_id` | Long | FK -> `Word(id)` | Tham chiếu đến Word |
| `ipa` | String | | Phiên âm quốc tế IPA |
| `audioUrl` | String | | Đường dẫn file audio |

### Bảng `WordRelation`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID liên kết |
| `word_id` | Long | FK -> `Word(id)` | Tham chiếu đến Word gốc |
| `relatedWord` | String | | Từ liên quan (dạng text) |
| `relationType` | String | | Loại quan hệ (synonym, antonym...) |

### Ràng buộc & Indexes (Core Domain)
- `User`: Unique index trên `email`.
- `Word`: Index trên `word` để tìm kiếm từ vựng hiệu quả.
- `user_authority`: Khóa chính ghép `(user_id, authority_name)`.
- Các FK (`word_id`, `definition_id`) cần có index để tối ưu truy vấn join.

## 2. Crawler & Topic Domain

Cấu trúc thu thập và tổ chức dữ liệu từ vựng theo chủ đề.

### Bảng `collections`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID bộ sưu tập |
| `name` | String | Unique, Not Null | Tên bộ sưu tập |
| `translation` | String | | Tên dịch nghĩa |

### Bảng `topics`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID chủ đề |
| `collection_id` | Long | FK -> `collections(id)` | Chủ đề thuộc bộ sưu tập nào |
| `parent_id` | Long | FK -> `topics(id)` | Chủ đề cha (nếu là sub-topic) |
| `name` | String | Not Null | Tên chủ đề |
| `translation` | String | | Dịch nghĩa chủ đề |
| `description` | Text | | Mô tả |

### Bảng `topic_attribute_groups`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID nhóm thuộc tính |
| `topic_id` | Long | FK -> `topics(id)` | Nhóm thuộc tính của chủ đề nào |
| `name` | String | Not Null | Tên nhóm thuộc tính |
| `multiple` | Boolean | Not Null | Cho phép nhiều thuộc tính không |
| `position` | SmallInt | Not Null | Vị trí hiển thị |

### Bảng `topic_attributes`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID thuộc tính |
| `group_id` | Long | FK -> `topic_attribute_groups(id)` | Thuộc tính thuộc nhóm nào |
| `name` | String | Not Null | Tên thuộc tính |
| `data_type` | String | Not Null | Kiểu dữ liệu |
| `required` | Boolean | Not Null | Bắt buộc không |
| `position` | SmallInt | Not Null | Vị trí hiển thị |

### Bảng `topic_items`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID item (thường là 1 từ) trong chủ đề |
| `topic_id` | Long | FK -> `topics(id)` | Item thuộc chủ đề nào |

### Bảng `topic_item_attribute_groups`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID instance của nhóm thuộc tính cho 1 item |
| `topic_item_id` | Long | FK -> `topic_items(id)` | Item tương ứng |
| `group_definition_id`| Long | FK -> `topic_attribute_groups(id)`| Nhóm thuộc tính gốc |
| `position` | SmallInt| Not Null | Vị trí |

### Bảng `topic_item_attribute_values`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID giá trị thuộc tính |
| `group_instance_id` | Long | FK -> `topic_item_attribute_groups(id)`| Instance nhóm tương ứng |
| `topic_attribute_id`| Long | FK -> `topic_attributes(id)` | Thuộc tính gốc |
| `value` | Text | Not Null | Giá trị thực tế được lưu |

### Ràng buộc & Indexes (Crawler & Topic Domain)
- `collections`: Unique index trên `name`.
- `topics`: Index trên `collection_id` và `parent_id`.
- FK indexes cho `topic_attributes(group_id)`, `topic_items(topic_id)`.

## 3. Flashcard & Spaced Repetition (SRS)

Hỗ trợ hệ thống Card Template mới (`custom_card.md`). 1 Note chỉ có 1 Card duy nhất.

### Bảng `CardTemplate`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID Template |
| `code` | String | Unique, Nullable| Mã system template (VD: CLASSIC) |
| `name` | String | Not Null | Tên hiển thị |
| `description` | String | | Mô tả template |
| `isSystem` | Boolean | Not Null | Là template hệ thống (không xóa) |
| `interactionType`| Enum | Not Null | Kiểu tương tác (FLIP, TYPE_IN, TAP_TO_REVEAL) |
| `baseLayout` | Enum | Not Null | Bố cục cơ bản |
| `userId` | Long | FK -> `User(id)` | Người tạo (nếu là custom), Null nếu là system |
| `isDeleted` | Boolean | | Đánh dấu xóa mềm |
| `createdAt` | Instant| | Thời điểm tạo |
| `updatedAt` | Instant| | Thời điểm cập nhật |

### Bảng `CardTemplateField`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID Trường template |
| `cardTemplateId` | Long | FK -> `CardTemplate(id)` | Tham chiếu Template |
| `fieldCode` | Enum | Not Null | Loại trường (WORD, MEANING, IPA, AUDIO, IMAGE...) |
| `side` | Enum | Not Null | Mặt trước/sau (FRONT/BACK) |
| `displayOrder` | Integer | Not Null | Thứ tự hiển thị |
| `isPrimary` | Boolean | Not Null | Là trường chính (nổi bật) |
| `fieldConfig` | JSON | | Cấu hình phụ (autoplay, maskPattern) |

### Bảng `Deck`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID Bộ thẻ |
| `userId` | Long | FK -> `User(id)` | Người sở hữu bộ thẻ |
| `cardTemplateId` | Long | FK -> `CardTemplate(id)` | Template hiển thị mặc định của bộ thẻ |
| `name` | String | | Tên bộ thẻ |
| `description` | String | | Mô tả |
| `createdAt` | Instant| | Thời điểm tạo |
| `updatedAt` | Instant| | Thời điểm cập nhật |

### Bảng `Note`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID Note dữ liệu gốc |
| `deckId` | Long | FK -> `Deck(id)` | Note thuộc bộ thẻ nào |
| `word` | String | | Từ vựng |
| `createdAt` | Instant| | Thời điểm tạo |
| `updatedAt` | Instant| | Thời điểm cập nhật |

### Bảng `NoteMeaning`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID nghĩa |
| `noteId` | Long | FK -> `Note(id)` | Thuộc Note nào |
| `meaning` | String | | Nội dung tiếng Việt |
| `partOfSpeech` | String | | Loại từ |
| `example` | String | | Câu ví dụ |
| `personalNote` | String | | Ghi chú cá nhân |

### Bảng `NotePronunciation`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID phát âm |
| `noteId` | Long | FK -> `Note(id)` | Thuộc Note nào |
| `ipa` | String | Not Null | Phiên âm quốc tế |

### Bảng `Card`
| Field | Type | Quan hệ / Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | Long | Khóa chính (PK) | ID Thẻ học (Dữ liệu SRS) |
| `noteId` | Long | FK -> `Note(id)`, Unique| Tương ứng với đúng 1 Note |
| `cardState` | Enum | | Trạng thái SRS |
| `dueAt` | Instant| | Thời điểm đến hạn ôn tập |
| `stability` | Double | | Chỉ số độ bền trí nhớ |
| `difficulty` | Double | | Chỉ số độ khó thẻ |
| `repetitions` | Integer| | Số lần ôn tập |
| `lapses` | Integer| | Số lần quên (quét sai) |
| `lastReviewedAt`| Instant| | Thời điểm review gần nhất |
| `createdAt` | Instant| | Thời điểm tạo |
| `updatedAt` | Instant| | Thời điểm cập nhật |

### Ràng buộc & Indexes (Flashcard & SRS)
- `CardTemplate`: Unique index trên `code`.
- `CardTemplateField`: Unique constraint ghép `(cardTemplateId, fieldCode, side)`.
- `Card`: Unique constraint trên `noteId`. Thêm Index trên `dueAt` và `cardState` để lấy danh sách review nhanh.
- Các FK (`userId`, `deckId`, `cardTemplateId`, `noteId`) cần có index.

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
| `userDailyMissionId`| UUID | FK -> `UserDailyMission(id)`| Nhận thưởng từ nhiệm vụ nào |
| `userId` | Long | FK -> `User(id)` | Người nhận thưởng |
| `idempotencyKey`| String | Unique | Key chống nhận thưởng nhiều lần |
| `rewardCoin` | Number | | Số xu thực tế được cộng |
| `rewardXp` | Number | | Điểm XP thực tế được cộng |
| `rewardItemCode`| String | Nullable | Vật phẩm đã cộng |
| `claimedBy` | Enum | | Ghi nhận người claim (USER) |
| `createdAt` | DateTime| | Thời điểm claim |

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

### Ràng buộc & Indexes (Daily Mission)
- `UserDailyMission`: Unique constraint ghép `(userId, missionDate, slot)` và `(userId, missionDate, missionTemplateId)`. Ràng buộc `currentProgress <= targetValue`.
- `UserDailyMissionClaimLog`: Unique index trên `idempotencyKey`.
- `UserWeeklyMilestone`: Unique constraint ghép `(userId, weekStartDate)`.
- Ràng buộc: `activityStampCount` nằm trong khoảng 0-7.
