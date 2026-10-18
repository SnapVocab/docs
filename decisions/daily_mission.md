# Đặc tả tính năng: Daily Mission

> **Trạng thái tài liệu:** ACTIVE / AUTHORITATIVE cho rule và contract của Daily Mission (SS-13). Authority theo domain: scope Product Owner (bên dưới) > file này cho rule Daily Mission; [specs.md](../spec/specs.md) FR-09 cho phạm vi chức năng; [database.md](../db/database.md) §4 cho schema và phải khớp §7 file này; [shop.md](./shop.md) cho Coin/Booster/item; [quiz.md](./quiz.md) cho quiz. Phân rã, SA, workbook, mobile/admin design là DERIVED.
>
> **Scope (2026-10-03, quyết định của Product Owner):** Chỉ có **nhiệm vụ ngày**. Mỗi ngày đúng **5 mission**; mỗi mission có progress riêng, được claim riêng và có reward riêng. **Không có** Daily Chest, Bonus Mission, Activity Stamp, Weekly Mission, Weekly Milestone/Weekly Chest, Monthly Mission/Reward, nhiệm vụ thành tựu, hay bất kỳ reward tổng hợp nào khi hoàn thành đủ 5 mission.
>
> **Readiness (2026-10-05):** **Đặc tả: đủ để triển khai · Production: chưa đạt.** Product Owner đã duyệt gói PD-DM-01 (P1–P7, L1–L2, P4(a)); PD-DM-03/04/05 được giải trong gói (§0). Gate này cho phép **bắt đầu implementation**, chưa phải sẵn sàng release. Còn mở nhưng không chặn việc xây backend: PD-DM-02, PD-DM-06, PD-LB-01; COPY-DM-01 đã chốt 2026-10-05 (seed tự chạy); DEP-DM-01 (Learning/mobile) phải xong **trước nghiệm thu luồng mobile** theo P4(a). Các mục PD-DM-07/08/09, REL-DM-01, PLAT-01, CLEAN-DM-01 và checklist §0.1 là **[Proposed]**, chờ PO duyệt. **Hiện trạng code (2026-10-05):** backend đã có (package `vn.ptit.snapvocab.mission`: entity, `MissionTracker`, `DailyMissionService`, API §8.1–§8.2, hook 5 producer, seed §5.1 đủ 7 template, migration SQL chưa áp production) kèm test trên MySQL local; mobile MH-GAME-02 và Quick Lookup điền main.translation đã có trong SnapVocab, có test tự động nhưng chưa nghiệm thu trên thiết bị; Admin Missions mock đã gỡ route/menu và liên kết khỏi bundle release (source prototype được giữ). Xem [kế hoạch nghiệm thu staging](../reviews/daily_mission_staging_acceptance.md); chưa nghiệm thu staging thật. Xem [báo cáo rà độc lập](../reviews/daily_mission_backend_mobile_2026-10-05.md). Backend test pass **không** có nghĩa là sẵn sàng production (§0.1).

Quy ước nhãn trong tài liệu:

- **[Authoritative]**: rule sản phẩm có nguồn gốc (scope 2026-10-03, gói PD-DM-01 duyệt 2026-10-04, nội dung gốc của file này, hoặc file được trích dẫn).
- **[Derived]**: contract kỹ thuật suy ra trực tiếp từ rule authoritative + code hiện có; không thêm hành vi sản phẩm mới.
- **[Unresolved]**: chưa có nguồn authoritative; ghi trong Decision Register, không tự đặt giá trị.
- **[Proposed]**: đề xuất từ review, **chưa** được PO duyệt; không thay đổi rule [Authoritative]. Chỉ khi PO duyệt rõ ràng mới chuyển sang Resolved và cập nhật các section bị ảnh hưởng.

---

## 0. Decision register

| ID | Câu hỏi | Trạng thái | Phân loại | Nội dung / ảnh hưởng |
| :--- | :--- | :--- | :--- | :--- |
| PD-DM-01 | Catalog nào luôn cấp được đúng 5 mission khả thi? | **Resolved 2026-10-04** | — | Gói duyệt: P1 catalog §5.1; P2 "quiz ngắn"/"mini quiz" = `requestedCount = 5`; P3 từ được tính §5.5; P4(a) content tự tạo + Quick Lookup điền bản dịch (DEP-DM-01); P5 multi-progress §6.3; P6 thiếu pool §4.4; P7 ranh giới ngày §3; L1–L2 giới hạn được chấp nhận §10. |
| PD-DM-03 | Eligibility xét trước hay sau hành động kích hoạt việc cấp? | **Resolved 2026-10-04** (trong gói) | — | **Trước** mutation của trigger đầu tiên trong ngày (§4.1). |
| PD-DM-04 | Hành động trước 00:00 nhưng được xử lý sau 00:00? | **Resolved 2026-10-04** (trong gói, P7) | — | Progress đồng bộ trong transaction của hành động (§6); ngày = ngày của `actionInstant` (§3.2); transaction commit sau 00:00 để lại credit ngày cũ, không claim được. |
| PD-DM-05 | Chống lặp mission giữa 2 ngày là cứng hay mềm? | **Resolved 2026-10-04** (trong gói) | — | Ưu tiên mềm theo từng slot (§4.2); cứng sẽ phá invariant đúng 5 khi pool nhỏ. |
| PD-DM-06 | Release đầu có Admin CRUD template (F-GAME-09) hay chỉ seed? | **[Unresolved]** | NON-BLOCKING cho runtime | Runtime chạy với seed (§7.7). Nếu Admin vào release thì triển khai đủ §8.3. Defer Admin phải được ghi thành quyết định. **[Proposed 2026-10-05]** Release đầu chỉ dùng seed; gỡ route/menu Admin Missions mock khỏi bản release; mọi sửa catalog qua SQL theo REL-DM-01 và không sửa snapshot đã cấp (§7.2). |
| PD-DM-02 | Giới hạn tổng reward Daily Mission/ngày (economy cap)? | **[Unresolved]** | NON-BLOCKING | Không triển khai validation cap. Tổng thực tế của catalog §5.1: 80–85 Coin, 150–160 XP/ngày. **[Proposed 2026-10-05]** Không thêm cap riêng cho catalog MVP: mức tối đa từ Daily Mission là **85 Coin / 160 XP mỗi người/ngày**, không gồm thưởng của chính hoạt động học. Đánh giá lại khi reward đổi qua SQL hoặc Admin. |
| PD-LB-01 | XP `MISSION_REWARD` có tính vào Weekly XP của Leaderboard không? | **[Unresolved]** | NON-BLOCKING cho Daily Mission; thuộc Leaderboard | Daily Mission luôn ghi XP qua `XpService.award(MISSION_REWARD)` và cộng `users.exp`. Projection Leaderboard lọc theo `experience_logs.source_type`. Đây là nơi **duy nhất** ghi câu hỏi này. |
| COPY-DM-01 | Tên UI của `ADD_5_WORDS`, `FINISH_2_SHORT_QUIZZES`, `PERFECT_MINI_QUIZ` | **[Resolved 2026-10-05]** | — | Mission Pool gốc không có tên UI cho 3 template này. **Chốt 2026-10-05:** `ADD_5_WORDS` = "Góp Nhặt Từ Mới", `FINISH_2_SHORT_QUIZZES` = "Hai Bài Quiz Ngắn", `PERFECT_MINI_QUIZ` = "Quiz Ngắn Hoàn Hảo"; mô tả §5.1 giữ nguyên (ghi rõ 5 từ, 5 câu, 100%). |
| DEP-DM-01 | Quick Lookup trên mobile điền `main.translation` | Dependency (Learning / mobile) | Bắt buộc theo P4(a) | **Đã có trong code (2026-10-05):** `use-add-word-form.ts` điền `main.translation` từ `translations` khi field còn trống (attribute optional trong `DEFAULT_ENGLISH`); chưa nghiệm thu trên thiết bị. Thiếu bản dịch ⇒ từ không dùng được cho quiz (§5.5). Khi chưa có tự điền, Learner vẫn nhập tay được bản dịch: dependency giảm thao tác, không phải điều kiện khả thi tuyệt đối. |
| PD-DM-07 | Đa dạng catalog: 4 template cốt lõi xuất hiện mỗi ngày | **[Proposed 2026-10-05]** | NON-BLOCKING cho code; chờ PO | MVP giữ 4 template cốt lõi cố định (§2.2, §5.1); không thêm biến thể chỉ để tăng số mẫu. Bộ 5 chỉ đổi được khi có **> 4** template cốt lõi `NONE` active; **≥ 6 là mục tiêu đề xuất**, không tự bảo đảm đa dạng (eligibility và thứ tự chọn slot §4.2 vẫn quyết định). Template quiz 10/20 câu với `NONE` làm tăng đáng kể công chuẩn bị ([quiz.md](./quiz.md): `QUIZ_INSUFFICIENT_ELIGIBLE_ITEMS`); muốn bảo đảm làm được ngay khi cấp thì cần eligibility rule kiểm tra content. Khi PO duyệt: ghi giới hạn L3 vào §10. |
| PD-DM-08 | Multi-progress với mission thêm từ | **[Proposed 2026-10-05]** | NON-BLOCKING | Giữ multi-progress cho quiz (P5, economy §5.1 đã duyệt). Ghi ngoại lệ vào §6.3: word credit trọn đời (§5.5) ⇒ một từ chỉ tăng **một** mission `TOPIC_ITEM_CREATED` (mission đầu tiên theo `slot` trong §6.2). Catalog §5.1 chỉ có một template thêm từ nên hiện chưa gây xung đột. |
| PD-DM-09 | Mất thưởng khi chưa claim trước 00:00 | **[Proposed 2026-10-05]** | NON-BLOCKING | Giữ hạn claim (§3.3, §15) và cảnh báo trong app (§2.3). Thêm metric `mission_completed_unclaimed_expired_rate_by_code` vào §12. **Chưa chốt** tự cộng thưởng: metric cao là căn cứ mở lại quyết định, vì tự cộng thay đổi rule hết hạn và contract claim (§8.2). |
| REL-DM-01 | Quản lý schema production | **[Proposed 2026-10-05]** | BLOCKING production | Migration SQL có version cho 5 bảng §7.1–§7.6, gồm CHECK. Production chuyển sang `ddl-auto: validate` chỉ sau khi đã kiểm tra **toàn bộ schema hiện hữu** khớp entity; CHECK kiểm tra riêng (`validate` không chứng minh đủ ràng buộc). Mọi migration sửa `mission_templates` phải chạy kiểm tra invariant §4.2 trong cùng transaction và **rollback** nếu vi phạm, không dựa vào kiểm tra lúc khởi động (§7.7). **Hiện trạng 2026-10-05:** đã có `migration_daily_mission_v1.sql` (5 bảng + `dm_assert_catalog_invariant`) và `migration_daily_mission_catalog_template.sql`; `MissionMigrationScriptTest` xác minh trên MySQL 8.0 local: script idempotent; Hibernate `ddl-auto: validate` thật pass trên schema do script tạo cho 5 bảng mission (và bắt được drift khi xóa cột); cột/UNIQUE/CHECK khớp bảng Hibernate sinh (validate không kiểm CHECK nên CHECK được kiểm riêng bằng row sai); migration catalog vi phạm invariant bị rollback (SQLSTATE 45000); cả hai script chạy được bằng `mysql` client 8.0 — công cụ theo convention `migration_*.sql` hiện tại của repo. Khi client dừng giữa chừng, procedure tạm `dm_tmp_catalog_change` còn lại; lần chạy sau tự `DROP` trước khi tạo. **Còn thiếu:** chưa áp vào staging/production; chưa chọn công cụ migration có version (Flyway/Liquibase hay giữ chạy tay); chưa kiểm tra schema hiện hữu của các bảng khác (production vẫn `ddl-auto: update`, ví dụ `shop_items.purchasable` do script cũ khai báo `BOOLEAN` trong khi Hibernate sinh `bit`); chưa đổi production sang `validate`. |
| PLAT-01 | `GlobalExceptionHandler` trả `ex.getMessage()` | **[Proposed 2026-10-05]** | BLOCKING production (ticket nền tảng) | Tách ticket nền tảng: lỗi không lường trước trả mã cố định `INTERNAL_ERROR` (§8). Tách ticket **không** loại việc này khỏi điều kiện production. **Hiện trạng 2026-10-05:** đã sửa trong code (`handleRuntime` trả `INTERNAL_ERROR`, log chi tiết phía server); các handler khác vẫn trả `ex.getMessage()` như cũ (ngoài phạm vi). |
| CLEAN-DM-01 | Mô hình cũ chest/stamp | **[Proposed 2026-10-05]** | BLOCKING UI release | Bỏ chest/stamp khỏi UI release. Chỉ xóa `DAILY_CHEST`/`WEEKLY_CHEST` khỏi `XpSourceType`/`CoinSourceType` khi DB không còn row **và** code không còn tham chiếu (§7.5); còn lịch sử thì giữ enum để đọc được. Không xóa prototype `snapVocab-frontend` trong phạm vi này. |

### 0.1. Checklist chặn production [Proposed 2026-10-05 — chờ PO xác nhận]

| # | Nhóm | Điều kiện đạt |
| :--- | :--- | :--- |
| 1 | Backend + test tích hợp MySQL local | §4–§8 và AC §13. Test MySQL tối thiểu: claim đồng thời; rollback thưởng khi lỗi sau khi đã cộng; retry cùng key sau khi mất response; rollover 0 giờ (`actionInstant` 23:59:59, claim sau 00:00); hai trigger song song chỉ tạo một bộ 5 (§13.1); retry add-item tạo hai TopicItem cùng từ chỉ tăng progress 1 lần (§13.2). **Hiện trạng 2026-10-05:** code + test đã có (`DailyMissionIntegrationTest`, `DailyMissionHttpEndToEndTest`, `MissionMigrationScriptTest`, `MissionTrackerTest`), pass trên MySQL local. |
| 2 | Mobile MH-GAME-02 | §9 trên app `SnapVocab/` |
| 3 | Seed §7.7 (COPY-DM-01 đã chốt 2026-10-05) | §7.7 |
| 4 | DEP-DM-01 | §9 (Quick Lookup) |
| 5 | REL-DM-01 | Migration xác minh trên schema thực tế |
| 6 | PLAT-01 | Handler lỗi chung trả `INTERNAL_ERROR` |
| 7 | Gỡ Admin Missions mock khỏi bản release | PD-DM-06 (Proposed) |
| 8 | PO xác nhận PD-DM-07/08/09 | Decision register §0 |

---

## 1. Phạm vi

### 1.1. Mục tiêu

- Tạo lý do rõ ràng để Learner quay lại học mỗi ngày.
- Khuyến khích luồng học chính của SnapVocab: lưu từ, làm quiz (MVP); scan, flashcard, ôn SRS khi capability tương ứng tồn tại (§5.3).
- Chu kỳ ngày đơn giản: làm nhiệm vụ → nhận thưởng của chính nhiệm vụ đó.
- Reward economy ổn định, đo lường được, khó gian lận.

### 1.2. Ngoài phạm vi

- Reward tổng hợp: Daily Chest, Activity Stamp, Weekly Milestone/Weekly Chest, Monthly Reward, reward khi đủ 5/5.
- Bonus Mission, Weekly Mission, Monthly Mission, nhiệm vụ thành tựu (achievement).
- Nhiệm vụ cộng đồng/nhóm học tập (Study Group, bình luận, bảng xếp hạng nhóm, mời bạn).
- Badges là subsystem riêng (F-GAME-05), **không** phải reward của Daily Mission.
- Leaderboard Weekly XP là subsystem xếp hạng riêng (F-GAME-06), **không** phát reward tuần.
- Notification khi hoàn thành mission: không có trong MVP; không có domain event "mission completed".

---

## 2. Trải nghiệm người dùng

### 2.1. Cấu trúc nhiệm vụ mỗi ngày [Authoritative]

Mỗi Learner nhận đúng **5 Daily Missions** mỗi ngày. Mỗi mission được claim độc lập và chỉ trả reward của chính nó.

### 2.2. Cân bằng danh mục [Authoritative]

| Slot | Nhóm nhiệm vụ | `category` hợp lệ |
| :--- | :--- | :--- |
| 1 | Scan & Capture | `SCAN` |
| 2 | Vocabulary Building | `VOCABULARY` |
| 3 | Flashcard / SRS | `FLASHCARD`, `SRS` |
| 4 | Quiz / Accuracy | `QUIZ` |
| 5 | Retention / Personal Gamification | `RETENTION`, `SHOP` |

Nếu một nhóm không có nhiệm vụ phù hợp, hệ thống chọn nhiệm vụ thay thế từ **nhóm học tập cốt lõi khác**. [Derived] "Nhóm học tập cốt lõi" = slot 1–4 (`SCAN`, `VOCABULARY`, `FLASHCARD`, `SRS`, `QUIZ`); slot 5 không phải nhóm học tập. Với catalog §5.1, slot 1 và 3 luôn dùng fallback ⇒ đây là **catalog kỹ thuật nặng về quiz** (2 họ producer: lưu từ, quiz), không phải đa dạng hoạt động thật.

### 2.3. Trạng thái hiển thị [Authoritative]

Mỗi mission hiển thị: tên, mô tả ngắn, tiến độ (`2/5`), reward Coin/XP (và item nếu có), trạng thái `Đang làm` / `Hoàn thành` / `Đã nhận thưởng`, nút `Nhận thưởng` khi `COMPLETED`. UI nhấn mạnh rằng mission hoàn thành nhưng chưa claim trước reset sẽ mất thưởng.

---

## 3. Chu kỳ ngày

### 3.1. Timezone [Authoritative]

Timezone cố định **Asia/Ho_Chi_Minh (GMT+7)** — trùng `ProgressService.ZONE` hiện có. Ngày nhiệm vụ: `00:00:00`–`23:59:59.999999` GMT+7. Server clock là nguồn duy nhất; client không gửi ngày hay timestamp.

### 3.2. `actionInstant` và ngày của một hành động [Authoritative P7 + Derived]

Mỗi hành động của **producer** (quiz complete, add-item, purchase, activate — §6.1) đi theo đúng thứ tự:

```text
1. Lấy đủ các lock cần thiết (bảng dưới)
2. Validation pass
3. actionInstant = Economy.now()                    -- chụp MỘT lần
4. date = LocalDate.ofInstant(actionInstant, Asia/Ho_Chi_Minh)
5. ensureAssigned(userId, date)                     -- §4, trước mutation (pre-action)
6. Mutation; mọi timestamp domain chính dùng actionInstant
7. MissionProgress.apply(...)                       -- §6, cùng transaction
```

| Hành động | Lock (theo thứ tự) | Timestamp domain dùng `actionInstant` | Thay đổi so với code hiện tại |
| :--- | :--- | :--- | :--- |
| Quiz complete | quiz row → `users` | `quiz.completed_at`; `ProgressService.record(…, actionInstant)` → `learning_events.created_at`, `learning_date` | Lấy lock `users` trước validation `unresolved`; `QuizService.now()` tại điểm complete thay bằng `actionInstant`; `record` nhận thêm tham số |
| Add-item (`POST /api/topics/{topicId}/items`) | kiểm tra quyền topic → `users` | Hook §6 | Add-item bắt đầu khóa `users` (tuần tự hóa add-item theo user) |
| Purchase / Activate | `users` | `booster_activations.started_at` | Chụp sau `blockReason`/eligibility, không phải đầu `doPurchase`/`doActivate` |

Không thuộc flow trên:

- **GET** (§8.1): lock `users` → `today = LocalDate(Economy.now())` → `ensureAssigned(user, today)` → đọc. Chỉ cấp/đọc, không mutation khác, không progress.
- **Claim** (§8.2): không gọi `ensureAssigned` hay `MissionProgress.apply`; lock `users` → mission, hạn claim so bằng `today` tính sau lock.

`experience_logs.created_at` và `coin_transactions.created_at` vẫn do `XpService`/`WalletService` tự đặt; Daily Mission không đọc hai cột này.

Hệ quả [Authoritative P7]: ngày của credit = ngày của `actionInstant`, kể cả khi transaction commit sau 00:00 (do chờ lock hoặc chạy lâu). Progress đồng bộ loại bỏ độ trễ queue, **không** loại bỏ thời gian transaction. Không có grace window.

### 3.3. Reset và hết hạn [Authoritative + Derived]

- Không cộng dồn: mission ngày trước không nhận progress của hành động thuộc ngày sau. [Authoritative]
- Mission `IN_PROGRESS` hoặc `COMPLETED` mà `mission_date < today` là `EXPIRED`, **không nhận thưởng**. [Authoritative]
- [Derived] `EXPIRED` là trạng thái **suy ra**, không lưu: `mission_date < today ∧ status ≠ CLAIMED`. Không scheduler, không ghi DB khi hết hạn.
- [Authoritative P7] Hành động có `actionInstant` trước 00:00 mà transaction commit sau 00:00: progress (và nếu chưa có thì cả bộ mission) thuộc ngày cũ, chỉ là lịch sử, không claim được.
- [Derived] **Hạn chót claim** xét bằng `today` tính **sau khi** giữ lock (§8.2 bước 5): request gửi 23:59:59 nhưng giữ lock lúc 00:00:01 ⇒ `MISSION_EXPIRED`.

---

## 4. Cấp nhiệm vụ (assignment)

### 4.1. Thời điểm cấp và eligibility [Authoritative PD-DM-03 + Derived]

`ensureAssigned(userId, date)` chạy trong: (a) `GET /api/me/daily-missions` (§8.1); (b) bước 5 của §3.2 ở mọi producer §6.1 — tùy cái nào tới trước trong ngày. Claim không gọi `ensureAssigned`.

- **Eligibility** được xét trên state **ngay trước mutation** của trigger đầu tiên trong ngày, một lần; sau đó bộ 5 không đổi. Nếu state đổi giữa lúc mở màn và lúc hành động, bộ mission phản ánh state tại trigger tới trước — đây là semantics được chấp nhận, không phải "giống hệt mọi thứ tự".
- **Transaction:** `@Transactional(propagation = MANDATORY)`, luôn join transaction của caller; caller đã giữ lock `users`.

### 4.2. Thuật toán [Authoritative rules, Derived ordering]

```text
ensureAssigned(userId, date) -> EXISTING | ASSIGNED | POOL_INSUFFICIENT:
  (caller đã giữ lock users)
  if exists user_daily_missions(user_id, mission_date = date) → return EXISTING
  eligible  = mission_templates WHERE is_active AND eligibility(rule, user state hiện tại) ORDER BY id
  yesterday = codes của user_daily_missions(user_id, date - 1)
  chosen = []                                          -- chọn HOÀN TOÀN trong bộ nhớ
  for slot in 1..5:
    pool = eligible ∩ categories(slot) − chosen
    if pool = ∅: pool = eligible ∩ CORE_CATEGORIES − chosen          -- §2.2 fallback
    if pool = ∅: log ERROR + metric; return POOL_INSUFFICIENT         -- chưa ghi gì
    fresh = pool − yesterday; if fresh ≠ ∅: pool = fresh              -- ưu tiên mềm (PD-DM-05)
    chosen += weightedRandom(pool, weight)
  INSERT 5 user_daily_missions (snapshot §7.2, IN_PROGRESS, current_progress 0)
  return ASSIGNED
```

- Lock `users` tuần tự hóa mọi caller của cùng user ⇒ một bộ duy nhất. Unique `(user_id, mission_date, slot)` và `(user_id, mission_date, mission_template_id)` là lớp chặn cuối; vi phạm là exception (§4.4).
- **Chống lặp [Authoritative PD-DM-05]:** best-effort trong pool của từng slot. Với catalog §5.1, 4 template cốt lõi xuất hiện mỗi ngày; chỉ slot 5 thay đổi.
- **Random:** `ORDER BY id` cố định thứ tự ứng viên; random không seed. Sau khi commit, mọi lần đọc trả cùng một bộ.
- **Invariant catalog [Derived]:** với user chỉ đủ điều kiện template `NONE`, phải có ≥ 4 template cốt lõi `NONE` active **và** (≥ 1 template slot 5 `NONE` active **hoặc** ≥ 5 template cốt lõi `NONE` active). Kiểm tra lúc khởi động (§7.7) và ở mọi thao tác Admin (§8.3). Catalog §5.1 thỏa: 4 cốt lõi + `KEEP_DAILY_STREAK`.

### 4.3. Eligibility rules [Authoritative + Derived]

| `eligibility_rule` | Điều kiện được cấp | Nguồn |
| :--- | :--- | :--- |
| `NONE` | Luôn hợp lệ. Khả thi theo L2 (§10): Learner tự chuẩn bị content | Gói PD-DM-01 |
| `HAS_IDLE_XP_BOOSTER` | Sở hữu XP Booster `quantity ≥ 1` **và** không có booster active | AC §13.1, [shop.md](./shop.md) S7 |
| `HAS_PURCHASABLE_ITEM` | Có ít nhất một item mà Learner mua được ngay: `ShopItem.isAvailableAt(now)` và `ShopService.blockReason(item, owned, coin) = null` | §10 mục 4 (Fairness); dùng lại logic Shop |

`eligibility_rule` là enum developer-controlled; rule cho template Deferred chỉ thêm khi template tương ứng được kích hoạt.

### 4.4. Thất bại khi cấp [Authoritative P6 + Derived]

`POOL_INSUFFICIENT` là **kết quả nghiệp vụ** (giá trị trả về, không phải exception, chưa ghi gì); mọi lỗi khác là **exception kỹ thuật**.

| Kết quả | Caller là hành động (§6.1) | Caller là GET |
| :--- | :--- | :--- |
| `POOL_INSUFFICIENT` (kết quả nghiệp vụ) | Bỏ qua progress; **hành động tiếp tục và commit**; credit mission của hành động đó **mất** (P6). Log ERROR + metric `mission_pool_insufficient_total`. | 503 `MISSION_POOL_INSUFFICIENT` |
| Exception kỹ thuật (SQL, vi phạm UNIQUE/FK/CHECK, bug) | **Lan ra ngoài, rollback cả hành động.** Không `catch` để chạy tiếp trong transaction đã lỗi (transaction có thể đã rollback-only hoặc chứa assignment dở). | như cột bên |

HTTP của exception kỹ thuật do handler tương ứng quyết định ([Derived], giữ mapping hiện có của `GlobalExceptionHandler`):

- `DataIntegrityViolationException` → **409 `CONCURRENT_MODIFICATION`**.
- `RuntimeException` không lường trước khác → **500 `INTERNAL_ERROR`** (mã cố định; chi tiết chỉ ghi log phía server).
- Purchase, activate (code hiện có) và claim (§8.2) chạy trong `TransactionTemplate` và **chạy lại một lần** khi gặp `DataIntegrityViolationException`: transaction lỗi đã rollback xong, lần chạy lại là transaction mới đánh giá lại từ state đã commit. Lỗi ở lần chạy lại lan ra ngoài theo mapping trên. Các producer khác (quiz complete, add-item) và GET không chạy lại.

`POOL_INSUFFICIENT` không xảy ra khi catalog thỏa invariant §4.2; nó chỉ phản ánh cấu hình sai.

---

## 5. Mission Catalog

### 5.1. Active MVP Catalog [Authoritative — gói PD-DM-01, P1]

Reward giữ nguyên giá trị Mission Pool gốc. `weight` mặc định 1 cho mọi template (chọn đều); đổi qua seed/Admin.

| Code | Tên UI | Mô tả | Category | `trigger_event` | Bộ lọc | Target | Reward | `eligibility_rule` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `ADD_5_WORDS` | Góp Nhặt Từ Mới | Thêm 5 từ mới có nghĩa | `VOCABULARY` | `TOPIC_ITEM_CREATED` | Từ dùng được cho quiz, chưa từng được tính (§5.5) | 5 | 15 Coin, 25 XP | `NONE` |
| `QUIZ_SCORE_80` | Trí Nhớ Siêu Phàm | Hoàn thành 1 quiz với độ chính xác ≥ 80% | `QUIZ` | `QUIZ_COMPLETED` | `min_quiz_accuracy_percent = 80` | 1 | 20 Coin, 40 XP | `NONE` |
| `FINISH_2_SHORT_QUIZZES` | Hai Bài Quiz Ngắn | Hoàn thành 2 quiz 5 câu | `QUIZ` | `QUIZ_COMPLETED` | `quiz_requested_count = 5` | 2 | 15 Coin, 30 XP | `NONE` |
| `PERFECT_MINI_QUIZ` | Quiz Ngắn Hoàn Hảo | Đạt 100% một quiz 5 câu | `QUIZ` | `QUIZ_COMPLETED` | `quiz_requested_count = 5`, `min_quiz_accuracy_percent = 100` | 1 | 25 Coin, 45 XP | `NONE` |
| `KEEP_DAILY_STREAK` | Giữ Lửa Mỗi Ngày | Có ít nhất 1 hoạt động học trong ngày | `RETENTION` | `LEARNING_ACTIVITY_RECORDED` | — | 1 | 10 Coin, 20 XP | `NONE` |
| `USE_XP_BOOSTER` | Tăng Tốc Học Tập | Kích hoạt 1 XP Booster | `SHOP` | `BOOSTER_USED` | — | 1 | 5 Coin, 10 XP | `HAS_IDLE_XP_BOOSTER` |
| `BUY_SHOP_ITEM` | Ghé Thăm Cửa Hàng | Mua 1 vật phẩm trong Shop | `SHOP` | `SHOP_ITEM_PURCHASED` | — | 1 | 5 Coin, 10 XP | `HAS_PURCHASABLE_ITEM` |

- **Bộ mỗi ngày:** slot 1–4 = 4 template cốt lõi (`ADD_5_WORDS` + 3 quiz) theo thứ tự fallback; slot 5 = random có trọng số trong {`KEEP_DAILY_STREAK`, và `USE_XP_BOOSTER`/`BUY_SHOP_ITEM` nếu đủ điều kiện}.
- **Economy (đã duyệt):** trọn bộ **85 Coin / 160 XP** (slot 5 là `KEEP_DAILY_STREAK`) hoặc **80 Coin / 150 XP** (slot 5 là `USE_XP_BOOSTER`/`BUY_SHOP_ITEM`). Một quiz 5 câu đạt 100% hoàn thành `QUIZ_SCORE_80` + `PERFECT_MINI_QUIZ` (45 Coin / 85 XP), cộng `KEEP_DAILY_STREAK` nếu slot 5 là template này (tổng 55 Coin / 105 XP), và đưa `FINISH_2_SHORT_QUIZZES` lên 1/2.
- **"Quiz ngắn" / "mini quiz" [P2]** = quiz có `requestedCount = 5` ([quiz.md](./quiz.md) D14: `{5, 10, 20}`).
- **Accuracy:** `correctCount / totalUnits` (unit đã chấm; Matching = cặp), so bằng số nguyên `correctCount * 100 >= totalUnits * min_quiz_accuracy_percent`. Mỗi quiz complete hợp lệ +1; quiz mới trên cùng content có `quizId` mới và được tính. Không có khái niệm "practice".
- **`KEEP_DAILY_STREAK`:** hiện chỉ quiz ghi `learning_events`; khi luồng học khác ghi `learning_events`, mission tự đếm. Tên "streak" không yêu cầu chuỗi nhiều ngày.

### 5.2. Audit toàn bộ Mission Pool gốc

| Template | Capability hiện có | Producer | MVP status | Lý do |
| :--- | :--- | :--- | :--- | :--- |
| `SCAN_SAVE_3_WORDS` | Scan trả kết quả; lưu từ qua `POST /api/topics/{topicId}/items` | Không | Deferred | Lưu từ không mang nguồn scan (không có `source`, không liên kết `scan_requests`) |
| `SCAN_SAVE_5_WORDS` | như trên | Không | Deferred | như trên |
| `SCAN_FROM_PARAGRAPH` | Không (AI là object detection Florence-2, không OCR đoạn văn) | Không | Deferred | Thiếu capability OCR paragraph |
| `SAVE_WORD_FROM_IMAGE` | như `SCAN_SAVE_*` | Không | Deferred | Không phân biệt từ lưu từ ảnh |
| `EDIT_SCANNED_WORD` | Không có API sửa TopicItem | Không | Deferred | Thiếu capability + producer |
| `ADD_5_WORDS` | `POST /api/topics/{topicId}/items` | Hook §6 (cần thêm) | **Active** | "Từ trùng" định nghĩa bằng word credit trọn đời (§5.5) |
| `TAG_3_WORDS` | Không có tính năng tag | Không | Deferred | Thiếu capability tagging |
| `REVIEW_YESTERDAY_WORDS` | Không định nghĩa "xem lại" | Không | Deferred | Thiếu định nghĩa hành động |
| `REACH_2_MASTERED` | Không có API review FSRS | Không | Deferred | Phụ thuộc SRS review API (M3) |
| `ADD_EXAMPLE_SENTENCE` | Không có API sửa TopicItem | Không | Deferred | Thiếu capability + producer |
| `STUDY_15_FLASHCARDS` | Không có API flashcard/review ở backend | Không | Deferred | Phụ thuộc M3 |
| `FINISH_2_FLASHCARD_SESSIONS` | Không có khái niệm session | Không | Deferred | Phụ thuộc M3 + định nghĩa session |
| `ANSWER_10_FLASHCARDS_CORRECT` | Không | Không | Deferred | Phụ thuộc M3 |
| `RETRY_5_WRONG_CARDS` | Không | Không | Deferred | Phụ thuộc M3 + định nghĩa "thẻ sai" |
| `CLEAR_SRS_SNAPSHOT` | Không có review queue API | Không | Deferred | Phụ thuộc M3 |
| `REVIEW_10_DUE_WORDS` | Không | Không | Deferred | Phụ thuộc M3 |
| `QUIZ_SCORE_80` | Quiz complete | Hook §6 | **Active** | Đủ dữ liệu |
| `FINISH_2_SHORT_QUIZZES` | Quiz complete (`requestedCount` có trên `Quiz`) | Hook §6 | **Active** | "Ngắn" = `requestedCount = 5` (P2) |
| `CORRECT_STREAK_5` | Quiz (có `position`) | — | Deferred | "Liên tiếp" không có nghĩa rõ với Matching (ghép theo thứ tự tùy ý) |
| `IMPROVE_QUIZ_SCORE` | Quiz | — | Deferred | Chưa định nghĩa "cùng dạng" và cách so điểm |
| `PERFECT_MINI_QUIZ` | Quiz complete | Hook §6 | **Active** | "Mini" = `requestedCount = 5` (P2) |
| `KEEP_DAILY_STREAK` | `ProgressService.record` | Hook §6 | **Active** | Capability có sẵn |
| `EARN_DAILY_XP_TARGET` | `UserSettings.dailyGoal` là số **từ**, không phải XP | Không | Deferred | Chưa có "XP mục tiêu ngày" |
| `CLIMB_PERSONAL_LEADERBOARD` | Chưa có Leaderboard | Không | Deferred | Thiếu leaderboard rule + event |
| `USE_XP_BOOSTER` | Activate booster | Hook §6 | **Active** | Đủ dữ liệu |
| `BUY_SHOP_ITEM` | Purchase | Hook §6 | **Active** | Đủ dữ liệu |

### 5.3. Deferred / Future Mission Ideas

Các template Deferred ở §5.2 **không** được seed vào `mission_templates`. Kích hoạt lại khi capability tương ứng tồn tại:

| Nhóm | Template | Điều kiện kích hoạt lại |
| :--- | :--- | :--- |
| Flashcard/SRS | `STUDY_15_FLASHCARDS`, `ANSWER_10_FLASHCARDS_CORRECT`, `REVIEW_10_DUE_WORDS`, `CLEAR_SRS_SNAPSHOT`, `RETRY_5_WRONG_CARDS`, `FINISH_2_FLASHCARD_SESSIONS`, `REACH_2_MASTERED` | SRS review API (M3) tồn tại; định nghĩa "thẻ sai"/"session" |
| Scan | `SCAN_SAVE_3_WORDS`, `SCAN_SAVE_5_WORDS`, `SAVE_WORD_FROM_IMAGE` | Lưu từ mang `source` (SCAN/DICT/…) |
| Scan nâng cao | `SCAN_FROM_PARAGRAPH` | Có OCR đoạn văn |
| Biên tập | `EDIT_SCANNED_WORD`, `ADD_EXAMPLE_SENTENCE`, `TAG_3_WORDS` | Có API sửa TopicItem / tagging |
| Quiz nâng cao | `CORRECT_STREAK_5`, `IMPROVE_QUIZ_SCORE` | Định nghĩa "liên tiếp" cho mọi mode, "cùng dạng" |
| Retention | `EARN_DAILY_XP_TARGET`, `REVIEW_YESTERDAY_WORDS`, `CLIMB_PERSONAL_LEADERBOARD` | Định nghĩa XP mục tiêu ngày; hành động "xem lại"; Leaderboard |

Reward đề xuất gốc của các template Deferred (Coin, XP): `SCAN_SAVE_3_WORDS` 10/20, `SCAN_SAVE_5_WORDS` 15/30, `SCAN_FROM_PARAGRAPH` 10/25, `SAVE_WORD_FROM_IMAGE` 5/10, `EDIT_SCANNED_WORD` 5/10, `TAG_3_WORDS` 5/15, `REVIEW_YESTERDAY_WORDS` 10/20, `REACH_2_MASTERED` 10/25, `ADD_EXAMPLE_SENTENCE` 5/15, `STUDY_15_FLASHCARDS` 15/30, `FINISH_2_FLASHCARD_SESSIONS` 15/30, `ANSWER_10_FLASHCARDS_CORRECT` 15/30, `RETRY_5_WRONG_CARDS` 20/35, `CLEAR_SRS_SNAPSHOT` 30/50, `REVIEW_10_DUE_WORDS` 20/40, `CORRECT_STREAK_5` 15/35, `IMPROVE_QUIZ_SCORE` 15/35, `EARN_DAILY_XP_TARGET` 15/30, `CLIMB_PERSONAL_LEADERBOARD` 20/40.

### 5.4. Tính khả thi ở zero-state [Authoritative P4(a), L2]

Learner mới (0 Coin, 0 item, chưa có từ) vẫn được cấp đủ 5 mission khả thi:

1. `ADD_5_WORDS`: tạo collection/topic USER (schema mặc định `DEFAULT_ENGLISH`), thêm từ kèm bản dịch.
2. 3 mission quiz: khả thi khi có **một topic** chứa ≥ 5 từ dùng được cho quiz với ≥ 4 bản dịch phân biệt sau chuẩn hóa (MCQ cần 4 đáp án, `QuizGenerator`: `NOT_ENOUGH_DISTINCT_ANSWERS`).
3. `KEEP_DAILY_STREAK`: hoàn thành cùng quiz đầu tiên.
4. `USE_XP_BOOSTER`/`BUY_SHOP_ITEM` không được cấp nếu không đủ điều kiện (§4.3).

Repo **không** seed content SYSTEM; catalog khả thi nhờ Learner tự chuẩn bị content (L2). Quick Lookup điền bản dịch (DEP-DM-01) giảm thao tác nhưng không bảo đảm bản dịch phân biệt hay nằm cùng một topic. `ADD_5_WORDS` hoàn thành **không** tự động mở khóa quiz.

### 5.5. Từ được tính cho `ADD_5_WORDS` [Authoritative P3 + Derived]

Một TopicItem tăng progress `ADD_5_WORDS` khi **tất cả** đúng:

1. Tạo qua `POST /api/topics/{topicId}/items`, trong topic thuộc collection `USER` của chính Learner. Fork/import/schema fork không gọi hook.
2. **Dùng được cho quiz:** cùng rule của quiz MCQ/MATCHING (`QuizContentResolver`, nguồn duy nhất): active template của topic bind đúng một attribute cho `TARGET_WORD` và `NATIVE_TRANSLATION` (`SINGLETON_ROLES`), và item có đúng một giá trị không rỗng sau trim cho mỗi role.
3. **Chưa từng được tính** cho Learner: `word_hash = SHA-256(UTF-8(normalize(TARGET_WORD)))`, với `normalize` = Unicode NFC → trim → gộp khoảng trắng liên tiếp thành một → `toLowerCase(Locale.ROOT)`; không tồn tại `user_word_credits(user_id, word_hash)` (§7.6).

- Word credit chỉ được ghi khi nó **thực sự tăng** một mission `ADD_5_WORDS` đang `IN_PROGRESS`. Không có mission đó hôm nay hoặc mission đã đủ 5/5 ⇒ không ghi, từ còn được tính ngày khác.
- Credit tồn tại suốt đời tài khoản: xóa topic/collection rồi thêm lại cùng từ không được tính lại.
- **Add-item không phải API idempotent:** retry có thể tạo hai TopicItem. Bảo đảm duy nhất là chúng không tăng mission hai lần.
- **L1:** P3 chỉ chặn việc dùng lại cùng một từ; không kiểm tra chất lượng từ/bản dịch (§10).

---

## 6. Progress contract — đồng bộ [Derived]

Progress mission được cập nhật **trong chính transaction của hành động** (cùng mẫu `ProgressEventListener` hiện có). Không có outbox, queue, sweeper hay listener `AFTER_COMMIT` cho Daily Mission. Hành động commit thì progress commit; một bên rollback thì cả hai rollback.

### 6.1. Producer

| `trigger_event` | Producer (nhánh lần đầu, không replay) | `source_key` | Facts cho bộ lọc |
| :--- | :--- | :--- | :--- |
| `QUIZ_COMPLETED` | `QuizService.complete`, sau khi set `COMPLETED` (replay `COMPLETED` return sớm) | `QUIZ_COMPLETED:{quizId}` | `totalUnits`, `correctCount`, `requestedCount` |
| `LEARNING_ACTIVITY_RECORDED` | `ProgressService.record`, chỉ khi insert `learning_events` mới | `LEARNING:{learningEventId}` | — |
| `TOPIC_ITEM_CREATED` | Service xử lý `POST /api/topics/{topicId}/items`, sau insert, chỉ collection `USER` của chính user | `TOPIC_ITEM:{topicItemId}` | `quizReady`, `wordHash` (§5.5) |
| `SHOP_ITEM_PURCHASED` | `ShopService.doPurchase`, nhánh không replay, **bên trong** `transaction.execute` | `SHOP_PURCHASE:{transactionId}` | — |
| `BOOSTER_USED` | `InventoryService.doActivate`, nhánh không replay, bên trong transaction | `BOOSTER_USED:{activationId}` | — |

- `ensureAssigned` chạy ở bước 5 §3.2 (trước mutation). `ProgressService.record` chạy trong transaction quiz nên không gọi `ensureAssigned` lần nữa.
- `ShopEvents` (publish sau commit) giữ nguyên; Daily Mission không dùng chúng.
- **Anti-spam [Authoritative]:** chỉ hành động backend xác nhận thành công mới gọi hook (quiz `CANCELLED`/`EXPIRED`, purchase/activate bị từ chối không gọi).

### 6.2. `MissionProgress.apply`

```text
apply(userId, date, trigger, sourceKey, facts, actionInstant):     -- trong transaction caller, đã giữ lock users
for m in user_daily_missions(userId, date)
      WHERE status = IN_PROGRESS AND trigger_event = trigger ORDER BY slot FOR UPDATE:
  if not filter(m, facts): continue
  if exists progress_log(m.id, sourceKey): continue                -- UNIQUE là lớp chặn cuối
  if trigger = TOPIC_ITEM_CREATED:
    if not facts.quizReady: continue
    if exists user_word_credits(userId, facts.wordHash): continue  -- UNIQUE(user_id, word_hash) là lớp chặn cuối
    INSERT user_word_credits(userId, facts.wordHash, m.id, actionInstant)
  INSERT user_daily_mission_progress_logs(m.id, sourceKey)
  m.current_progress = min(m.target_value, m.current_progress + 1)
  if m.current_progress = m.target_value: m.status = COMPLETED, m.completed_at = actionInstant
```

`filter`: `quiz_requested_count` (nếu có) = `facts.requestedCount`; `min_quiz_accuracy_percent` (nếu có) thỏa §5.1. Không có bộ mission cho `date` (vì `POOL_INSUFFICIENT`) ⇒ vòng lặp rỗng.

### 6.3. Bảo đảm

- **Multi-progress [Authoritative P5]:** mỗi mission đánh giá độc lập; một hành động có thể tăng nhiều mission. Mỗi mission ghi log riêng ⇒ không mission nào chặn mission khác.
  - **Ngoại lệ đang chạy trong code, chưa duyệt (PD-DM-08 [Proposed]):** với `TOPIC_ITEM_CREATED`, word credit trọn đời (§5.5, §6.2) được ghi ngay ở mission thêm từ đầu tiên khớp ⇒ cùng một từ không tăng mission thêm từ thứ hai trong cùng bộ (`MissionTracker.apply`). Câu "không mission nào chặn mission khác" ở trên chưa đúng cho trường hợp này cho tới khi PO chốt PD-DM-08. Catalog §5.1 chỉ có một template thêm từ; Admin cho phép tạo thêm nên editor ghi rõ rule này (2026-10-06).
- **Dedupe hai phạm vi:** `UNIQUE(user_daily_mission_id, source_key)` ⇒ mỗi hành động tăng mỗi mission tối đa 1 lần; `user_word_credits` `UNIQUE(user_id, word_hash)` ⇒ mỗi từ được tính tối đa 1 lần trong đời tài khoản. Không có tra cứu theo `(user_id, source_key)`.
- **Duplicate:** kiểm tra tồn tại dưới lock rồi mới insert. Không bắt `DataIntegrityViolationException` rồi chạy tiếp trong cùng transaction; vi phạm UNIQUE ⇒ rollback cả hành động, HTTP theo §4.4. Lần chạy lại một lần của purchase/activate/claim (§4.4) là transaction mới, không phải chạy tiếp.
- **Completion:** clamp + lọc `IN_PROGRESS` dưới row lock ⇒ một lần chuyển `COMPLETED`.
- **Đếm trùng:** mỗi template có đúng một `trigger_event`; quiz và learning record của cùng quiz không cùng tăng một mission.
- **Lock / deadlock:** mọi đường ghi lấy lock `users` trước khi chạm row mission/shop/credit (quiz: quiz row → `users`; không đường nào khóa `users` rồi tới quiz). Mọi row khác đều thuộc riêng một user ⇒ không có chu trình chờ.
- **Isolation [Derived]:** mọi transaction cấp/ghi mission chạy `READ_COMMITTED` (như Quiz/Shop hiện có; add-item được chuyển sang `READ_COMMITTED`). Dưới `REPEATABLE READ` của MySQL, snapshot tạo bởi lần đọc trước khi chờ lock `users` (VD đọc topic của add-item) sẽ không thấy bộ mission vừa được transaction khác commit ⇒ cấp bộ thứ hai và vi phạm UNIQUE.
- **Failure:** theo §4.4; exception trong `apply` rollback cả hành động (trade-off giống `XpService`/`ProgressService` trong transaction quiz).

---

## 7. Data model [Derived — theo quy ước [database.md](../db/database.md): `snake_case`, PK `BIGINT AUTO_INCREMENT`, `DATETIME(6)`]

Dev/test: schema do Hibernate sinh từ entity (`ddl-auto`); mọi CHECK dưới đây khai báo bằng `@Check` trên entity; cột enum lưu `VARCHAR` (`@JdbcTypeCode(VARCHAR)`, không dùng kiểu `ENUM` mặc định của Hibernate trên MySQL) và Hibernate thêm CHECK `IN (...)` cho miền giá trị; Hibernate chỉ sinh CHECK khi tạo bảng mới — mọi bảng mission đều mới. Môi trường quản lý schema thủ công: script có version `snap-vocab-backend/migration_daily_mission_v1.sql` theo convention `migration_*.sql` của repo (REL-DM-01, Proposed). DDL MySQL tự commit ngầm nên script schema **không** rollback được theo transaction; nó idempotent (`CREATE TABLE IF NOT EXISTS`). Chỉ DML sửa catalog mới chạy trong transaction có kiểm tra invariant (§7.7).

### 7.1. `mission_templates` (kế thừa `BaseTimeEntity`)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `code` | VARCHAR(64) | UNIQUE, NOT NULL, bất biến sau tạo | VD `QUIZ_SCORE_80` |
| `name` | VARCHAR(100) | NOT NULL | Tên UI |
| `description` | VARCHAR(255) | NOT NULL | Mô tả ngắn |
| `category` | VARCHAR(20) | NOT NULL | `SCAN`, `VOCABULARY`, `FLASHCARD`, `SRS`, `QUIZ`, `RETENTION`, `SHOP` |
| `trigger_event` | VARCHAR(40) | NOT NULL | `QUIZ_COMPLETED`, `LEARNING_ACTIVITY_RECORDED`, `TOPIC_ITEM_CREATED`, `SHOP_ITEM_PURCHASED`, `BOOSTER_USED` |
| `min_quiz_accuracy_percent` | INT | NULL, CHECK 1–100; chỉ khi `trigger_event = QUIZ_COMPLETED` | Bộ lọc accuracy |
| `quiz_requested_count` | INT | NULL, CHECK IN (5, 10, 20); chỉ khi `trigger_event = QUIZ_COMPLETED` | Bộ lọc kích thước quiz |
| `target_value` | INT | NOT NULL, CHECK `>= 1` | |
| `reward_coin` | INT | NOT NULL, CHECK `>= 0` | |
| `reward_xp` | INT | NOT NULL, CHECK `>= 0` | |
| `reward_item_code` | VARCHAR(64) | NULL | `shop_items.code`, 1 đơn vị ([shop.md](./shop.md) S17) |
| `reward_item_fallback_coin` | INT | NULL, CHECK `>= 0`; NOT NULL ⇔ có `reward_item_code` | Coin thay khi không cấp được item |
| `weight` | INT | NOT NULL, CHECK `>= 1`, DEFAULT 1 | Trọng số random |
| `eligibility_rule` | VARCHAR(40) | NOT NULL, DEFAULT `NONE` | §4.3 |
| `is_active` | BOOLEAN | NOT NULL, DEFAULT FALSE | |
| `version` | BIGINT | NOT NULL, DEFAULT 0 | Optimistic lock Admin |
| `created_at`, `updated_at` | DATETIME(6) | NOT NULL | |

CHECK: `reward_coin > 0 OR reward_xp > 0 OR reward_item_code IS NOT NULL` (mỗi mission có reward riêng — scope 2026-10-03).

### 7.2. `user_daily_missions` (kế thừa `BaseTimeEntity`)

Snapshot định nghĩa template lúc cấp: sửa template chỉ ảnh hưởng mission cấp sau; reward claim luôn bằng reward đã hiển thị.

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL | |
| `mission_template_id` | BIGINT | FK → `mission_templates(id)`, NOT NULL | |
| `mission_date` | DATE | NOT NULL | Ngày GMT+7 của `actionInstant` (§3.2) |
| `slot` | TINYINT | NOT NULL, CHECK 1–5 | |
| `code`, `name`, `description`, `category`, `trigger_event`, `min_quiz_accuracy_percent`, `quiz_requested_count`, `target_value`, `reward_coin`, `reward_xp`, `reward_item_code`, `reward_item_fallback_coin` | như §7.1 | Snapshot, bất biến | |
| `current_progress` | INT | NOT NULL, DEFAULT 0, CHECK `0 <= current_progress <= target_value` | |
| `status` | VARCHAR(20) | NOT NULL, CHECK IN (`IN_PROGRESS`, `COMPLETED`, `CLAIMED`) | `EXPIRED` không lưu (§3.3) |
| `completed_at` | DATETIME(6) | NULL | `actionInstant` của hành động làm đạt target |
| `claimed_at` | DATETIME(6) | NULL | |
| `created_at` (= assigned at), `updated_at` | DATETIME(6) | NOT NULL | |

Unique `(user_id, mission_date, slot)`; Unique `(user_id, mission_date, mission_template_id)`; index `(mission_template_id)` cho FK. Cột snapshot `code` **không** UNIQUE.

CHECK: `status = IN_PROGRESS` ⇔ `completed_at IS NULL`; `status IN (COMPLETED, CLAIMED)` ⇒ `current_progress = target_value`; `status = CLAIMED` ⇔ `claimed_at IS NOT NULL`; snapshot reward giữ cùng CHECK như §7.1.

State machine: `IN_PROGRESS → COMPLETED → CLAIMED`. Không có chiều ngược; không tính lại progress/target từ template live.

### 7.3. `user_daily_mission_progress_logs` (append-only, `BaseCreatedAtEntity`)

| Column | Type | Ràng buộc |
| :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT |
| `user_daily_mission_id` | BIGINT | FK → `user_daily_missions(id)`, NOT NULL |
| `source_key` | VARCHAR(191) | NOT NULL; dạng §6.1 (id số, độ dài bị chặn) |
| `created_at` | DATETIME(6) | NOT NULL |

Unique `(user_daily_mission_id, source_key)`. Chỉ dùng dedupe **theo từng mission**.

### 7.4. `user_daily_mission_claims` (append-only, `BaseCreatedAtEntity`)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_daily_mission_id` | BIGINT | FK, UNIQUE, NOT NULL | Một mission claim tối đa 1 lần |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL | |
| `idempotency_key` | VARCHAR(128) | NOT NULL | Header của request claim thành công |
| `reward_coin` | INT | NOT NULL | Coin đã cộng |
| `reward_xp` | INT | NOT NULL | XP đã cộng (base; mission XP không được booster nhân) |
| `reward_item_code` | VARCHAR(64) | NULL | |
| `granted_item_quantity` | INT | NOT NULL, DEFAULT 0 | Từ `RewardGrant.grantedQuantity` |
| `fallback_coin` | BIGINT | NOT NULL, DEFAULT 0 | Từ `RewardGrant.fallbackCoin` |
| `created_at` | DATETIME(6) | NOT NULL | |

Unique `(user_id, idempotency_key)`. CHECK mọi cột số `>= 0`. `user_id` phải bằng owner của mission (§8.2 bước 2). Bảng này là idempotency anchor **bắt buộc** cho item: `ShopRewardService` không tự dedupe item theo `eventKey` (chỉ fallback Coin có ledger UNIQUE).

### 7.5. Ledger (bảng có sẵn)

| Ledger | `source_type` | `event_key` | Khác |
| :--- | :--- | :--- | :--- |
| `experience_logs` | `MISSION_REWARD` (`boostable = false`) | `MISSION_REWARD:{userDailyMissionId}` | Bỏ qua khi `reward_xp = 0` (`XpService.award` yêu cầu `> 0`) |
| `coin_transactions` | `MISSION_REWARD` | `MISSION_REWARD:{userDailyMissionId}` | `reference_id = userDailyMissionId`, `description = name` snapshot; bỏ qua khi `reward_coin = 0` |
| `user_items` + fallback coin | `MISSION_REWARD` | `MISSION_REWARD:{userDailyMissionId}:ITEM` (+`:FALLBACK` do `ShopRewardService`) | Chỉ khi có `reward_item_code` |

`XpSourceType` và `CoinSourceType` chỉ cần `MISSION_REWARD` cho Daily Mission; `DAILY_CHEST`, `WEEKLY_CHEST` là legacy, đã xóa khỏi code 2026-10-18 ([shop.md](./shop.md) §13; DB local clone 0 row dùng các giá trị này).

### 7.6. `user_word_credits` (append-only)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | Theo quy ước PK của [database.md](../db/database.md) |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL | |
| `word_hash` | BINARY(32) | NOT NULL | SHA-256 của từ chuẩn hóa (§5.5); so sánh nhị phân, không phụ thuộc collation |
| `user_daily_mission_id` | BIGINT | FK → `user_daily_missions(id)`, NOT NULL | Mission đã tiêu credit |
| `created_at` | DATETIME(6) | NOT NULL | `actionInstant` |

`UNIQUE(user_id, word_hash)` — bất biến trọn đời do DB bảo vệ; index `(user_daily_mission_id)` cho FK.

### 7.7. Retention, FK và seed

- **Retention:** `user_daily_missions`, `user_daily_mission_progress_logs`, `user_daily_mission_claims`, `user_word_credits` không bao giờ bị purge theo tuổi. Mọi FK không cascade (RESTRICT). Khi xóa tài khoản: cùng policy với ledger của user.
- **Seed [Derived]:** `MissionCatalogSeeder` (chạy lúc khởi động, cùng mẫu `createDefaultEnglishSchemaIfNotFound` của `DataInitializer`): chèn từng template §5.1 theo `code` nếu chưa có, `is_active = true`; **không** ghi đè row đã tồn tại (Admin có thể đã sửa). Sau seed, kiểm tra invariant §4.2; vi phạm ⇒ fail-fast lúc khởi động. Tên UI theo COPY-DM-01 (chốt 2026-10-05). Lúc khởi động phân biệt:
  - **Chưa kích hoạt** — không có template nào `is_active`: log WARN, vẫn khởi động, GET trả 503 (§4.4).
  - **Catalog đang hoạt động** — có ít nhất một template `is_active` (từ seed, SQL hay Admin sau này): phải thỏa invariant §4.2, vi phạm ⇒ fail-fast.

- **Sửa catalog bằng SQL [Derived, REL-DM-01 Proposed]:** mọi DML sửa `mission_templates` chạy trong một transaction, gọi `CALL dm_assert_catalog_invariant()` (tạo bởi `migration_daily_mission_v1.sql`) trước `COMMIT`; vi phạm ⇒ `SIGNAL` lỗi và `ROLLBACK`. Mẫu: `snap-vocab-backend/migration_daily_mission_catalog_template.sql`. Không sửa snapshot `user_daily_missions`.

---

## 8. API contract

Base `/api` ([quiz.md](./quiz.md) D1), JWT bắt buộc, envelope `RestResponse {statusCode, data, error}`; `error` là mã lỗi ổn định: `MissionException(MissionErrorCode)` có handler riêng trong `GlobalExceptionHandler`. Exception kỹ thuật rollback toàn bộ và map theo §4.4: `DataIntegrityViolationException` ⇒ 409 `CONCURRENT_MODIFICATION`; `RuntimeException` khác ⇒ 500 `INTERNAL_ERROR` (mã cố định, chi tiết chỉ ghi log server — PLAT-01). Với claim, retry sau 409 `CONCURRENT_MODIFICATION`/500 bằng **cùng** `Idempotency-Key` là an toàn.

### 8.1. `GET /api/me/daily-missions`

Không body, không query. Trong transaction: lock `users` → `today` → `ensureAssigned(user, today)` (§4). Header `Cache-Control: no-store`. Luôn trả **đúng 5** mission của `today`, sắp theo `slot`; `status` ∈ {`IN_PROGRESS`, `COMPLETED`, `CLAIMED`}.

```json
{
  "missionDate": "2026-10-03",
  "timezone": "Asia/Ho_Chi_Minh",
  "serverNow": "2026-10-03T03:15:00Z",
  "resetAt": "2026-10-03T17:00:00Z",
  "missions": [
    {
      "id": 1041,
      "code": "QUIZ_SCORE_80",
      "name": "Trí Nhớ Siêu Phàm",
      "description": "Hoàn thành 1 quiz với độ chính xác ≥ 80%",
      "category": "QUIZ",
      "triggerEvent": "QUIZ_COMPLETED",
      "slot": 4,
      "targetValue": 1,
      "currentProgress": 0,
      "status": "IN_PROGRESS",
      "reward": { "coin": 20, "xp": 40, "itemCode": null, "itemFallbackCoin": null },
      "completedAt": null,
      "claimedAt": null
    }
  ]
}
```

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 503 | `MISSION_POOL_INSUFFICIENT` | `ensureAssigned` trả `POOL_INSUFFICIENT` (§4.4). Không trả danh sách thiếu. |
| 409 | `CONCURRENT_MODIFICATION` | `DataIntegrityViolationException` (§4.4); rollback, không ghi bộ dở |
| 500 | `INTERNAL_ERROR` | Exception kỹ thuật khác |

### 8.2. `POST /api/me/daily-missions/{id}/claim`

Header `Idempotency-Key` (1–128 ký tự, cùng quy ước Shop). Không body.

```text
BEGIN
1. lock users (Economy.lockUser)
2. m = user_daily_missions WHERE id AND user_id FOR UPDATE    -- không có / của user khác → 404 MISSION_NOT_FOUND
3. c = user_daily_mission_claims WHERE user_id AND idempotency_key
     c tồn tại ∧ c.user_daily_mission_id ≠ m.id → 409 IDEMPOTENCY_KEY_REUSED
4. m.status = CLAIMED → COMMIT, trả replay (reward từ claim row, số dư hiện tại, replayed = true)
5. today = LocalDate(Economy.now(), GMT+7)  -- tính SAU khi giữ lock (§3.3)
   m.mission_date < today → 409 MISSION_EXPIRED (không ghi gì)
6. m.status = IN_PROGRESS → 409 MISSION_NOT_COMPLETED
7. reward_coin > 0 → WalletService.apply(+coin, MISSION_REWARD, …)       -- §7.5
   reward_xp   > 0 → XpService.award(userId, MISSION_REWARD, xp, …)
   item           → ShopRewardService.grantShopItemReward(userId, code, 1, fallback, MISSION_REWARD, …)
8. INSERT user_daily_mission_claims; m.status = CLAIMED, m.claimed_at = now
COMMIT
```

Response 200:

```json
{
  "missionId": 1041,
  "status": "CLAIMED",
  "rewardGranted": { "coin": 20, "xp": 40, "itemCode": null, "itemQuantity": 0, "fallbackCoin": 0 },
  "coinBalance": 260,
  "totalXp": 1340,
  "replayed": false
}
```

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 400 | `IDEMPOTENCY_KEY_REQUIRED` | Thiếu/sai header |
| 404 | `MISSION_NOT_FOUND` | Không tồn tại hoặc không thuộc user |
| 409 | `MISSION_NOT_COMPLETED` | `IN_PROGRESS` |
| 409 | `MISSION_EXPIRED` | Mission chưa `CLAIMED` và `mission_date < today` tại thời điểm giữ lock. Mission đã `CLAIMED` hôm trước ⇒ 200 replay (bước 4 đứng trước bước 5). |
| 409 | `IDEMPOTENCY_KEY_REUSED` | Key đã dùng để claim mission khác |
| 409 | `CONCURRENT_MODIFICATION` | `DataIntegrityViolationException` vẫn xảy ra ở lần chạy lại (§4.4); rollback toàn bộ, retry cùng key an toàn |
| 500 | `INTERNAL_ERROR` | Lỗi kỹ thuật khác; rollback toàn bộ, retry cùng key an toàn |

- **Duplicate claim:** mission đã `CLAIMED` ⇒ luôn 200 `replayed = true` (mission là idempotency anchor; key khác cũng replay).
- **Concurrency:** lock `users` → `user_daily_missions`; UNIQUE `user_daily_mission_id` và `event_key` ledger là lớp chặn cuối. Service claim chạy trong `TransactionTemplate` và chạy lại một lần (transaction mới) khi gặp `DataIntegrityViolationException` (cùng mẫu `ShopService`, §4.4) để trả replay thay vì 409 `CONCURRENT_MODIFICATION`.
- **Lỗi kỹ thuật** ở bước 7–8 ⇒ rollback toàn bộ; không thể "cộng coin mà mission chưa CLAIMED".

### 8.3. Admin — `/api/admin/mission-templates` (`ROLE_ADMIN`)

Chỉ cần khi Admin thuộc release (PD-DM-06). Runtime không phụ thuộc phần này.

| Method | Endpoint | Mô tả |
| :--- | :--- | :--- |
| GET | `/api/admin/mission-templates?active=&category=` | Danh sách (không phân trang) |
| GET | `/api/admin/mission-templates/{id}` | Chi tiết |
| POST | `/api/admin/mission-templates` | Tạo, `is_active = false` |
| PATCH | `/api/admin/mission-templates/{id}` | Sửa mọi field trừ `code`, kèm `version`; bật/tắt bằng `isActive` |

Không có PUT, không có DELETE — tắt bằng `isActive = false`.

| DTO | Field |
| :--- | :--- |
| `MissionTemplateDTO` | `id`, `code`, `name`, `description`, `category`, `triggerEvent`, `minQuizAccuracyPercent`, `quizRequestedCount`, `targetValue`, `rewardCoin`, `rewardXp`, `rewardItemCode`, `rewardItemFallbackCoin`, `weight`, `eligibilityRule`, `isActive`, `version`, `createdAt`, `updatedAt` |
| `CreateMissionTemplateRequest` | Bắt buộc: `code`, `name`, `description`, `category`, `triggerEvent`, `targetValue`, `rewardCoin`, `rewardXp`. Tùy chọn: `minQuizAccuracyPercent`, `quizRequestedCount`, `rewardItemCode` + `rewardItemFallbackCoin` (đi cặp), `weight` (mặc định 1), `eligibilityRule` (mặc định `NONE`). Không nhận `isActive`, `version`, `id`. |
| `PatchMissionTemplateRequest` | Bắt buộc: `version`. Field khác: **vắng mặt** = giữ nguyên; **`null` tường minh** chỉ hợp lệ với cột NULL được (`minQuizAccuracyPercent`, `quizRequestedCount`, `rewardItemCode` + `rewardItemFallbackCoin`). Gửi `code` ⇒ `FIELD_LOCKED`. Validation chạy trên trạng thái sau khi áp patch. |

**Bảo vệ invariant catalog:** mọi POST/PATCH trong một transaction: `SELECT id FROM mission_templates FOR UPDATE` (khóa toàn catalog) → áp thay đổi → kiểm tra invariant §4.2 → commit. `ponytail:` khóa toàn catalog, chấp nhận vì catalog nhỏ và ghi rất hiếm. Assignment chỉ đọc catalog (MVCC), không bị khóa.

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 400 | `VALIDATION_ERROR` | Field sai; `trigger_event`/`eligibility_rule` ngoài enum; `min_quiz_accuracy_percent`/`quiz_requested_count` với trigger khác `QUIZ_COMPLETED`; thiếu fallback khi có item; không có reward |
| 400 | `REWARD_ITEM_NOT_ELIGIBLE` | `reward_item_code` không tồn tại hoặc không `PUBLISHED` ([shop.md](./shop.md) §7.5) |
| 404 | `MISSION_TEMPLATE_NOT_FOUND` | Không có `id` |
| 409 | `MISSION_TEMPLATE_CODE_EXISTS` | Trùng `code` |
| 409 | `FIELD_LOCKED` | Đổi `code` |
| 409 | `VERSION_CONFLICT` | Lệch `version` |
| 409 | `MISSION_POOL_INSUFFICIENT` | Thao tác làm catalog vi phạm invariant §4.2 |

---

## 9. Mobile contract (MH-GAME-02) [Derived]

| Hạng mục | Contract |
| :--- | :--- |
| Query | `['daily-missions']` → `GET /api/me/daily-missions` |
| Render | 5 card theo `slot`; `IN_PROGRESS` → "Đang làm" + progress; `COMPLETED` → "Hoàn thành" + nút "Nhận thưởng"; `CLAIMED` → "Đã nhận thưởng". Reward chỉ đọc từ `reward`, không tự tính. Không có rương, thưởng tổng, stamp, bonus. |
| Loading / lỗi | Skeleton 5 card khi chưa có data; 503 `MISSION_POOL_INSUFFICIENT` hoặc lỗi mạng → trạng thái lỗi + nút thử lại. Không render bộ thiếu, không dùng copy kiểu "hôm nay nghỉ" để che lỗi. |
| Claim | Mỗi lần nhấn tạo 1 `Idempotency-Key` (UUID) **bên ngoài** `mutationFn`; retry của cùng lần nhấn dùng lại key. Nút disabled khi đang gửi. 200 → cập nhật mission trong cache, `useAuthStore.getState().updateUser({ coin: coinBalance, exp: totalXp })`, invalidate `shopKeys.all`, `['gamification','xp']`, `['me']`. 409 `MISSION_EXPIRED`/`MISSION_NOT_COMPLETED` → refetch `['daily-missions']`. 500/lỗi mạng → cho thử lại với **cùng** key. |
| Invalidation sau hành động | Invalidate `['daily-missions']` sau khi thành công: complete quiz (thêm vào `invalidateProgress`), add word, purchase, activate booster. Progress commit cùng hành động (§6) nên refetch sau response đã thấy progress mới. Không polling, không realtime. |
| Offline | "Offline" = request thất bại do lỗi mạng (app chưa có thư viện phát hiện mạng). Card hiển thị từ cache; claim lỗi mạng giữ key để thử lại. Không tự tăng progress hay cộng reward ở client. |
| Rollover / foreground | Tính `resetAt` theo độ lệch `serverNow`; tới `resetAt` hoặc app trở lại foreground (`AppState`) → refetch và thay **toàn bộ** bộ 5. Không tự đổi ngày ở client. |
| Đi làm nhiệm vụ | Nút trên card `IN_PROGRESS`, map theo `triggerEvent` (developer-controlled): `TOPIC_ITEM_CREATED` → `/collections` (màn thêm từ cần `topicId`: Learner chọn collection của mình, FAB "Thêm từ" truyền topic; card hiển thị gợi ý điều kiện §5.5); `QUIZ_COMPLETED`, `LEARNING_ACTIVITY_RECORDED` → `/study/quiz-setup`; `SHOP_ITEM_PURCHASED` → tab Shop; `BOOSTER_USED` → `/profile/inventory`. |
| Cảnh báo hết hạn | Mission `COMPLETED` chưa claim hiển thị cảnh báo mất thưởng tại `resetAt` (§2.3). |
| Quick Lookup (DEP-DM-01) | Màn thêm từ điền `main.translation` từ kết quả lookup (`translations`) khi field còn trống. |

---

## 10. Business rules

1. **Idempotency:** claim idempotent theo mission; progress idempotent theo `(mission, source_key)`; word credit theo `(user, word_hash)`; ledger theo `event_key`. [Authoritative + Derived]
2. **Transaction:** claim cập nhật status, cộng Coin/XP/item và ghi claim trong cùng transaction; progress commit cùng hành động gốc. [Authoritative + Derived]
3. **Anti-spam:** không cộng progress cho hành động không tạo giá trị học tập (quiz hủy, lưu từ trùng, từ không dùng được cho quiz). [Authoritative]
4. **Fairness:** không cấp mission user không thể hoàn thành. Template `NONE` khả thi qua content tự tạo (L2); template phụ thuộc tài nguyên chỉ cấp khi đủ điều kiện (§4.3). [Authoritative]
5. **Reward cap:** PD-DM-02 — chưa có giá trị. [Unresolved]
6. **No rollover** và **manual claim deadline:** §3.3. [Authoritative]
7. **Per-mission reward only:** không reward khi đủ 5/5, theo chuỗi ngày, tuần hay tháng. [Authoritative — scope 2026-10-03]
8. **No XP Booster on mission XP:** [shop.md](./shop.md) S6; code `XpSourceType.MISSION_REWARD(boostable = false)`. [Authoritative]
9. **L1 — giới hạn chống gian lận được chấp nhận:** nội dung giả nhưng không rỗng (VD "abc"/"xyz") vẫn tạo progress và reward của `ADD_5_WORDS` và có thể dùng để làm quiz. P3 chỉ chặn việc dùng lại cùng một từ. [Authoritative — duyệt 2026-10-04]
10. **L2 — content tự chuẩn bị:** catalog khả thi nhờ Learner tự chuẩn bị content; không có đường học dựng sẵn (§5.4). [Authoritative — duyệt 2026-10-04]

---

## 11. Admin configuration [Authoritative: F-GAME-09 "Admin CRUD Missions"; Derived: §8.3]

Admin quản lý Mission Pool qua §8.3: tên, mô tả, category, trigger + bộ lọc, target, reward (item phải `PUBLISHED`), weight, eligibility, bật/tắt. Template mới chỉ chọn `trigger_event` trong enum developer-controlled §7.1 — Admin không tạo loại trigger mới. Sửa template không ảnh hưởng mission đã cấp (snapshot §7.2). Trước khi Admin thuộc release (PD-DM-06), catalog đến từ seed §7.7.

---

## 12. Chỉ số theo dõi

| Metric | Ý nghĩa |
| :--- | :--- |
| `daily_mission_open_rate` | Tỷ lệ Learner mở màn Daily Mission mỗi ngày |
| `mission_completion_rate_by_code` | Tỷ lệ hoàn thành theo template |
| `mission_claim_rate_by_code` | Tỷ lệ claim trên mission đã hoàn thành |
| `all_missions_claimed_rate` | Tỷ lệ Learner claim đủ 5/5 (chỉ đo, không có reward) |
| `reward_coin_issued_daily` | Tổng Coin phát hành từ Daily Mission mỗi ngày |
| `mission_abandon_rate_by_slot` | Slot nào thường bị bỏ dở |
| `mission_pool_insufficient_total` | Số lần `ensureAssigned` trả `POOL_INSUFFICIENT` (§4.4); kỳ vọng 0, > 0 là cấu hình sai. Đã có counter Micrometer `mission.pool.insufficient` (tên Prometheus `mission_pool_insufficient_total`) trong `MeterRegistry` của app (actuator có sẵn qua `redisson-spring-boot-starter`) kèm log ERROR. **Chưa có:** export ra hệ thống giám sát (chưa có registry Prometheus/OTLP, `/actuator/metrics` chưa expose) và rule cảnh báo khi counter > 0 — bắt buộc trước rollout |

---

## 13. Acceptance criteria

### 13.1. Cấp nhiệm vụ

- Trigger đầu tiên trong ngày (GET hoặc hành động §6.1) tạo đúng 5 mission khác template; hai trigger song song chỉ tạo 1 bộ.
- Sau khi cấp, mọi lần GET trong ngày trả cùng bộ 5 đã persist.
- Learner mới (0 Coin, 0 item, chưa có từ) nhận `ADD_5_WORDS`, `QUIZ_SCORE_80`, `FINISH_2_SHORT_QUIZZES`, `PERFECT_MINI_QUIZ`; slot 5 là `KEEP_DAILY_STREAK`, hoặc `BUY_SHOP_ITEM` nếu Shop có item mua được với 0 Coin; không bao giờ `USE_XP_BOOSTER`.
- Không cấp `USE_XP_BOOSTER` khi không có booster idle; không cấp `BUY_SHOP_ITEM` khi không có item mua được ngay.
- Activate booster cuối cùng **trước** lần mở màn đầu tiên: eligibility xét trước activation ⇒ `USE_XP_BOOSTER` là ứng viên; nếu được chọn thì hoàn thành ngay trong cùng transaction.
- Ưu tiên mềm: trong pool của mỗi slot, template không có hôm qua được ưu tiên; không có ứng viên mới thì lặp lại hợp lệ.
- Catalog vi phạm invariant ⇒ GET 503; hành động vẫn commit, không tạo mission, metric `mission_pool_insufficient_total` tăng.
- Exception khi insert assignment ⇒ rollback cả hành động.
- Không cấp nhiệm vụ community/study group.

### 13.2. Tracking

- Quiz 5 câu đạt 100%: `QUIZ_SCORE_80`, `PERFECT_MINI_QUIZ` thành `COMPLETED`; `FINISH_2_SHORT_QUIZZES` = 1/2; nếu slot 5 là `KEEP_DAILY_STREAK` thì cũng thành `COMPLETED`.
- Quiz 10 câu đạt 100%: tăng `QUIZ_SCORE_80`, không tăng `FINISH_2_SHORT_QUIZZES`/`PERFECT_MINI_QUIZ`.
- Complete lại quiz đã `COMPLETED`: không tăng gì.
- `ADD_5_WORDS`: thêm từ có bản dịch +1; từ thiếu bản dịch hoặc topic không bind `TARGET_WORD`/`NATIVE_TRANSLATION` không tăng; cùng từ (khác hoa thường/khoảng trắng) lần hai không tăng; retry tạo hai TopicItem cùng từ chỉ tăng 1; từ đã được tính hôm trước, xóa collection rồi thêm lại hôm nay không tăng; ngày không có `ADD_5_WORDS` thì không ghi credit.
- Hành động có `actionInstant` 23:59:59 GMT+7 cập nhật mission của ngày đó, kể cả khi commit sau 00:00; mission đó không claim được sau 00:00.
- Một hành động khớp nhiều mission: tất cả cập nhật trong transaction của hành động; lỗi ⇒ hành động và mọi progress cùng rollback.
- Hai hành động cùng đẩy mission tới target ⇒ đúng một lần chuyển `COMPLETED`; progress không vượt target.

### 13.3. Claim reward

- Chỉ claim được `COMPLETED` của hôm nay; cộng đúng reward snapshot vào `coin_transactions`/`experience_logs`/`user_items` với source `MISSION_REWARD`.
- Claim trùng (cùng hoặc khác key, tuần tự hoặc song song) chỉ cộng 1 lần, response sau `replayed = true`.
- Claim mission hôm trước **chưa `CLAIMED`** khi đã qua 00:00 (tính sau khi giữ lock) ⇒ `409 MISSION_EXPIRED`, không cộng gì. Mission đã `CLAIMED` hôm trước ⇒ 200 replay.
- Lỗi được inject sau khi đã cộng Coin/XP/item ⇒ rollback toàn bộ, mission vẫn `COMPLETED`, retry cùng key cộng đúng 1 lần.
- Response mất sau commit ⇒ retry cùng key trả `replayed = true`, không cộng lại.
- Cùng key dùng cho mission khác ⇒ `409 IDEMPOTENCY_KEY_REUSED`.
- Claim đủ 5/5 không phát sinh reward bổ sung.
- XP mission không được XP Booster nhân.

### 13.4. Config

- Sửa target/reward hoặc tắt template sau khi đã cấp không đổi snapshot của mission đã cấp.
- Seed không ghi đè template đã tồn tại; catalog sau seed vi phạm invariant ⇒ ứng dụng không khởi động.
- (Khi Admin thuộc release) Hai thao tác song song cùng làm pool thiếu ⇒ ít nhất một bị `409 MISSION_POOL_INSUFFICIENT`.

---

## 14. Future Enhancement: phân bổ theo `missionProfile`

Sau MVP, hệ thống có thể tính `missionProfile` mỗi ngày để điều chỉnh weight/target/eligibility, vẫn giữ đúng 5 Daily Missions, snapshot theo ngày, không đổi UI chính. Profile gợi ý: `NEW_LEARNER`, `RETURNING_LEARNER`, `SCAN_HEAVY`, `REVIEW_BACKLOG`, `QUIZ_WEAK`, `CONSISTENT_LEARNER`, `POWER_LEARNER` (ưu tiên theo thứ tự `RETURNING_LEARNER` → `NEW_LEARNER` → `REVIEW_BACKLOG` → `QUIZ_WEAK` → `SCAN_HEAVY` → `POWER_LEARNER` → `CONSISTENT_LEARNER`). Để sau MVP vì cần dữ liệu hành vi, test economy và dashboard theo profile.

---

## 15. Quyết định MVP

- Mỗi ngày đúng **5 Daily Missions**; không Bonus Mission.
- Reward chỉ theo từng mission (Coin/XP/item tùy chọn); không Daily Chest, không thưởng tuần/tháng, không thưởng tổng.
- Reset theo **Asia/Ho_Chi_Minh (GMT+7)**; mission chưa claim trước reset **hết hạn và mất reward**.
- Không mission cộng đồng/nhóm học tập; không notification khi hoàn thành mission.
- Active Catalog = §5.1 (gói PD-DM-01 duyệt 2026-10-04); template Deferred §5.3 không được seed.
- Progress cập nhật đồng bộ trong transaction của hành động (§6); không outbox/queue.
- Còn mở, không chặn: PD-DM-02, PD-DM-06, PD-LB-01; dependency DEP-DM-01 (Learning/mobile).
