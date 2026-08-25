# Đặc tả tính năng: Daily Mission

Tài liệu này định nghĩa cơ chế **Daily Mission** cho SnapVocab nhằm khuyến khích Learner duy trì thói quen học từ vựng hằng ngày thông qua nhiệm vụ ngắn hạn, phần thưởng Coin/XP, Daily Chest và Weekly Milestone.

---

## 1. Phạm vi

### 1.1. Mục tiêu

Daily Mission cần đạt các mục tiêu sau:

- Tạo lý do rõ ràng để Learner quay lại học mỗi ngày.
- Khuyến khích đầy đủ luồng học chính của SnapVocab: scan từ mới, lưu từ, học flashcard, làm quiz, ôn SRS.
- Tăng retention bằng Daily Chest và Weekly Milestone mà không tạo thêm hệ thống nhiệm vụ tuần phức tạp.
- Giữ reward economy ổn định, dễ đo lường và khó gian lận.

### 1.2. Ngoài phạm vi

Daily Mission **không bao gồm** các nhiệm vụ cộng đồng hoặc nhóm học tập, ví dụ:

- Tham gia Study Group.
- Bình luận bài đăng cộng đồng.
- Đóng góp điểm cho bảng xếp hạng nhóm.
- Mời bạn bè vào nhóm.

SnapVocab vẫn có thể dùng các nhiệm vụ liên quan đến **leaderboard cá nhân**, XP cá nhân, streak cá nhân và shop cá nhân.

---

## 2. Trải nghiệm người dùng

### 2.1. Cấu trúc nhiệm vụ mỗi ngày

Mỗi Learner nhận được:

- **5 Daily Missions bắt buộc**.
- **Tối đa 1 Bonus Mission tùy chọn** nếu hệ thống có nhiệm vụ phù hợp.

Daily Chest được mở khi Learner hoàn thành và claim đủ **5/5 Daily Missions bắt buộc**. Bonus Mission không bắt buộc để mở Daily Chest.

### 2.2. Cân bằng danh mục nhiệm vụ

5 Daily Missions bắt buộc mỗi ngày nên được phân bổ theo cấu trúc:

| Vị trí | Nhóm nhiệm vụ | Mục đích |
| :--- | :--- | :--- |
| 1 | Scan & Capture | Thúc đẩy luồng scan-to-learn |
| 2 | Vocabulary Building | Làm giàu bộ từ cá nhân |
| 3 | Flashcard / SRS | Tạo thói quen ôn tập |
| 4 | Quiz / Accuracy | Đo chất lượng ghi nhớ |
| 5 | Retention / Personal Gamification | Duy trì streak, XP, leaderboard cá nhân hoặc shop |

Nếu một nhóm không có nhiệm vụ phù hợp với ngữ cảnh của Learner, hệ thống chọn nhiệm vụ thay thế từ nhóm học tập cốt lõi khác.

### 2.3. Trạng thái hiển thị

Mỗi mission trên UI cần thể hiện:

- Tên nhiệm vụ.
- Mô tả ngắn.
- Tiến độ hiện tại, ví dụ `2/5`.
- Phần thưởng Coin/XP.
- Trạng thái: `Đang làm`, `Hoàn thành`, `Đã nhận thưởng`.
- Nút `Nhận thưởng` khi nhiệm vụ đã hoàn thành nhưng chưa claim.

---

## 3. Chu kỳ ngày và claim reward

### 3.1. Timezone

MVP sử dụng timezone cố định: **Asia/Ho_Chi_Minh (GMT+7)**.

Một ngày nhiệm vụ bắt đầu lúc `00:00:00` và kết thúc lúc `23:59:59` theo GMT+7.

### 3.2. Reset nhiệm vụ

Khi sang ngày mới:

- Hệ thống tạo bộ Daily Missions mới cho ngày hiện tại.
- Mission chưa hoàn thành của ngày trước không được cộng dồn.
- Mission đã hoàn thành nhưng chưa claim sẽ hết hạn theo rule ở mục 3.3.

### 3.3. Claim reward

Daily Mission sử dụng cơ chế **manual claim** để tạo cảm giác nhận thưởng rõ ràng trên UI.

Rule xử lý:

- Khi tiến độ đạt target, mission chuyển sang `COMPLETED`.
- Learner nhấn `Nhận thưởng` để nhận Coin/XP, mission chuyển sang `CLAIMED`.
- Nếu mission đã `COMPLETED` nhưng Learner chưa claim trước khi reset ngày, mission chuyển sang `EXPIRED` và **không nhận thưởng**.
- Mission chưa đạt target trước reset cũng chuyển sang `EXPIRED` và không nhận thưởng.
- UI cần nhấn mạnh thời hạn nhận thưởng để Learner hiểu rằng hoàn thành nhiệm vụ chưa đồng nghĩa với đã nhận reward.

---

## 4. Phân bổ nhiệm vụ

### 4.1. Mission Pool

Mission Pool là danh sách template nhiệm vụ có thể được cấp cho Learner. Mỗi template cần có:

- Nhóm nhiệm vụ.
- Điều kiện đủ để được cấp.
- Loại event dùng để cập nhật tiến độ.
- Target mặc định.
- Reward mặc định.
- Weight để random.

### 4.2. Context-aware filtering

Trước khi random, hệ thống lọc Mission Pool theo ngữ cảnh của Learner.

Ví dụ:

- Nếu Review Queue snapshot rỗng, không cấp mission `Hoàn thành ôn tập SRS hôm nay`.
- Nếu Learner chưa có từ nào từng học sai, không cấp mission `Học lại 5 thẻ sai`.
- Nếu Learner chưa mở khóa booster/shop, không cấp mission `Dùng 1 booster XP`.
- Nếu Learner không có dữ liệu leaderboard cá nhân trong ngày, có thể thay bằng nhiệm vụ XP hoặc streak.

### 4.3. Weighted random

Sau khi lọc theo ngữ cảnh:

1. Hệ thống chọn 1 nhiệm vụ phù hợp cho từng vị trí trong cấu trúc 5 nhóm.
2. Nếu một nhóm thiếu template hợp lệ, hệ thống chọn template thay thế từ nhóm học tập cốt lõi.
3. Hệ thống tránh cấp cùng một `missionTemplate.code` cho cùng Learner trong 2 ngày liên tiếp nếu còn lựa chọn khác.
4. Bonus Mission được random từ các template còn lại, ưu tiên nhiệm vụ có độ khó cao hơn hoặc reward nhỏ nhưng tạo engagement thêm.

---

## 5. Mission Pool đề xuất

### 5.1. Scan & Capture

| Code | Tên UI | Điều kiện hoàn thành | Target | Reward |
| :--- | :--- | :--- | :--- | :--- |
| `SCAN_SAVE_3_WORDS` | Thợ Săn Từ Vựng | Scan và lưu thành công 3 từ mới từ camera/ảnh | 3 | 10 Coin, 20 XP |
| `SCAN_SAVE_5_WORDS` | Máy Quét Siêng Năng | Scan và lưu thành công 5 từ mới | 5 | 15 Coin, 30 XP |
| `SCAN_FROM_PARAGRAPH` | Đọc Một Đoạn Hay | Scan từ một đoạn văn và lưu ít nhất 2 từ | 2 | 10 Coin, 25 XP |
| `SAVE_WORD_FROM_IMAGE` | Bắt Chữ Từ Ảnh | Lưu 1 từ vựng mới từ ảnh | 1 | 5 Coin, 10 XP |
| `EDIT_SCANNED_WORD` | Chỉnh Từ Cho Chuẩn | Chỉnh sửa nghĩa, phiên âm hoặc ví dụ cho 1 từ đã scan | 1 | 5 Coin, 10 XP |

### 5.2. Vocabulary Building

| Code | Tên UI | Điều kiện hoàn thành | Target | Reward |
| :--- | :--- | :--- | :--- | :--- |
| `ADD_5_WORDS` | Xây Kho Từ Vựng | Thêm 5 từ vào bộ từ cá nhân | 5 | 15 Coin, 25 XP |
| `TAG_3_WORDS` | Sắp Xếp Gọn Gàng | Gắn tag/chủ đề cho 3 từ | 3 | 5 Coin, 15 XP |
| `REVIEW_YESTERDAY_WORDS` | Gặp Lại Bạn Cũ | Xem lại 5 từ đã lưu hôm qua | 5 | 10 Coin, 20 XP |
| `MARK_2_MASTERED` | Thành Thạo Hơn Mỗi Ngày | Đánh dấu mastered cho 2 từ đủ điều kiện | 2 | 10 Coin, 25 XP |
| `ADD_EXAMPLE_SENTENCE` | Đặt Câu Ghi Nhớ | Thêm ví dụ cho 1 từ trong bộ từ cá nhân | 1 | 5 Coin, 15 XP |

### 5.3. Flashcard & SRS

| Code | Tên UI | Điều kiện hoàn thành | Target | Reward |
| :--- | :--- | :--- | :--- | :--- |
| `STUDY_15_FLASHCARDS` | Học Giả Siêng Năng | Học 15 flashcards bất kỳ | 15 | 15 Coin, 30 XP |
| `FINISH_2_FLASHCARD_SESSIONS` | Hai Lượt Ôn Nhanh | Hoàn thành 2 lượt flashcard | 2 | 15 Coin, 30 XP |
| `ANSWER_10_FLASHCARDS_CORRECT` | Nhớ Chính Xác | Trả lời đúng 10 flashcards | 10 | 15 Coin, 30 XP |
| `RETRY_5_WRONG_CARDS` | Không Né Từ Khó | Học lại 5 thẻ từng trả lời sai | 5 | 20 Coin, 35 XP |
| `CLEAR_SRS_SNAPSHOT` | Không Bỏ Cuộc | Hoàn thành 100% Review Queue snapshot của ngày | 100% | 30 Coin, 50 XP |
| `REVIEW_10_DUE_WORDS` | Ôn Đúng Hẹn | Ôn 10 từ đến hạn trong SRS | 10 | 20 Coin, 40 XP |

### 5.4. Quiz & Accuracy

| Code | Tên UI | Điều kiện hoàn thành | Target | Reward |
| :--- | :--- | :--- | :--- | :--- |
| `QUIZ_SCORE_80` | Trí Nhớ Siêu Phàm | Hoàn thành 1 quiz với độ chính xác >= 80% | 1 | 20 Coin, 40 XP |
| `FINISH_2_SHORT_QUIZZES` | Thử Tài Liên Tục | Hoàn thành 2 quiz ngắn | 2 | 15 Coin, 30 XP |
| `CORRECT_STREAK_5` | Chuỗi Đúng Ấn Tượng | Trả lời đúng liên tiếp 5 câu trong quiz | 5 | 15 Coin, 35 XP |
| `IMPROVE_QUIZ_SCORE` | Tốt Hơn Lần Trước | Cải thiện điểm quiz so với lần làm gần nhất cùng dạng | 1 | 15 Coin, 35 XP |
| `PERFECT_MINI_QUIZ` | Hoàn Hảo Mini Quiz | Hoàn thành 1 mini quiz không sai câu nào | 1 | 25 Coin, 45 XP |

### 5.5. Retention & Personal Gamification

| Code | Tên UI | Điều kiện hoàn thành | Target | Reward |
| :--- | :--- | :--- | :--- | :--- |
| `KEEP_DAILY_STREAK` | Giữ Lửa Mỗi Ngày | Hoàn thành ít nhất 1 hoạt động học trong ngày để giữ streak | 1 | 10 Coin, 20 XP |
| `EARN_DAILY_XP_TARGET` | Chạm Mốc XP | Nhận đủ XP mục tiêu ngày | 1 | 15 Coin, 30 XP |
| `CLIMB_PERSONAL_LEADERBOARD` | Nhích Lên Một Bậc | Tăng ít nhất 1 bậc trên leaderboard cá nhân | 1 | 20 Coin, 40 XP |
| `USE_XP_BOOSTER` | Tăng Tốc Học Tập | Dùng 1 booster XP | 1 | 5 Coin, 10 XP |
| `BUY_SHOP_ITEM` | Ghé Thăm Cửa Hàng | Mua 1 vật phẩm trong shop cá nhân | 1 | 5 Coin, 10 XP |

---

## 6. Tracking event

Backend cập nhật tiến độ nhiệm vụ thông qua event phát sinh từ các hành động học tập.

| Event | Khi nào phát sinh | Mission liên quan |
| :--- | :--- | :--- |
| `WORD_SCANNED_AND_SAVED` | User scan và lưu thành công từ mới | Scan & Capture, Vocabulary Building |
| `WORD_UPDATED` | User chỉnh sửa thông tin từ | `EDIT_SCANNED_WORD`, `ADD_EXAMPLE_SENTENCE` |
| `WORD_TAGGED` | User gắn tag/chủ đề cho từ | `TAG_3_WORDS` |
| `FLASHCARD_ANSWERED` | User trả lời flashcard | Flashcard, SRS |
| `FLASHCARD_SESSION_COMPLETED` | User kết thúc một lượt flashcard | `FINISH_2_FLASHCARD_SESSIONS` |
| `SRS_REVIEW_COMPLETED` | User ôn một item SRS đến hạn | SRS mission |
| `QUIZ_COMPLETED` | User hoàn thành quiz | Quiz mission |
| `XP_EARNED` | User nhận XP từ hoạt động hợp lệ | `EARN_DAILY_XP_TARGET` |
| `BOOSTER_USED` | User dùng booster | `USE_XP_BOOSTER` |
| `SHOP_ITEM_PURCHASED` | User mua vật phẩm shop | `BUY_SHOP_ITEM` |
| `LEADERBOARD_RANK_CHANGED` | Thứ hạng cá nhân của user tăng | `CLIMB_PERSONAL_LEADERBOARD` |

### 6.1. Quy tắc tính tiến độ

- Chỉ cập nhật mission của ngày hiện tại theo GMT+7.
- Chỉ tính hành động hợp lệ đã được backend xác nhận thành công.
- Không tính scan rỗng, scan trùng không lưu, quiz bị hủy, flashcard session chưa hoàn thành.
- `currentProgress` không vượt quá `targetValue`.
- Khi `currentProgress >= targetValue`, mission chuyển sang `COMPLETED`.

### 6.2. SRS snapshot

Với mission liên quan Review Queue:

- Hệ thống chụp snapshot danh sách SRS due tại thời điểm cấp Daily Missions.
- Target của mission được tính từ snapshot này.
- Các item mới đến hạn sau thời điểm snapshot không làm tăng target của mission ngày đó.
- Nếu snapshot rỗng, không cấp mission `CLEAR_SRS_SNAPSHOT`.

---

## 7. Data model bổ sung

### 7.1. `MissionTemplate`

| Field | Kiểu | Mô tả |
| :--- | :--- | :--- |
| `id` | UUID | Khóa chính |
| `code` | String | Mã duy nhất, ví dụ `SCAN_SAVE_3_WORDS` |
| `name` | String | Tên hiển thị |
| `description` | String | Mô tả ngắn |
| `type` | Enum | `DAILY`, `BONUS`, `CHEST` |
| `category` | Enum | `SCAN`, `VOCABULARY`, `FLASHCARD`, `SRS`, `QUIZ`, `RETENTION`, `SHOP` |
| `triggerEvent` | String | Event dùng để cập nhật tiến độ |
| `targetValue` | Number | Mục tiêu cần đạt |
| `rewardCoin` | Number | Coin thưởng |
| `rewardXp` | Number | XP thưởng |
| `rewardItemCode` | Nullable String | Vật phẩm thưởng nếu có |
| `weight` | Number | Trọng số random |
| `eligibilityRule` | JSON | Rule lọc theo ngữ cảnh |
| `isActive` | Boolean | Bật/tắt template |
| `createdAt` | DateTime | Thời điểm tạo |
| `updatedAt` | DateTime | Thời điểm cập nhật |

### 7.2. `UserDailyMission`

| Field | Kiểu | Mô tả |
| :--- | :--- | :--- |
| `id` | UUID | Khóa chính |
| `userId` | Long | Learner nhận mission |
| `missionTemplateId` | UUID | Template được cấp |
| `missionDate` | Date | Ngày nhiệm vụ theo GMT+7 |
| `slot` | Number | Vị trí 1-5, bonus dùng slot 6 |
| `targetValue` | Number | Target đã snapshot khi cấp mission |
| `currentProgress` | Number | Tiến độ hiện tại |
| `status` | Enum | `IN_PROGRESS`, `COMPLETED`, `CLAIMED`, `EXPIRED` |
| `snapshotData` | JSON | Dữ liệu snapshot, ví dụ danh sách SRS item due |
| `assignedAt` | DateTime | Thời điểm cấp mission |
| `completedAt` | Nullable DateTime | Thời điểm hoàn thành |
| `claimedAt` | Nullable DateTime | Thời điểm nhận thưởng |
| `updatedAt` | DateTime | Thời điểm cập nhật |

Ràng buộc dữ liệu:

- Unique `(userId, missionDate, slot)`.
- Unique `(userId, missionDate, missionTemplateId)`.
- `currentProgress <= targetValue`.
- Mission chỉ được claim một lần.

### 7.3. `UserDailyMissionClaimLog`

| Field | Kiểu | Mô tả |
| :--- | :--- | :--- |
| `id` | UUID | Khóa chính |
| `userDailyMissionId` | UUID | Mission được claim |
| `userId` | Long | Learner nhận thưởng |
| `idempotencyKey` | String | Khóa chống claim trùng |
| `rewardCoin` | Number | Coin đã cộng |
| `rewardXp` | Number | XP đã cộng |
| `rewardItemCode` | Nullable String | Vật phẩm đã cộng |
| `claimedBy` | Enum | `USER` |
| `createdAt` | DateTime | Thời điểm claim |

### 7.4. `UserWeeklyMilestone`

| Field | Kiểu | Mô tả |
| :--- | :--- | :--- |
| `id` | UUID | Khóa chính |
| `userId` | Long | Learner sở hữu milestone |
| `weekStartDate` | Date | Thứ 2 đầu tuần theo GMT+7 |
| `weekEndDate` | Date | Chủ Nhật cuối tuần theo GMT+7 |
| `activityStampCount` | Number | Số ngày đã claim Daily Chest trong tuần |
| `bronzeStatus` | Enum | `LOCKED`, `UNLOCKED`, `CLAIMED` |
| `silverStatus` | Enum | `LOCKED`, `UNLOCKED`, `CLAIMED` |
| `goldStatus` | Enum | `LOCKED`, `UNLOCKED`, `CLAIMED` |
| `updatedAt` | DateTime | Thời điểm cập nhật |

Ràng buộc dữ liệu:

- Unique `(userId, weekStartDate)`.
- `activityStampCount` nằm trong khoảng `0-7`.
- Mỗi mốc thưởng tuần chỉ được claim một lần.

---

## 8. API đề xuất

### 8.1. Lấy Daily Missions hôm nay

`GET /api/me/daily-missions`

Response mẫu:

```json
{
  "missionDate": "2026-08-24",
  "timezone": "Asia/Ho_Chi_Minh",
  "requiredMissionCount": 5,
  "dailyChestStatus": "LOCKED",
  "missions": [
    {
      "id": "user_mission_id",
      "code": "SCAN_SAVE_3_WORDS",
      "name": "Thợ Săn Từ Vựng",
      "description": "Scan và lưu thành công 3 từ mới từ camera/ảnh",
      "category": "SCAN",
      "slot": 1,
      "targetValue": 3,
      "currentProgress": 2,
      "status": "IN_PROGRESS",
      "reward": {
        "coin": 10,
        "xp": 20,
        "itemCode": null
      }
    }
  ]
}
```

### 8.2. Claim một mission

`POST /api/me/daily-missions/{userDailyMissionId}/claim`

Request header:

```http
Idempotency-Key: claim-user_mission_id-2026-08-24
```

Response mẫu:

```json
{
  "missionId": "user_mission_id",
  "status": "CLAIMED",
  "rewardGranted": {
    "coin": 10,
    "xp": 20,
    "itemCode": null
  },
  "dailyChestUnlocked": false
}
```

### 8.3. Claim Daily Chest

`POST /api/me/daily-missions/chest/claim`

Điều kiện:

- Có đúng 5 Daily Missions bắt buộc trong ngày.
- Cả 5 mission đã ở trạng thái `CLAIMED`.
- Daily Chest của ngày chưa được claim.

### 8.4. Lấy Weekly Milestone hiện tại

`GET /api/me/weekly-milestones/current`

Response mẫu:

```json
{
  "weekStartDate": "2026-08-24",
  "weekEndDate": "2026-08-30",
  "timezone": "Asia/Ho_Chi_Minh",
  "activityStampCount": 3,
  "milestones": [
    {
      "code": "BRONZE_WEEKLY_CHEST",
      "name": "Rương Đồng",
      "requiredStamps": 3,
      "status": "UNLOCKED",
      "reward": {
        "coin": 50,
        "xp": 0,
        "itemCode": null
      }
    },
    {
      "code": "SILVER_WEEKLY_CHEST",
      "name": "Rương Bạc",
      "requiredStamps": 5,
      "status": "LOCKED",
      "reward": {
        "coin": 100,
        "xp": 0,
        "itemCode": "XP_BOOSTER_1H"
      }
    },
    {
      "code": "GOLD_WEEKLY_CHEST",
      "name": "Rương Vàng",
      "requiredStamps": 7,
      "status": "LOCKED",
      "reward": {
        "coin": 200,
        "xp": 0,
        "itemCode": "SPECIAL_BADGE_OR_AVATAR_FRAME"
      }
    }
  ]
}
```

### 8.5. Claim Weekly Milestone

`POST /api/me/weekly-milestones/{milestoneCode}/claim`

Điều kiện:

- `activityStampCount` đạt mốc yêu cầu của milestone.
- Milestone đang ở trạng thái `UNLOCKED`.
- Milestone đó chưa được claim trong tuần hiện tại.

---

## 9. Business rules

1. **Idempotency:** Claim mission và claim Daily Chest phải idempotent. Gọi API nhiều lần chỉ cộng reward đúng 1 lần.
2. **Transaction:** Claim reward cần cập nhật mission status, cộng Coin/XP/item và ghi claim log trong cùng transaction.
3. **Anti-spam:** Không cộng tiến độ cho hành động không tạo giá trị học tập, ví dụ scan ảnh rỗng, lưu từ trùng, quiz bị hủy, flashcard session chưa kết thúc.
4. **Fairness:** Không cấp mission mà user không thể hoàn thành do thiếu dữ liệu hoặc chưa mở khóa tính năng.
5. **Reward cap:** Tổng reward Daily Missions + Daily Chest mỗi ngày cần nằm trong giới hạn economy đã định nghĩa.
6. **No rollover:** Mission chưa hoàn thành không được cộng dồn sang ngày sau.
7. **Manual claim deadline:** Mission đã hoàn thành nhưng chưa claim trước khi reset ngày sẽ hết hạn và không được cộng reward.
8. **Bonus isolation:** Bonus Mission có reward riêng nhưng không ảnh hưởng điều kiện mở Daily Chest.

---

## 10. Weekly Milestone

Weekly Milestone là một phần của scope chính thức. Daily Mission không tạo danh sách nhiệm vụ tuần riêng; thay vào đó, hệ thống dùng **Activity Stamp** để mở các mốc thưởng trong tuần.

### 10.1. Activity Stamp

Learner nhận 1 Activity Stamp cho ngày hiện tại khi claim Daily Chest.

Quy tắc:

- Mỗi ngày chỉ nhận tối đa 1 Activity Stamp.
- Activity Stamp chỉ được tạo sau khi Daily Chest được claim thành công.
- Nếu Learner hoàn thành 5/5 Daily Missions nhưng không claim đủ mission hoặc không claim Daily Chest trước reset ngày, ngày đó không có Activity Stamp.
- Activity Stamp không được cộng bù sau khi ngày đã reset.

### 10.2. Weekly Chests

Chu kỳ tuần tính từ Thứ 2 đến Chủ Nhật theo GMT+7.

| Mốc | Điều kiện | Reward đề xuất | Vai trò |
| :--- | :--- | :--- | :--- |
| Rương Đồng | 3 Activity Stamps trong tuần | 50 Coin | Mốc dễ đạt để tạo động lực giữa tuần |
| Rương Bạc | 5 Activity Stamps trong tuần | 100 Coin, 1 XP Booster 1 giờ | Mốc chính cho người học đều các ngày trong tuần |
| Rương Vàng | 7 Activity Stamps trong tuần | 200 Coin, huy hiệu đặc biệt hoặc khung avatar | Mốc danh giá cho người duy trì đủ cả tuần |

### 10.3. Quy tắc claim Weekly Chest

- Mỗi mốc chỉ được claim 1 lần trong tuần.
- Claim mốc cao không tự động claim mốc thấp nếu user chưa nhận, nhưng UI nên hiển thị tất cả mốc đang có thể claim.
- Weekly Chest không yêu cầu nhiệm vụ riêng, chỉ dựa trên Activity Stamp.
- Weekly Chest chưa claim sẽ hết hạn khi sang tuần mới.
- Hệ thống không auto-claim Weekly Chest khi hết tuần.

---

## 11. Acceptance criteria

### 11.1. Cấp nhiệm vụ

- Khi Learner mở Daily Mission lần đầu trong ngày, hệ thống tạo đúng 5 Daily Missions bắt buộc.
- Hệ thống không cấp nhiệm vụ community/study group.
- Hệ thống không cấp mission SRS nếu Review Queue snapshot rỗng.
- Hệ thống không cấp mission shop/booster nếu user chưa mở khóa shop/booster.
- Nếu đủ template hợp lệ, user không nhận cùng một mission code trong 2 ngày liên tiếp.

### 11.2. Tracking

- Khi user scan và lưu từ hợp lệ, mission scan/vocabulary liên quan tăng tiến độ.
- Khi user làm quiz đạt điều kiện, mission quiz liên quan tăng tiến độ.
- Khi user hoàn thành review item trong SRS snapshot, mission SRS liên quan tăng tiến độ.
- Tiến độ không vượt quá target.
- Mission tự chuyển sang `COMPLETED` khi đạt target.

### 11.3. Claim reward

- User chỉ claim được mission ở trạng thái `COMPLETED`.
- Claim thành công cộng đúng Coin/XP/item đã cấu hình.
- Claim trùng hoặc request song song không cộng reward lần thứ hai.
- Khi 5/5 mission bắt buộc đã claim, Daily Chest chuyển sang trạng thái có thể claim.
- Mission đã completed nhưng chưa claim trước reset ngày chuyển sang `EXPIRED` và không cộng reward.

### 11.4. Weekly Milestone

- Claim Daily Chest tạo đúng 1 Activity Stamp cho ngày đó.
- User có 3/5/7 Activity Stamps trong tuần sẽ mở khóa đúng Weekly Chest tương ứng.
- Weekly Chest chỉ được claim một lần trong một chu kỳ tuần.

---

## 12. Chỉ số cần theo dõi

| Metric | Ý nghĩa |
| :--- | :--- |
| `daily_mission_open_rate` | Tỷ lệ Learner mở màn Daily Mission mỗi ngày |
| `mission_completion_rate_by_code` | Tỷ lệ hoàn thành theo từng template |
| `daily_chest_claim_rate` | Tỷ lệ Learner claim đủ 5 nhiệm vụ |
| `bonus_mission_completion_rate` | Mức hấp dẫn của Bonus Mission |
| `activity_stamp_3_5_7_rate` | Tỷ lệ đạt các mốc Weekly Chest |
| `reward_coin_issued_daily` | Tổng Coin phát hành từ Daily Mission mỗi ngày |
| `mission_abandon_rate_by_slot` | Slot nào thường bị bỏ dở |

---

## 13. Future Enhancement: phân bổ nhiệm vụ theo trạng thái người dùng

Nếu có thêm thời gian sau MVP, SnapVocab có thể nâng cấp Daily Mission bằng cơ chế chia nhiệm vụ theo `missionProfile`. Đây là hướng cá nhân hóa mission dựa trên trạng thái học tập gần đây của Learner, nhưng chưa bắt buộc trong MVP.

### 13.1. Ý tưởng chính

Hệ thống tính một `missionProfile` cho mỗi Learner vào đầu ngày, sau đó dùng profile này để điều chỉnh weight, target và eligibility của Mission Pool.

Quan trọng:

- Vẫn giữ cấu trúc **5 Daily Missions bắt buộc + tối đa 1 Bonus Mission**.
- Profile không làm thay đổi UI chính, chỉ ảnh hưởng cách hệ thống chọn nhiệm vụ.
- Profile được snapshot theo ngày để nhiệm vụ không đổi liên tục trong ngày.
- Nếu chưa đủ dữ liệu để phân loại, user dùng cấu hình mặc định.

### 13.2. Các profile gợi ý

| Mission Profile | Điều kiện gợi ý | Trọng tâm nhiệm vụ | Ví dụ ưu tiên |
| :--- | :--- | :--- | :--- |
| `NEW_LEARNER` | User mới, ít hơn 3 ngày hoạt động hoặc ít hơn 20 từ đã lưu | Hướng dẫn luồng chính, target thấp | Scan 1-3 từ, học 10 flashcards, hoàn thành 1 mini quiz |
| `RETURNING_LEARNER` | Quay lại sau nhiều ngày không hoạt động | Tái kích hoạt nhẹ, giảm áp lực | Scan 1 từ, học 5 flashcards, review vài từ dễ |
| `SCAN_HEAVY` | Scan/lưu từ nhiều nhưng ít ôn tập trong 7 ngày gần nhất | Chuyển từ thu thập sang học thật | Học flashcard, ôn từ đã scan hôm qua, quiz từ mới lưu |
| `REVIEW_BACKLOG` | Có nhiều SRS item đến hạn hoặc tỷ lệ quá hạn cao | Dọn backlog ôn tập | Review 10 từ đến hạn, clear SRS snapshot, học lại thẻ sai |
| `QUIZ_WEAK` | Điểm quiz trung bình thấp hoặc accuracy giảm | Cải thiện độ chính xác | Quiz >= 70/80%, học lại câu sai, correct streak 5 |
| `CONSISTENT_LEARNER` | Có streak ổn định và hoàn thành mission đều | Duy trì thói quen, tăng nhẹ thử thách | XP target cao hơn, clear SRS, perfect mini quiz |
| `POWER_LEARNER` | XP cao, học nhiều ngày liên tiếp, thường hoàn thành hết mission | Thêm thử thách tùy chọn | Bonus Mission khó hơn, perfect quiz, climb leaderboard cá nhân |

### 13.3. Rule ưu tiên

Nếu user khớp nhiều profile, hệ thống có thể ưu tiên theo thứ tự:

1. `RETURNING_LEARNER`
2. `NEW_LEARNER`
3. `REVIEW_BACKLOG`
4. `QUIZ_WEAK`
5. `SCAN_HEAVY`
6. `POWER_LEARNER`
7. `CONSISTENT_LEARNER`

### 13.4. Lý do để để sau MVP

- Cần đủ dữ liệu hành vi để phân loại chính xác.
- Cần test reward economy kỹ hơn vì target/reward có thể thay đổi theo profile.
- Cần dashboard theo dõi completion rate theo từng profile.
- MVP có thể chạy tốt với context-aware filtering đơn giản trước, rồi nâng cấp profile sau.

---

## 14. Quyết định MVP

- Mỗi ngày cấp **5 Daily Missions bắt buộc**.
- Có thể cấp **tối đa 1 Bonus Mission tùy chọn**.
- Daily Chest mở khi claim đủ 5/5 mission bắt buộc.
- Reset theo **Asia/Ho_Chi_Minh (GMT+7)**.
- Mission completed nhưng chưa claim trước reset sẽ **hết hạn và mất reward**.
- Không triển khai mission cộng đồng hoặc nhóm học tập.
- Weekly Milestone dùng Activity Stamp và Weekly Chest là scope chính thức.
- Không tạo nhiệm vụ tuần riêng.
- Phân bổ nhiệm vụ theo `missionProfile` là hướng nâng cấp sau MVP.
