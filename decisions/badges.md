# Đặc tả tính năng: Badges & Achievements

> **Trạng thái tài liệu:** **ACTIVE / AUTHORITATIVE** cho rule và contract của Badges (SS-13, F-GAME-05). Product Owner duyệt gói PD-BDG-01…12 + COPY-BDG-01 ngày 2026-10-07 và PD-BDG-13a–d (Admin Huy hiệu) ngày 2026-10-08 (§0). Authority theo domain: quyết định PO > file này cho rule Badge; [specs.md](../spec/specs.md) FR-09.04 cho phạm vi chức năng; [database.md](../db/database.md) cho schema và phải khớp §7; [design.md](../../snapVocab-frontend/docs/design.md) §10.5 cho hiển thị; [quiz.md](./quiz.md) cho quiz; [daily_mission.md](./daily_mission.md) cho Daily Mission (Badge **không** phải reward của Daily Mission). Phân rã, SA, mobile/admin design là DERIVED.
>
> **Readiness (2026-10-07):** **Đặc tả: đủ để bắt đầu triển khai MVP · Production: chưa đạt.** Mọi mục PD-BDG đã Resolved; CLEAN-BDG-01 đã kiểm tra và hoàn tất (§13). DEP-BDG-01…03 chỉ chặn badge Deferred. Nghiệm thu và readiness production là bước riêng. **Hiện trạng code (2026-10-07):** backend đã có domain (package `vn.ptit.snapvocab.badge`: entity §7.1–§7.3, `BadgeMetric`, repository, `BadgeCatalogSeeder` §4.1, `migration_badges_v1.sql`, `BadgeCatalogSeederTest`); `BadgeService` + `BadgeController` (`GET /api/me/badges`, `POST /api/me/badges/sync`, `POST /api/me/badges/seen` §8) và word credit trong add-item §6.2; `BadgeIntegrationTest` pass trên MySQL local; script chưa áp staging/production; mobile đã tích hợp theo [achievements_ui.md](../ui/achievements_ui.md) (package `SnapVocab/src/features/badges`, route `/profile/achievements`, `BadgeUnlockHost`, dòng nối tiếp trên Quiz Result, Profile; unit/component test pass; chưa nghiệm thu trên thiết bị; P2: confetti mốc cuối, haptics); Admin Badges mock đã gỡ import/route/menu khỏi bundle release, giữ source prototype; Admin Huy hiệu có prototype P1 dev-only (`snapAdmin/src/features/admin-badges`, fixture, cần bản dev + `VITE_ADMIN_BADGES_ENABLED=true`), chưa có API admin (§8.4, §10); `snapVocab-frontend/app/(tabs)/achievements.tsx` là mock trái scope (§4.3).
>
> **Lịch sử:** rev 1 (2026-10-07) bản nháp · rev 2 (2026-10-07) áp review: đếm từ theo C, thêm PD-BDG-11/12, TD-BDG-01, sửa contract · rev 3 (2026-10-07) PO duyệt toàn bộ phương án đề xuất; đổi nhãn [Proposed] → [Authoritative]/[Derived] · rev 4 (2026-10-07) Product/Design duyệt gói UI-BDG-01…08; visual chuyển sang [docs/ui/achievements_ui.md](../ui/achievements_ui.md); cập nhật §2, §9. Không đổi rule nghiệp vụ · rev 5 (2026-10-08) PO duyệt PD-BDG-13a–d: mở Admin Huy hiệu theo bậc (§10), contract admin (§8.4), cập nhật §5.3, §7.1, §7.4, thêm §12.5. Không đổi rule cấp badge.

> **Tiến độ Admin P1 (2026-10-08):** đã có `AdminBadgeController` + `AdminBadgeService` (GET list/detail chỉ đọc, kèm `earnerCount`); Admin dùng API thật thay fixture và giữ flag opt-in `VITE_ADMIN_BADGES_ENABLED`. `version` trả `null` và bị ẩn vì chưa có migration P2. UI ghi tại [admin_badges_ui.md](../ui/admin_badges_ui.md). Chưa phát hành staging/production; kiểm chứng local và staging acceptance là hai bước riêng.

Quy ước nhãn (giống [daily_mission.md](./daily_mission.md)):

- **[Authoritative]**: có nguồn gốc (FR-09.04, F-GAME-05, design.md §10.5, quiz.md, scope Daily Mission 2026-10-03, hoặc file được trích dẫn).
- **[Derived]**: suy ra trực tiếp từ rule authoritative + code hiện có; không thêm hành vi sản phẩm mới.
- **[Proposed]**: đề xuất mới sau khi gói được duyệt, **chưa** được PO duyệt; không thay đổi rule [Authoritative]. Hiện không có mục nào ở trạng thái này.

---

## 0. Decision register

PO duyệt cả gói ngày **2026-10-07**, chọn đúng phương án đã đề xuất ở mọi mục. PD-BDG-13a–d (Admin Huy hiệu, release tiếp theo) duyệt ngày **2026-10-08**.

| ID | Câu hỏi | Trạng thái | Quyết định |
| :--- | :--- | :--- | :--- |
| PD-BDG-01 | Phạm vi MVP? | **Resolved 2026-10-07** | MVP gọn §1.2. Mỗi nhóm trong §1.3 (Titles, secret, cấp tay…) muốn đưa vào phải có PD riêng |
| PD-BDG-02 | Catalog v1? | **Resolved 2026-10-07** | 12 badge / 4 metric §4.1. Scan, mastered, leaderboard hoãn (§4.2) |
| PD-BDG-03 | "Từ đã thêm" đếm thế nào? | **Resolved 2026-10-07** | **C:** số từ khác nhau (chuẩn hóa) từng tự thêm qua add-item, item quiz-ready (§3, §6.2). Không chọn A (tính cả fork, giảm khi xoá) và B (lặp một từ vẫn tính) |
| PD-BDG-04 | Đạt badge có kèm XP/Coin? | **Resolved 2026-10-07** | **Không.** Badge là phần thưởng; không ghi ledger (§5.6) |
| PD-BDG-05 | Vòng đời badge? | **Resolved 2026-10-07** | Nhận một lần, không thu hồi, ngưỡng bất biến (đổi ngưỡng = badge mới), tắt badge thì người đã nhận vẫn thấy (§5) |
| PD-BDG-06 | Khi nào ghi nhận đạt badge? | **Resolved 2026-10-07** | **Khi app đồng bộ** (§6.1). PO chấp nhận ngày đạt = ngày hệ thống ghi nhận và các hệ quả §6.3. Ghi nhận lúc hành động là hướng nâng cấp (§14) |
| PD-BDG-07 | User cũ đã đủ điều kiện lúc ra mắt? | **Resolved 2026-10-07** | Cấp bù streak và quiz từ dữ liệu có sẵn. Nhóm "từ" chỉ tính từ ngày bật tracking; màn chi tiết ghi rõ (§2.2) |
| PD-BDG-08 | Báo "badge mới" thế nào? | **Resolved 2026-10-07** | Modal gộp + chấm đỏ; không ngắt phiên quiz/flashcard; lỗi đánh dấu đã xem chỉ thử gửi lại trong phiên (§2.3, §9). Không push, không hộp thông báo |
| PD-BDG-09 | Hiển thị? | **Resolved 2026-10-07** | Không tier/rarity; badge chưa đạt vẫn hiện tên + điều kiện; Profile hiện 3 badge gần nhất, thứ tự ổn định (§2.4) |
| PD-BDG-10 | Release đầu có Admin CRUD? | **Resolved 2026-10-07** | **Không.** Catalog qua seed/migration (§7.4); đã gỡ import/route/menu Admin Badges mock khỏi bản release, giữ source prototype. Release tiếp theo theo PD-BDG-13a |
| PD-BDG-11 | Hoạt động nào tính là một ngày học? | **Resolved 2026-10-07** | **Chỉ hoàn thành quiz** trong MVP, mô tả badge streak ghi rõ (§3, §4.1). Mở rộng khi luồng học khác có tracking |
| PD-BDG-12 | Perfect Quiz làm lại cùng topic có tính? | **Resolved 2026-10-07** | **Có**, mỗi quiz đạt 100% tính một lần. Tên/mô tả chỉ nói số lần đạt điểm tuyệt đối (§4.1) |
| PD-BDG-13a | Admin Huy hiệu: phạm vi và phát hành? | **Resolved 2026-10-08** | MVP = P1 (chỉ đọc) + P2 (sửa hiển thị, Bật/Tắt), phát hành theo bậc khi backend và nghiệm thu của bậc đạt; P1 được phát hành trước P2. P3 (tạo, thêm mốc) là mở rộng. G11 không bắt buộc cho MVP (§10) |
| PD-BDG-13b | Admin sửa nội dung huy hiệu đã có người đạt? | **Resolved 2026-10-08** | **Có**: `name`, `description`, `icon_key`, `sort_order`; áp dụng cả người đã đạt, giữ ngày đạt và trạng thái đã xem. Người sửa giữ nguyên ý nghĩa và điều kiện; không cần PO duyệt từng lần sửa (§10) |
| PD-BDG-13c | Thêm mốc mới rồi ngừng cấp mốc cũ? | **Resolved 2026-10-08** | Hai thành tựu độc lập; người học có thể giữ cả hai. `code`/`metric`/`threshold` bất biến; không chuyển đổi, hợp nhất, thu hồi. Không mở G9, không mở P3 (§5.3, §10) |
| PD-BDG-13d | Ước tính cấp bù khi Bật? | **Resolved 2026-10-08** | Nên có, không chặn P2. Tính tài khoản `activated = true` đủ điều kiện chưa nhận, kèm thời điểm tính; lỗi hoặc thiếu ⇒ không hiện `0` (§8.4) |
| COPY-BDG-01 | Tên + mô tả 12 badge | **Resolved 2026-10-07** | Theo cột "Tên", "Mô tả" §4.1 và câu ghi chú §2.2. Từ PD-BDG-13b đây là nội dung khởi tạo và mẫu văn phong đã duyệt; nội dung đang phát hành là dữ liệu catalog trên server |

**Quyết định kỹ thuật (đội kỹ thuật chốt):**

| ID | Nội dung | Quyết định |
| :--- | :--- | :--- |
| TD-BDG-01 | HTTP cho việc ghi nhận khi đồng bộ | **Accepted 2026-10-07:** `POST /api/me/badges/sync` ghi nhận + trả view; `GET /api/me/badges` **chỉ đọc** (§8). Không theo mẫu GET có ghi của `GET /api/me/daily-missions` |

**Quyết định UI và contract mobile (Product/Design duyệt 2026-10-07, chi tiết ở [achievements_ui.md](../ui/achievements_ui.md)):**

| ID | Nội dung | Quyết định |
| :--- | :--- | :--- |
| UI-BDG-01 (OD-1) | Badge mới sau quiz | Không bật modal ở Quiz Result; dòng nối tiếp trên Quiz Result; modal ở route hợp lệ đầu tiên (denylist: quiz-session, flashcard, quiz-result, achievements) |
| UI-BDG-02 (OD-2) | Visual huy chương | Hình dáng theo chuỗi, emblem đậm trên mặt sáng (thay "tròn, icon trắng" vì chỉ đạt 1,59:1); chi tiết ở file UI |
| UI-BDG-03 (OD-3) | Badge mới trên màn Thành tựu | Giới thiệu ngay trên màn, không modal; `/seen` theo snapshot `introducedIds` khi rời màn |
| UI-BDG-04 (B1) | Điểm sync | Thêm "vào màn Thành tựu" và "kéo làm mới"; gọi tường minh trong mutation quiz và thêm từ |
| UI-BDG-05 (B2) | Tiến độ trên gallery | Mốc kế tiếp hiện `cur/thr`, mốc xa hiện ngưỡng, đầy đủ trong chi tiết (§2.1) |
| UI-BDG-06 (B5, B6) | Lời mời hành động, mức phản hồi | Lời mời cấp ba chỉ cho badge chưa đạt; phản hồi lớn hơn cho mốc cuối chuỗi |
| UI-BDG-07 (B8) | Thứ tự canonical ở client | Item: `threshold`, `sortOrder`, `id`; series: `min(sortOrder)`; Profile: `earnedAt`, `sortOrder`, `id` giảm dần |
| UI-BDG-08 (B9) | Ưu tiên trạng thái màn | `NEW → COMPLETE → RECORDING → EMPTY → NORMAL`, STALE phủ trên |

**Dependency (chặn badge tương ứng):**

| ID | Nội dung | Chặn |
| :--- | :--- | :--- |
| DEP-BDG-01 | Lưu nguồn gốc scan → từ đã lưu. Hiện `scan_requests` không ghi từ nào được lưu, `topic_items` không có cột nguồn, `LearningEventType.WORD_SCANNED_AND_SAVED` không có producer | Badge scan (§4.2) |
| DEP-BDG-02 | SRS review ghi `fsrs_records`. Hiện chỉ `ProgressService` **đọc** bảng này, không code nào ghi ⇒ số từ mastered luôn bằng 0 | Badge "từ đã thuộc" (§4.2) |
| DEP-BDG-03 | Schema Leaderboard (database.md: "chưa thiết kế") | Badge xếp hạng (§4.2) |
| CLEAN-BDG-01 | **Hoàn tất 2026-10-07:** tài liệu dẫn xuất đã đối chiếu; import/route/menu Admin Badges mock đã gỡ; build production pass. Chi tiết [báo cáo kiểm tra](../reviews/badges_clean_consistency_2026-10-07.md) | Đã hoàn tất; không thay thế nghiệm thu Badge MVP |

---

## 1. Phạm vi

### 1.1. Mục tiêu [Authoritative FR-09.04 + PD-BDG-04]

- Ghi nhận cột mốc học tập thật: chuỗi ngày học, số quiz, số lần đạt điểm tuyệt đối, vốn từ tự xây.
- Cho Learner thấy mốc tiếp theo và còn cách bao xa (progress của badge chưa đạt).
- Không tạo thêm nguồn phát hành Coin/XP (PD-BDG-04).

### 1.2. Trong MVP [Authoritative PD-BDG-01]

- Catalog cố định (seed), 12 badge thuộc 4 metric (§4.1).
- Tự động ghi nhận khi app đồng bộ; mỗi badge tối đa 1 lần; không thu hồi.
- Gallery + chi tiết + progress của badge chưa đạt (MH-GAME-03).
- Báo "badge mới" bằng modal gộp + chấm đỏ (PD-BDG-08).
- Profile hiện 3 badge đạt gần nhất.

### 1.3. Ngoài phạm vi MVP [Authoritative PD-BDG-01]

Mỗi mục cần một quyết định riêng trước khi đưa vào scope:

- Thưởng XP/Coin/item khi đạt badge (PD-BDG-04).
- Danh hiệu/Flair (`HonoraryTitle` trong admin mock). Không spec nào định nghĩa; dễ nhầm với `levels.title` ([database.md](../db/database.md) §3).
- Tiền tệ `gems`: hệ thống chỉ có Coin.
- Huy hiệu bí mật: trái design.md §10.5 ("không dùng dấu ? bí ẩn").
- Badge sự kiện / giới hạn thời gian, league / mùa giải.
- Rarity / tier (bronze→diamond, Common→Legendary).
- Admin cấp tay, thu hồi, sửa ngưỡng hồi tố.
- Push notification, mục trong hộp thông báo (PD-BDG-08).
- Ghim badge lên profile.
- Mọi metric phụ thuộc rương (`CHESTS_OPENED`): rương đã bị loại khỏi scope ([daily_mission.md](./daily_mission.md) scope 2026-10-03).

---

## 2. Trải nghiệm người dùng [Authoritative PD-BDG-08/09 + UI-BDG (§0)]

Visual và interaction chi tiết (huy chương, kệ, trạng thái, sheet, modal, motion, accessibility) thuộc [docs/ui/achievements_ui.md](../ui/achievements_ui.md). Mục này chỉ giữ các rule hành vi.

### 2.1. Gallery (MH-GAME-03)

- Badge hiển thị theo **kệ, mỗi `metric` một kệ**; trong kệ sắp theo ngưỡng tăng dần (thứ tự canonical: [achievements_ui.md](../ui/achievements_ui.md) §4.1).
- Đầu màn: `Đã đạt {earnedCount}/{totalCount}` lấy từ API (§8.1). Không viết cứng số 12: catalog đổi được và badge đã tắt vẫn hiện với người đã đạt.
- Badge chưa đạt **vẫn hiện** tên và điều kiện. Trên kệ, mốc kế tiếp của mỗi chuỗi hiện tiến độ `min(currentValue, threshold)/threshold`; mốc xa hơn hiện ngưỡng; tiến độ đầy đủ của mọi badge chưa đạt hiện trong chi tiết (§2.2). Trạng thái không chỉ phân biệt bằng màu.
- Đủ ngưỡng nhưng chưa được ghi nhận (`!earned && currentValue >= threshold`) ⇒ hiện "Đang ghi nhận…", không hiện lời mời của trạng thái chưa có badge.
- Không có bộ lọc trong MVP.

### 2.2. Chi tiết badge

Tên, mô tả (điều kiện), ngày đạt (`earnedAt`, định dạng ngày GMT+7) hoặc tiến độ nếu chưa đạt. **Không** hiển thị rarity hay tỷ lệ người đạt. Badge metric `DISTINCT_WORDS_ADDED` có thêm dòng ghi chú cố định: "Chỉ tính các từ thêm sau ngày ra mắt huy hiệu" (PD-BDG-07, COPY-BDG-01). Badge chưa đạt (và chưa đủ ngưỡng) có **một** lời mời hành động cấp ba dẫn tới hoạt động của metric (quiz hoặc thêm từ); badge đã đạt không có lời mời.

### 2.3. Badge mới

- Có badge `seen = false` ⇒ 1 badge: modal chi tiết; ≥ 2 badge: **một** modal gộp "Bạn có N huy hiệu mới" liệt kê các badge. Mốc cuối của một chuỗi (≥ 2 mốc) có phản hồi lớn hơn (UI-BDG-06).
- **Không ngắt phiên học:** không hiện modal ở màn làm quiz, flashcard, màn kết quả quiz và màn Thành tựu. Modal hiện ở **route hợp lệ đầu tiên** sau đó (UI-BDG-01). Màn kết quả quiz hiện một dòng nối tiếp; chạm vào thì mở modal ngay.
- Trên màn Thành tựu, badge chưa xem được giới thiệu ngay trên màn; khi rời màn, đánh dấu đã xem **đúng các badge đã được giới thiệu** (snapshot `introducedIds`, UI-BDG-03).
- Đóng modal ⇒ đánh dấu đã xem đúng các id đã hiện (§8.3).
- Shortcut Achievements trên Profile có chấm đỏ khi `unseenCount > 0`.

### 2.4. Profile (MH-PROFILE)

Hiện tối đa 3 badge đã đạt, sắp theo `earnedAt` giảm dần, rồi `sortOrder` giảm dần, rồi `id` giảm dần (ổn định khi nhiều badge được cấp cùng lúc; badge ngưỡng cao hơn trong cùng nhóm đứng trước). Lấy từ cùng response §8.1. Chưa có badge ⇒ ẩn khối này, chỉ giữ shortcut Achievements.

---

## 3. Metric và quy tắc đếm

| Metric | Nguồn | Giá trị có thể giảm? | Ghi chú |
| :--- | :--- | :--- | :--- |
| `LONGEST_STREAK_DAYS` | `users.longest_streak_days` | Không | [Derived] Dùng streak **dài nhất** ⇒ mất streak không ảnh hưởng (PD-BDG-05). Streak tăng trong `ProgressService.record`; producer duy nhất của `learning_events` hiện là quiz complete ⇒ **một ngày học = ngày có ít nhất 1 quiz hoàn thành** (PD-BDG-11) |
| `QUIZ_COMPLETED` | `COUNT(*) FROM quizzes WHERE user_id = ? AND status = 'COMPLETED'` | Không, theo code hiện tại | Quiz `CANCELLED`/`EXPIRED` không tính. [Derived] Backend không có đường xoá quiz, và quiz là snapshot độc lập với topic ([quiz.md](./quiz.md)): xoá topic hay từ không xoá quiz |
| `PERFECT_QUIZ` | `COUNT(*) FROM quizzes WHERE user_id = ? AND status = 'COMPLETED' AND correct_count = total_units` | Không, theo code hiện tại | Mọi mode. [Derived] Quiz luôn có đúng `requestedCount` ∈ {5, 10, 20} unit ([quiz.md](./quiz.md) D14) ⇒ không cần điều kiện tối thiểu riêng. Unit của MCQ/FILL_BLANK là câu, của MATCHING là cặp (`roundNo` không phải đơn vị chấm). Làm lại cùng topic vẫn tính (PD-BDG-12) |
| `DISTINCT_WORDS_ADDED` | `COUNT(*) FROM user_badge_word_credits WHERE user_id = ?` (§7.3) | Không | [Authoritative PD-BDG-03 = C] Mỗi từ chuẩn hóa được tính **một lần trong đời tài khoản**, khi lần đầu được thêm qua `POST /api/topics/{topicId}/items` thành item **quiz-ready** trong collection `USER` của chính Learner (§6.2). Fork/import không tính |

Quy tắc chung:

- Badge đã cấp **không** bị thu hồi dù metric có giảm trong tương lai (§5).
- `currentValue` trong API là giá trị metric lúc đọc, không cắt theo ngưỡng.
- **Nếu sau này thêm tính năng xoá quiz:** hai metric quiz có thể giảm trước lần đồng bộ kế tiếp và user mất thành tựu chưa được ghi nhận. Khi đó phải chuyển sang ghi nhận lúc hành động (§14) trước khi phát hành tính năng xoá.
- **L-BDG-1 [Authoritative PD-BDG-03/04]:** nội dung giả nhưng không rỗng (VD "abc") vẫn được tính là một từ khác nhau, giống L1 của Daily Mission. Chấp nhận vì badge không có thưởng.
- **L-BDG-2 [Authoritative PD-BDG-12]:** `PERFECT_QUIZ` đạt được bằng cách làm lại quiz 5 câu trên cùng một topic dễ. Chấp nhận; tên và mô tả badge chỉ nói số lần đạt điểm tuyệt đối.

---

## 4. Catalog

### 4.1. Active MVP Catalog [Authoritative PD-BDG-02, COPY-BDG-01]

| `sort_order` | `code` | Metric | Ngưỡng | Tên | Mô tả (điều kiện) | `icon_key` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 10 | `STREAK_7` | `LONGEST_STREAK_DAYS` | 7 | Một Tuần Bền Bỉ | Hoàn thành ít nhất 1 quiz mỗi ngày, 7 ngày liên tiếp | `flame` |
| 11 | `STREAK_30` | `LONGEST_STREAK_DAYS` | 30 | Tháng Không Nghỉ | Hoàn thành ít nhất 1 quiz mỗi ngày, 30 ngày liên tiếp | `flame` |
| 12 | `STREAK_100` | `LONGEST_STREAK_DAYS` | 100 | Trăm Ngày Kiên Trì | Hoàn thành ít nhất 1 quiz mỗi ngày, 100 ngày liên tiếp | `flame` |
| 20 | `QUIZ_1` | `QUIZ_COMPLETED` | 1 | Bài Quiz Đầu Tiên | Hoàn thành 1 quiz | `clipboard-check` |
| 21 | `QUIZ_10` | `QUIZ_COMPLETED` | 10 | Quen Tay Quiz | Hoàn thành 10 quiz | `clipboard-check` |
| 22 | `QUIZ_50` | `QUIZ_COMPLETED` | 50 | Chiến Binh Quiz | Hoàn thành 50 quiz | `clipboard-check` |
| 30 | `PERFECT_1` | `PERFECT_QUIZ` | 1 | Điểm Tuyệt Đối | Đúng tất cả câu (hoặc tất cả cặp) trong 1 quiz | `target` |
| 31 | `PERFECT_10` | `PERFECT_QUIZ` | 10 | Mười Lần Tuyệt Đối | Đạt điểm tuyệt đối 10 lần (mỗi quiz đúng tất cả câu hoặc cặp) | `target` |
| 32 | `PERFECT_50` | `PERFECT_QUIZ` | 50 | Năm Mươi Lần Tuyệt Đối | Đạt điểm tuyệt đối 50 lần (mỗi quiz đúng tất cả câu hoặc cặp) | `target` |
| 40 | `WORDS_10` | `DISTINCT_WORDS_ADDED` | 10 | Mười Từ Đầu Tiên | Tự thêm 10 từ khác nhau vào bộ sưu tập của bạn | `book-open` |
| 41 | `WORDS_100` | `DISTINCT_WORDS_ADDED` | 100 | Kho Từ Trăm Chữ | Tự thêm 100 từ khác nhau vào bộ sưu tập của bạn | `book-open` |
| 42 | `WORDS_500` | `DISTINCT_WORDS_ADDED` | 500 | Thư Viện Sống | Tự thêm 500 từ khác nhau vào bộ sưu tập của bạn | `book-open` |

- Ngưỡng streak 7/30/100 trùng mốc streak của design.md §10.3.
- `icon_key` là tên icon `lucide-react-native` (thư viện app đang dùng); mobile map `icon_key` lạ về icon mặc định.
- Zero-state: Learner mới đạt được `QUIZ_1` và `PERFECT_1` ngay trong phiên đầu tiên.
- "Từ" trong mô tả nhóm `WORDS_*` hiểu theo §6.2: item có đủ từ đích + bản dịch (quiz-ready); thêm lại cùng từ (khác hoa thường/khoảng trắng) không tính lần hai.

### 4.2. Deferred (không seed)

| Ý tưởng | Lý do hoãn |
| :--- | :--- |
| Scan có lưu từ (10/100) — ví dụ trong F-GAME-05 | DEP-BDG-01. Khi có nguồn gốc cần chốt thêm: một scan lưu 5 từ tính 1 hay 5; lưu lại cùng kết quả có tính không |
| Từ đã thuộc (mastered = `REVIEW` ∧ `stability ≥ 21`, theo `ProgressService`) | DEP-BDG-02 |
| Thứ hạng Leaderboard | DEP-BDG-03; Leaderboard chỉ là bảng cá nhân, không league/mùa giải |
| Hoàn thành 100% topic (`TOPICS_COMPLETED`) | Chưa có định nghĩa "topic hoàn thành" |
| Học đêm khuya, sự kiện, badge bí mật | Ngoài scope (§1.3) |

### 4.3. Đối chiếu admin mock và prototype

| Hạng mục mock | Xử lý | Lý do |
| :--- | :--- | :--- |
| `BadgeCategory` (6 giá trị) / `Category` prototype (5 giá trị) | Thay bằng `metric` | Nhóm hiển thị = metric |
| `BadgeTier`, `Rarity` | Bỏ | PD-BDG-09 |
| `BadgeStatus.secret`, `isSecret`, `secretHint` | Bỏ | design.md §10.5 |
| `BadgeStatus.draft`/`archived` | Thay bằng `is_active` | §5 |
| `BadgeReward` (xp, coins, gems, titleId) | Bỏ | PD-BDG-04, §1.3 |
| `HonoraryTitle`, tab Titles & Flair | Bỏ | §1.3 |
| `UnlockMetricType.STREAK_DAYS` | Đổi thành `LONGEST_STREAK_DAYS` | §3 |
| `AI_SCAN_SAVED`, `SRS_MASTERED`, `LEADERBOARD_RANK`, `TOPICS_COMPLETED` | Deferred | §4.2 |
| `CHESTS_OPENED` | Bỏ | Rương ngoài scope |
| `PERFECT_QUIZ` | Giữ | §3 |
| `seriesId`/`seriesLevel` | Bỏ | Nhóm theo metric + ngưỡng là đủ |
| Tab Trigger Rules, Manual Grant, Revoke | Bỏ | PD-BDG-01, PD-BDG-10 |
| `isEquippedFeatured` (ghim) | Bỏ | PD-BDG-09 |
| `totalEarners`, `unlockRate`, metrics ribbon | Hoãn | Chỉ cần khi có Admin (PD-BDG-10) |
| Badge "Cú Đêm" dùng `STREAK_DAYS` cho "lượt học đêm khuya" | Bỏ | Gán sai metric; ngoài scope |

---

## 5. Vòng đời và business rules [Authoritative PD-BDG-05]

1. **Một lần:** mỗi `(user, badge)` được cấp tối đa 1 lần; DB bảo vệ bằng `UNIQUE(user_id, badge_id)`.
2. **Không thu hồi:** mất streak, xoá từ, xoá topic không làm mất badge đã cấp. Không có API thu hồi.
3. **Bất biến:** `code`, `metric`, `threshold` không đổi sau khi tạo. Muốn đổi ngưỡng ⇒ tạo badge mới, tắt badge cũ; theo PD-BDG-13c đây là **thêm một thành tựu độc lập**, không phải thay thế: người đã có badge cũ giữ nó và có thể nhận thêm badge mới (§10). `name`, `description`, `icon_key`, `sort_order`, `is_active` sửa được qua migration SQL (§7.4) hoặc Admin P2 (PD-BDG-13b, §8.4).
4. **Tắt badge (`is_active = false`):** không cấp mới trong các lần sync đọc catalog sau khi Tắt commit (sync đã đọc catalog trước đó vẫn có thể cấp, §10); người đã nhận vẫn thấy badge trong gallery và profile; người chưa nhận không thấy badge đó.
5. **Không xoá:** không có DELETE. Tắt bằng `is_active` (cùng mẫu `mission_templates`).
6. **Không thưởng:** cấp badge không ghi `coin_transactions` hay `experience_logs` (PD-BDG-04).
7. **Cấp bù:** badge mới bật sau này, hoặc user đã đủ điều kiện từ trước, được cấp ở lần đồng bộ kế tiếp (§6.1). Nhóm "từ" chỉ có dữ liệu từ ngày bật tracking (PD-BDG-07).
8. **Server là nguồn duy nhất:** client không gửi metric hay yêu cầu cấp một badge cụ thể.

---

## 6. Ghi nhận badge [Authoritative PD-BDG-06, TD-BDG-01]

### 6.1. Đồng bộ (`POST /api/me/badges/sync`)

```text
POST /api/me/badges/sync                               -- @Transactional(isolation = READ_COMMITTED)
1. user = Economy.lockUser(userId)                     -- tuần tự với quiz complete / add-item của cùng user
2. values = { LONGEST_STREAK_DAYS: user.longestStreakDays,
              QUIZ_COMPLETED: count…, PERFECT_QUIZ: count…,
              DISTINCT_WORDS_ADDED: count user_badge_word_credits }   -- §3
3. earnedIds = user_badges WHERE user_id
4. for b in badges WHERE is_active AND id ∉ earnedIds AND values[b.metric] >= b.threshold
         ORDER BY sort_order:
     INSERT user_badges(user_id, badge_id = b.id, seen_at = NULL)   -- created_at = now = earnedAt
5. trả view §8.1
```

- **Không có hook cấp badge trong quiz hay producer khác.** Producer duy nhất được thêm là ghi word credit (§6.2).
- Mobile gọi sync khi mở app, khi app quay lại foreground, và sau khi complete quiz / add word thành công (§9) ⇒ badge thường hiện ngay sau hành động.
- **Isolation:** `READ_COMMITTED` như Quiz/Shop/Mission. Dưới `REPEATABLE READ`, snapshot tạo trước khi chờ lock `users` sẽ không thấy quiz vừa commit.
- **Concurrency:** mọi đường ghi `user_badges` đều giữ lock `users` trước ⇒ hai sync song song không cấp trùng (sync sau chờ lock rồi thấy row của sync trước). `UNIQUE(user_id, badge_id)` là lớp chặn cuối ⇒ 409 `CONCURRENT_MODIFICATION`, retry an toàn.
- **Lỗi:** exception chỉ rollback sync đó; quiz và add-item không bị ảnh hưởng.

### 6.2. Word credit cho `DISTINCT_WORDS_ADDED`

Trong `TopicService`, cùng chỗ gọi `missionTracker.apply(… TOPIC_ITEM_CREATED …)`, khi `learnerId != null && wordHash != null` (item quiz-ready trong collection `USER` của chính Learner; `wordHash` = `MissionTracker.wordHash`, chuẩn hóa theo [daily_mission.md](./daily_mission.md) §5.5 mục 3):

```text
-- trong transaction add-item, đã giữ lock users (daily_mission.md §3.2)
if not exists user_badge_word_credits(learnerId, wordHash):
    INSERT user_badge_word_credits(user_id = learnerId, word_hash = wordHash, created_at = actionInstant)
```

- Kiểm tra tồn tại dưới lock rồi mới insert, giống `user_word_credits`; `UNIQUE(user_id, word_hash)` là lớp chặn cuối.
- Add-item rollback ⇒ credit rollback theo. Retry add-item tạo hai item cùng từ ⇒ chỉ 1 credit.
- **Tách khỏi `user_word_credits`:** bảng của Daily Mission chỉ ghi khi mission `ADD_5_WORDS` thực sự tăng (daily_mission.md §5.5), nên không dùng làm số từ trọn đời được. Hai bảng dùng cùng hàm `wordHash`.
- Chỉ ghi credit, không cấp badge; badge được cấp ở lần sync kế tiếp.

### 6.3. Hệ quả PO đã chấp nhận (PD-BDG-06)

- `earnedAt` = thời điểm sync ghi nhận, không phải thời điểm hành động. Ví dụ: đạt streak 30 hôm thứ Hai bằng một quiz cuối ngày nhưng sync lỗi mạng, thứ Tư mới mở lại app ⇒ `earnedAt` = thứ Tư.
- Không gửi được push đúng lúc đạt badge (server không biết badge mới cho tới khi có sync). Push đang ngoài MVP (PD-BDG-08).
- Không mất thành tựu với dữ liệu hiện tại: cả 4 metric đều không giảm theo code hiện tại (§3). Rủi ro chỉ xuất hiện nếu sau này thêm tính năng xoá quiz.
- **Hướng nâng cấp** (khi cần `earnedAt` chính xác, push, hoặc có tính năng xoá quiz): gọi cùng logic §6.1 bước 2–4 ở cuối quiz complete và add-item, giữ sync để cấp bù.

---

## 7. Data model [Derived — theo quy ước [database.md](../db/database.md): `snake_case`, PK `BIGINT AUTO_INCREMENT`, `DATETIME(6)`, enum lưu `VARCHAR` + CHECK]

### 7.1. `badges` (kế thừa `BaseTimeEntity`)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `code` | VARCHAR(64) | UNIQUE, NOT NULL, bất biến | VD `STREAK_7` |
| `name` | VARCHAR(100) | NOT NULL | Tên UI |
| `description` | VARCHAR(255) | NOT NULL | Điều kiện đạt, hiển thị cả khi chưa đạt |
| `metric` | VARCHAR(30) | NOT NULL, CHECK IN (`LONGEST_STREAK_DAYS`, `QUIZ_COMPLETED`, `PERFECT_QUIZ`, `DISTINCT_WORDS_ADDED`), bất biến | §3 |
| `threshold` | INT | NOT NULL, CHECK `>= 1`, bất biến | |
| `icon_key` | VARCHAR(50) | NOT NULL | Tên icon lucide |
| `sort_order` | INT | NOT NULL | Thứ tự hiển thị; thứ tự phụ của Profile (§2.4) |
| `is_active` | BOOLEAN | NOT NULL, DEFAULT TRUE | §5.4 |
| `version` | BIGINT | NOT NULL, DEFAULT 0 | Optimistic lock Admin (§8.4). **Chưa có:** thêm bằng `migration_badges_v2.sql` + `@Version`, bắt buộc trước khi phát hành P2 (§10) |
| `created_at`, `updated_at` | DATETIME(6) | NOT NULL | |

`UNIQUE(metric, threshold)` chặn hai badge cùng điều kiện, tính cả badge đang tắt.

### 7.2. `user_badges` (append-only trừ `seen_at`, kế thừa `BaseCreatedAtEntity`)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL | |
| `badge_id` | BIGINT | FK → `badges(id)`, NOT NULL | |
| `seen_at` | DATETIME(6) | NULL | NULL = chưa xem (§2.3) |
| `created_at` | DATETIME(6) | NOT NULL | = `earnedAt` (giống `user_daily_missions.created_at` = assigned at) |

`UNIQUE(user_id, badge_id)`; index `(badge_id)` cho FK.

### 7.3. `user_badge_word_credits` (append-only, kế thừa `BaseCreatedAtEntity`)

| Column | Type | Ràng buộc | Mô tả |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → `users(id)`, NOT NULL | |
| `word_hash` | BINARY(32) | NOT NULL | SHA-256 của từ chuẩn hóa (cùng hàm với `user_word_credits`) |
| `created_at` | DATETIME(6) | NOT NULL | `actionInstant` của add-item |

`UNIQUE(user_id, word_hash)`. Không FK tới `topic_items`: credit tồn tại trọn đời, xoá topic/item không bị chặn và không làm giảm số đếm.

### 7.4. Retention, seed và đổi catalog

- **Retention:** cả ba bảng không purge theo tuổi; FK không cascade (RESTRICT); khi xoá tài khoản theo cùng policy ledger của user.
- **Seed:** `BadgeCatalogSeeder` chạy lúc khởi động (cùng mẫu `MissionCatalogSeeder`): **chỉ chèn** badge §4.1 có `code` chưa tồn tại; **không bao giờ** cập nhật row đã có.
- **Đổi catalog:** thêm badge mới ⇒ thêm vào seed (hoặc migration) cho tới khi có P3 (§10). Sửa `name`/`description`/`icon_key`/`sort_order`/`is_active` của badge đã có ⇒ qua migration SQL, hoặc Admin khi P2 phát hành (PD-BDG-13b). Seeder không ghi đè nội dung đã sửa. Không UPDATE `code`/`metric`/`threshold` (§5.3).
- **Schema thủ công:** script `snap-vocab-backend/migration_badges_v1.sql` (`CREATE TABLE IF NOT EXISTS`, idempotent) theo convention `migration_*.sql`.
- **Index:** đếm quiz dùng prefix `user_id` của `idx_quiz_user_created`. Chưa thêm index `(user_id, status)`; thêm khi đo thấy sync chậm.

---

## 8. API contract [Derived TD-BDG-01]

Base `/api`, JWT bắt buộc, envelope `RestResponse {statusCode, data, error}`. Lỗi kỹ thuật theo PLAT-01: `DataIntegrityViolationException` ⇒ 409 `CONCURRENT_MODIFICATION`; khác ⇒ 500 `INTERNAL_ERROR`. Thay đường dẫn `/gamification/badges` trong SS-13 bằng `/api/me/badges`, theo quy ước `/me/*` của wallet và daily-missions.

### 8.1. `GET /api/me/badges` — chỉ đọc

Không query, không ghi dữ liệu (`@Transactional(readOnly = true)`). Header `Cache-Control: no-store`. Trả badge active cộng badge inactive mà user đã đạt, sắp theo `sort_order`. Badge đủ điều kiện nhưng chưa sync vẫn hiện `earned = false`, kèm `currentValue >= threshold`.

```json
{
  "earnedCount": 1,
  "totalCount": 2,
  "unseenCount": 1,
  "badges": [
    {
      "id": 1,
      "code": "STREAK_7",
      "name": "Một Tuần Bền Bỉ",
      "description": "Hoàn thành ít nhất 1 quiz mỗi ngày, 7 ngày liên tiếp",
      "metric": "LONGEST_STREAK_DAYS",
      "threshold": 7,
      "currentValue": 9,
      "iconKey": "flame",
      "sortOrder": 10,
      "earned": true,
      "earnedAt": "2026-10-07T03:15:00Z",
      "seen": false
    },
    {
      "id": 2,
      "code": "STREAK_30",
      "name": "Tháng Không Nghỉ",
      "description": "Hoàn thành ít nhất 1 quiz mỗi ngày, 30 ngày liên tiếp",
      "metric": "LONGEST_STREAK_DAYS",
      "threshold": 30,
      "currentValue": 9,
      "iconKey": "flame",
      "sortOrder": 11,
      "earned": false,
      "earnedAt": null,
      "seen": null
    }
  ]
}
```

`totalCount` = số phần tử `badges`; `earnedCount` = số phần tử `earned = true`. `seen` = null khi chưa đạt.

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 500 | `INTERNAL_ERROR` | Lỗi kỹ thuật |

### 8.2. `POST /api/me/badges/sync` — ghi nhận + trả view

Không body. Chạy §6.1, trả **cùng shape** §8.1. Idempotent: gọi lặp lại không cấp thêm gì khi metric không đổi; không cần `Idempotency-Key`.

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 409 | `CONCURRENT_MODIFICATION` | Vi phạm UNIQUE khi cấp (§6.1); rollback, retry an toàn |
| 500 | `INTERNAL_ERROR` | Lỗi kỹ thuật khác |

### 8.3. `POST /api/me/badges/seen`

Body `{ "badgeIds": [1, 5] }` (1–50 id). Set `seen_at = now` cho các row `user_badges` của user có `badge_id` trong danh sách và `seen_at IS NULL`. Id không thuộc user, chưa đạt hoặc đã xem ⇒ bỏ qua. Idempotent. Gửi danh sách id thay vì "đánh dấu tất cả" để badge vừa cấp sau lần đọc không bị đánh dấu đã xem khi chưa hiện.

Response 200 (ví dụ): `{ "unseenCount": 0 }`. `unseenCount` là số badge chưa xem còn lại của user sau cập nhật; có thể lớn hơn 0 nếu body chỉ đánh dấu một phần hoặc badge mới được cấp đồng thời.

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 400 | `VALIDATION_ERROR` | Thiếu `badgeIds`, rỗng, quá 50 phần tử, id không phải số dương |

### 8.4. Admin (`/api/admin/badges`) [Derived PD-BDG-13a–d]

Không có trong release đầu (PD-BDG-10). Release tiếp theo phát hành theo bậc (§10). `ROLE_ADMIN` (áp sẵn cho `/api/admin/**`). Không PUT, không DELETE. Theo mẫu [daily_mission.md](./daily_mission.md) §8.3 nhưng **không khóa toàn catalog**: Nhiệm vụ cần khóa vì ràng buộc đúng 5 nhiệm vụ giữa các row; Badges chỉ có `UNIQUE` do DB giữ, nên `version` theo row là đủ.

| Bậc | Endpoint | Nội dung |
| :--- | :--- | :--- |
| P1 | `GET /api/admin/badges` → `AdminBadgeDTO[]` | Toàn catalog kể cả đang tắt, theo `sort_order, id`; không phân trang |
| P1 | `GET /api/admin/badges/{id}` → `AdminBadgeDTO` | Làm mới, đối chiếu sau lỗi mạng |
| P2 | `PATCH /api/admin/badges/{id}` → `AdminBadgeDTO` | Nội dung `{ version, name?, description?, iconKey?, sortOrder? }` hoặc bật/tắt `{ version, isActive }`; client gửi riêng hai loại |
| Nên có | Ước tính cấp bù (PD-BDG-13d) | Chỉ đọc, xem dưới; đường dẫn chốt khi làm |
| P3 | `POST /api/admin/badges` → 201 `AdminBadgeDTO` | Luôn tạo `is_active = false`; contract chốt khi mở P3 |

`AdminBadgeDTO { id, code, name, description, metric, threshold, iconKey, sortOrder, isActive, version, earnerCount, createdAt, updatedAt }`:

- `version`: `null` khi chưa có cột (§7.1). Admin ẩn thông tin phiên bản khi `null`, không hiện giá trị giả.
- `earnerCount`: số row `user_badges` theo `badge_id` (dùng index `(badge_id)`); `null` khi chưa hỗ trợ ⇒ Admin ẩn cột, không hiện `0`.

**PATCH (P2):**

- Có `code`, `metric` hoặc `threshold` ⇒ 409 `FIELD_LOCKED` (`details.fields`), không ngoại lệ (§5.3).
- `version` bắt buộc; khác giá trị hiện tại ⇒ 409 `VERSION_CONFLICT` (`details.currentVersion`).
- Server trim `name` (1–100 ký tự) và `description` (1–255) trước khi validate và lưu; `sortOrder` số nguyên; `iconKey` ∈ {`flame`, `clipboard-check`, `target`, `book-open`, `award`} (bắt buộc từ P2).
- Sửa nội dung không chạm `user_badges` (`created_at`, `seen_at`) và không tạo thông báo "badge mới" (PD-BDG-13b).
- Bật/Tắt không tạo hay xoá `user_badges` có sẵn. Hiệu lực tính theo lúc sync đọc catalog (§10).

| HTTP | `error` | Khi nào |
| :--- | :--- | :--- |
| 400 | `VALIDATION_ERROR` | Sai kiểu, thiếu trường, vi phạm độ dài, `iconKey` không thuộc danh sách (`fieldErrors`) |
| 404 | `BADGE_NOT_FOUND` | `id` không tồn tại |
| 409 | `FIELD_LOCKED` | Body chứa `code`/`metric`/`threshold` |
| 409 | `VERSION_CONFLICT` | `version` lệch |
| 409 | `BADGE_CODE_EXISTS`, `BADGE_THRESHOLD_EXISTS` | Chỉ P3: trùng `code` hoặc trùng `(metric, threshold)`, tính cả badge đang tắt |

**Ước tính cấp bù (PD-BDG-13d):** trả `{ eligibleCount, calculatedAt, version }`. `eligibleCount` = số tài khoản `activated = true` có metric ≥ `threshold` và chưa có `user_badges` của badge này. Metric dùng chung định nghĩa với sync (`BadgeService.metrics`, `QuizRepository.countCompleted`/`countPerfect`, `user_badge_word_credits`), không viết điều kiện riêng. Truy vấn tổng hợp một lần (không đếm từng user), đo hiệu năng trước phát hành. Chỉ đọc, không cấp badge. Tính khi mở modal Bật, có thể làm mới. Copy: "Tại thời điểm kiểm tra, có N tài khoản đủ điều kiện và chưa nhận. Huy hiệu được cấp khi từng tài khoản đồng bộ; số thực tế có thể thay đổi." Lỗi hoặc chưa có API ⇒ hiện "Chưa có ước tính", vẫn cho xác nhận Bật; không hiện `0`.

---

## 9. Mobile contract (MH-GAME-03, MH-PROFILE) [Derived]

| Hạng mục | Contract |
| :--- | :--- |
| Query | `['badges']` → `GET /api/me/badges` (màn Achievements, Profile) |
| Sync | `syncBadges()` → `POST /api/me/badges/sync`; thành công ⇒ `setQueryData(['badges'], data)`. Gọi khi: mở app, app quay lại foreground (`AppState`), sau complete quiz thành công, sau add word thành công (gọi tường minh trong hai mutation, không qua `invalidateProgress`), vào màn Thành tựu, kéo làm mới trên màn Thành tựu. Không polling. Lỗi ⇒ im lặng, lần kích hoạt sau gọi lại |
| Gallery / detail | Theo §2.1–§2.2 và [achievements_ui.md](../ui/achievements_ui.md); chỉ đọc từ response, không tự tính đạt/chưa đạt; sắp xếp theo UI-BDG-07, không dùng thứ tự response |
| Watcher badge mới | `BadgeUnlockHost` ở `(app)/_layout.tsx`, theo dõi `['badges']` và route. Giữ trong bộ nhớ (theo phiên app) tập `shownIds`. Khi có badge `seen = false` mà id chưa thuộc `shownIds`, route hiện tại **không** thuộc denylist (`/study/quiz-session`, phiên flashcard, `/study/quiz-result`, `/profile/achievements`) và không có modal khác đang mở ⇒ hiện modal §2.3 cho các id đó, thêm chúng vào `shownIds`. Trên màn Thành tựu áp dụng UI-BDG-03 |
| Đánh dấu đã xem | Đóng modal ⇒ `POST /seen` với đúng các id vừa hiện ⇒ cập nhật cache. **POST lỗi:** không hiện lại modal trong phiên (vì id đã nằm trong `shownIds`); giữ các id vào hàng đợi `pendingSeenIds` và gửi lại ở lần sync thành công kế tiếp trong phiên. Đóng app khi còn lỗi ⇒ lần mở app sau, badge vẫn `seen = false` và **được hiện lại một lần** |
| Profile | 3 badge `earned` theo thứ tự §2.4 (`earnedAt` giảm dần, rồi `sortOrder` giảm dần); chấm đỏ shortcut khi `unseenCount > 0` |
| Loading / lỗi | Skeleton lưới khi chưa có data; lỗi GET ⇒ trạng thái lỗi + nút thử lại |
| Offline | Hiện từ cache, không tự cấp badge ở client |
| Component | `AchievementArtifact` theo [achievements_ui.md](../ui/achievements_ui.md) §3 (không dùng `AchievementBadge3D`); icon theo `iconKey` (lucide), lạ ⇒ `award` |

---

## 10. Admin configuration [Authoritative PD-BDG-10, PD-BDG-13a–d]

**Release đầu (PD-BDG-10):** catalog qua seed và migration (§7.4). Gỡ route/menu `snapAdmin/src/features/badges` khỏi bản release (giữ source prototype, giống Admin Missions mock). P1 hiện ở `snapAdmin/src/features/admin-badges`, dùng GET API thật; fixture chỉ còn phục vụ test. Module mặc định ẩn, cần `VITE_ADMIN_BADGES_ENABLED=true` và nghiệm thu của bậc trước khi phát hành.

**Release tiếp theo (PD-BDG-13a):** phát hành theo bậc; mỗi bậc phát hành khi backend bắt buộc có trên staging và nghiệm thu §12.5 của bậc đạt. Mốc phát hành cụ thể chốt theo khả năng kỹ thuật.

| Bậc | Admin làm được | Backend bắt buộc | Điều kiện phát hành |
| :--- | :--- | :--- | :--- |
| **P1** (MVP) | Xem toàn catalog theo chuỗi, chi tiết chỉ đọc, preview kệ | `GET` list + `GET {id}` (§8.4); nên có `earnerCount` | Thay fixture bằng API, có trạng thái tải/lỗi, kiểm bàn phím; chưa có `version` thì ẩn phiên bản. Được phát hành trước P2 |
| **P2** (MVP) | Sửa tên, mô tả, biểu tượng, thứ tự; Bật/Tắt | `PATCH`, cột `version` (§7.1), validate `iconKey` ở server | Nghiệm thu xung đột phiên bản, lỗi mạng, rời trang khi chưa lưu, sync đồng thời với Bật/Tắt |
| Nên có | Ước tính cấp bù trong modal Bật (PD-BDG-13d) | API ước tính (§8.4) | Không chặn P2 |
| **P3** (mở rộng) | Tạo huy hiệu, thêm mốc vào chuỗi | `POST` (§8.4) | Chỉ khi vận hành cần thêm mốc thường xuyên tới mức deploy là trở ngại |

**Thứ tự ưu tiên:** P1 bằng API thật → P2 và các kiểm tra an toàn → ước tính cấp bù → đánh giá nhu cầu P3.

**Sửa nội dung (PD-BDG-13b):**

- Cho sửa `name`, `description`, `icon_key`, `sort_order`, kể cả huy hiệu đã có người đạt. Thay đổi áp dụng khi app tải lại dữ liệu (`user_badges` không lưu bản sao nội dung).
- Không đổi điều kiện, quyền sở hữu, ngày đạt, `seen_at`; không phát "badge mới".
- Người sửa giữ nguyên ý nghĩa thành tựu và điều kiện thực tế (vd "hoàn thành quiz" không được viết thành "học flashcard"; mô tả không hứa ngưỡng khác). Cảnh báo mô tả trên Admin chỉ phát hiện thiếu số ngưỡng, không bảo đảm nội dung đúng; trách nhiệm thuộc người vận hành. Không cần PO duyệt từng lần sửa.
- `sort_order` có thể đổi vị trí kệ và thứ tự phụ trên Profile (UI-BDG-07); editor nói rõ phạm vi ảnh hưởng.

**Thêm mốc, ngừng cấp mốc cũ (PD-BDG-13c).** Ví dụ `QUIZ_10` → thêm `QUIZ_15`, bật nó, tắt `QUIZ_10`:

- Người đã có `QUIZ_10` giữ nó và có thể nhận thêm `QUIZ_15`; kệ có thể có 4 mốc.
- Người chưa nhận `QUIZ_10` không được cấp nó trong các lần sync đọc catalog sau khi Tắt.
- Người đã đủ 15 quiz được cấp bù `QUIZ_15` ở lần sync kế tiếp.
- Catalog và tổng số badge có thể khác giữa các nhóm người học.
- Không chuyển đổi, hợp nhất, thu hồi; thêm mốc không tự tắt mốc khác. Không có thao tác thay thế nguyên tử: Bật mới trước có khoảng cấp cả hai, Tắt cũ trước có khoảng thiếu mốc; Product chọn trình tự theo mục tiêu.
- Huy hiệu tạo nhầm và chưa bật: giữ tắt, sửa phần hiển thị để dùng đúng cặp (metric, threshold) đã có.
- Trước P3 chỉ thêm mốc qua seed/migration. Duyệt 13c không mở G9 (sửa logic khi tắt và chưa ai đạt) hay P3.

**Tắt và sync đồng thời (PO chấp nhận cho MVP):** sync khóa row `users` rồi đọc catalog không khóa. Mốc thời gian là lúc sync **đọc catalog**: sync đọc catalog sau commit Tắt không cấp; sync đã đọc trước đó vẫn có thể cấp một lần cho chính tài khoản đó. Copy Tắt: "Ngừng cấp trong các lần đồng bộ tiếp theo. Người đã đạt vẫn giữ huy hiệu." Muốn cam kết dừng tuyệt đối từ thời điểm commit phải làm G11 trước (sync đọc catalog bằng `FOR SHARE`, PATCH khóa row bằng `FOR UPDATE`; cần thử nghiệm bằng hai transaction).

**Hoãn, mỗi mục cần PD riêng:** G9 sửa `metric`/`threshold` khi tắt và chưa ai đạt (cần G11); lịch sử thay đổi; tra cứu badge theo người học; tỷ lệ đạt.

---

## 11. Chỉ số theo dõi

| Metric | Ý nghĩa |
| :--- | :--- |
| `badge_earned_total{code}` | Số lần cấp theo badge (đếm row `user_badges`) |
| `badge_unlock_rate{code}` | % Learner hoạt động đã đạt từng badge; phát hiện ngưỡng quá khó/quá dễ |
| `time_to_first_badge` | Từ lúc đăng ký tới badge đầu tiên |
| `badge_unseen_older_than_7d` | Badge chưa xem quá 7 ngày; cao ⇒ watcher hoặc `/seen` có vấn đề |
| `badge_sync_failures_total` | Sync lỗi 409/500; kỳ vọng ≈ 0 |

Bốn chỉ số đầu đo bằng truy vấn SQL trên `user_badges`; chỉ số cuối đọc từ log server. Chưa cần counter Micrometer cho MVP.

---

## 12. Acceptance criteria

### 12.1. Ghi nhận

- Learner mới hoàn thành quiz 5 câu đạt 100% ⇒ sync kế tiếp cấp `QUIZ_1` và `PERFECT_1`, `seen = false`.
- Quiz MATCHING đúng tất cả cặp ⇒ tính `PERFECT_QUIZ`; sai 1 cặp ⇒ chỉ tính `QUIZ_COMPLETED`.
- Quiz `CANCELLED`/`EXPIRED` không tăng metric nào.
- Làm lại quiz cùng topic, đạt 100% hai lần ⇒ `PERFECT_QUIZ` tăng 2 (PD-BDG-12).
- `longest_streak_days = 30`, streak hiện tại = 0 ⇒ sync cấp `STREAK_7` và `STREAK_30`.
- Một lần sync có thể cấp nhiều badge (backfill streak 100 ⇒ cả 3 badge streak).
- `GET /api/me/badges` không bao giờ tạo row `user_badges`: user đủ điều kiện nhưng chưa sync ⇒ GET trả `earned = false`, `currentValue >= threshold`.
- Hai sync song song của cùng user ⇒ mỗi badge chỉ có 1 row; cả hai trả 200, hoặc một bên 409 và retry trả 200.
- Badge `is_active = false` không được cấp mới; user đã đạt vẫn thấy nó trong response; `totalCount` của hai user có thể khác nhau.
- Badge mới seed sau release ⇒ user đủ điều kiện được cấp ở lần sync kế tiếp.
- Lỗi được inject trong sync ⇒ rollback, không có row dở; quiz và add-item không bị ảnh hưởng.

### 12.2. Từ khác nhau

- Thêm item quiz-ready "Cat" vào collection của mình ⇒ credit +1; thêm " cat " vào topic khác ⇒ không tăng.
- Item thiếu bản dịch hoặc topic không bind `TARGET_WORD`/`NATIVE_TRANSLATION` ⇒ không tăng.
- Fork/import collection ⇒ không tăng.
- Retry add-item tạo hai item cùng từ ⇒ credit chỉ +1.
- Xoá item/topic rồi thêm lại cùng từ ⇒ không tăng; số đếm không giảm.
- Add-item rollback ⇒ không có credit.
- Dữ liệu trước ngày bật tracking không được tính (PD-BDG-07).

### 12.3. Badge mới và đã xem

- Có 1 badge unseen ⇒ modal chi tiết; có ≥ 2 ⇒ một modal gộp.
- Đang ở màn làm quiz/flashcard ⇒ không hiện modal; rời màn ⇒ modal hiện.
- `POST /seen` với id đã đạt ⇒ `seen = true`, `unseenCount` giảm; gọi lại cùng body ⇒ 200, không đổi gì.
- Id chưa đạt hoặc của user khác ⇒ bỏ qua, không lỗi.
- Badge được cấp giữa lần đọc và lần POST (không nằm trong body) ⇒ vẫn `seen = false`.
- `POST /seen` lỗi ⇒ modal không hiện lại trong phiên; sync thành công kế tiếp gửi lại `/seen`; nếu app bị đóng trước đó thì lần mở sau modal hiện lại đúng một lần.
- Profile: ba badge cấp cùng lúc (cùng `earnedAt`) ⇒ thứ tự theo `sortOrder` giảm dần, giống nhau ở mọi lần mở.

### 12.4. Vòng đời

- Mất streak, xoá từ, xoá topic ⇒ không mất badge nào.
- Không có endpoint thu hồi hay xoá badge.
- Cấp badge không ghi `coin_transactions`/`experience_logs` (PD-BDG-04).
- Seeder chạy lại sau khi migration đã sửa `name` ⇒ `name` giữ giá trị của migration.

### 12.5. Admin (PD-BDG-13a–d)

**P1:**

- `GET /api/admin/badges` trả cả badge đang tắt, theo `sort_order, id`.
- `version = null` ⇒ Admin không hiện phiên bản; `earnerCount = null` ⇒ ẩn cột "Đã đạt", không hiện `0`.
- API lỗi ⇒ trạng thái lỗi + thử lại, không rơi về fixture.

**P2:**

- PATCH chứa `threshold` ⇒ 409 `FIELD_LOCKED`, row không đổi.
- Hai Admin cùng mở `version = 3`, người lưu sau ⇒ 409 `VERSION_CONFLICT` kèm `currentVersion = 4`.
- `iconKey = "rocket"` ⇒ 400 `VALIDATION_ERROR`.
- Sửa tên `QUIZ_1` khi learner A đã đạt ⇒ `GET /api/me/badges` của A trả tên mới; `earnedAt`, `seen` không đổi; không hiện modal "badge mới".
- Tắt X, commit lúc T ⇒ mọi sync đọc catalog sau T không cấp X; người đã có X vẫn thấy X.
- Bật X ⇒ learner đủ điều kiện nhận X ở lần sync kế tiếp.
- Ước tính cấp bù lỗi ⇒ modal Bật hiện "Chưa có ước tính", vẫn xác nhận được.

---

## 13. Đồng bộ tài liệu dẫn xuất (CLEAN-BDG-01) — đã làm 2026-10-07

| File | Sửa |
| :--- | :--- |
| [specs.md](../spec/specs.md) FR-09.04 | Trỏ tới file này |
| [phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md) F-GAME-05 | Đổi ví dụ "100 lượt scan có lưu từ" sang catalog §4.1; ghi badge scan là Deferred |
| [buss_mainflow.md](../spec/buss_mainflow.md) BF-12 | Bước 1: "đạt điều kiện ⇒ trao Badge" ⇒ "ghi nhận khi app đồng bộ (§6)"; bước 2: "In-app Notification" ⇒ "đánh dấu badge mới (§2.3)" |
| [buss_mainflow.md](../spec/buss_mainflow.md) BF-13 | Bỏ "Badge mới" khỏi push trong MVP; bỏ "Mission hoàn thành" (trái [daily_mission.md](./daily_mission.md) §1.2) |
| [phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md) MH-GAME-03, MH-PROFILE | Bỏ "rarity"; profile "3–5 badges" ⇒ "3 badge gần nhất" |
| [phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md) SS-13 + bảng mapping liên phân hệ | Entity `Badge`/`UserBadge` theo §7 + `UserBadgeWordCredit`; API `/me/badges` + `/sync` + `/seen`, bỏ DELETE; `BadgeService` = sync §6; bỏ `RewardEventHandler` (lớp event đã bị bỏ); Gamification → Notification: không có trong MVP |
| [database.md](../db/database.md) §4 | Thêm §7.1–§7.3 |
| [sa/server.md](../sa/server.md), [sa/techstack.md](../sa/techstack.md), [sa/sa.md](../sa/sa.md) | Bỏ "Badges: chưa thiết kế schema"; cập nhật danh sách bảng |
| [snap-vocab-backend/README.md](../../snap-vocab-backend/README.md) | Bỏ "not yet specified", trỏ tới file này |
| [docs/README.md](../README.md) | Thêm file này vào bảng tài liệu (ACTIVE / AUTHORITATIVE) |
| `snapAdmin/src/domains/badges`, `features/badges` | Gỡ khỏi bản release (PD-BDG-10); khi làm Admin thật thì viết lại domain type theo §4.3, §7 |
| `snapVocab-frontend` (prototype) | Không sửa (DEPRECATED) |

---

## 14. Future enhancement (ngoài MVP, mỗi mục cần quyết định riêng)

- Badge scan (DEP-BDG-01), badge từ đã thuộc (DEP-BDG-02).
- Ghi nhận lúc hành động (thay thế §6.1 làm đường chính); bắt buộc trước khi có tính năng xoá quiz.
- Mục trong hộp thông báo / push khi đạt badge, sau khi SS-15 có service; cần ghi nhận lúc hành động và chống gửi trùng theo `user_badges.id`.
- Mở rộng "ngày học" sang flashcard/SRS review khi các luồng đó có tracking (PD-BDG-11).
- Admin Huy hiệu: P1/P2 đã duyệt cho release tiếp theo (§10). Còn mở: P3 tạo/thêm mốc, G9, G11, lịch sử thay đổi, thống kê tỷ lệ mở khóa.
- Thưởng XP/Coin (cần mở lại PD-BDG-04).
- Ghim badge nổi bật lên profile.

---

## 15. Quyết định MVP (PO duyệt 2026-10-07)

- 12 badge cố định, 4 metric; không tier, không bí mật, không danh hiệu, không gems.
- Nhóm "từ" đếm số từ khác nhau từng tự thêm (C); nhóm streak tính ngày có quiz hoàn thành.
- Nhận một lần, không thu hồi, không thưởng.
- Ghi nhận khi app đồng bộ qua `POST /api/me/badges/sync`; `GET` chỉ đọc; không hook cấp badge vào quiz.
- Badge mới: modal gộp + chấm đỏ, không ngắt phiên học; không push, không hộp thông báo.
- Release đầu chỉ seed/migration, không Admin CRUD.
- Release tiếp theo (PO duyệt 2026-10-08, PD-BDG-13a–d): Admin Huy hiệu P1 chỉ đọc → P2 sửa hiển thị + Bật/Tắt; ước tính cấp bù nên có; P3 là mở rộng (§10).
