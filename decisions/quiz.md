# Architecture Decision: Quiz — Incremental Grading & SemanticRole Resolution

> **Trạng thái:** APPROVED (Human Architectural Decision, 2026-09-23).
> **Implementation:** Backend Quiz **IMPLEMENTED** (2026-09-23: `QuizController`, `QuizService`, `QuizContentResolver`, `QuizGenerator`, `SemanticBindingMigration`). Mobile Quiz (`SnapVocab/src/features/quiz`) **IMPLEMENTED**: gọi thật `/api/me/quizzes` · `/api/questions/{id}/check` · `.../rounds/{roundNo}/attempts` · `complete` · `cancel` · `result` · history; đúng/sai lấy từ Backend, không còn mock/chấm client. `durationSeconds` (D15) và TTL 24h + `QuizStatus.EXPIRED` (D17) đã implement cả hai phía. Xem §7.
> **Traceability:** FR-06 ([specs.md](../spec/specs.md)) → BF-09 ([buss_mainflow.md](../spec/buss_mainflow.md)) → SS-10 ([phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md)) → API (`QuizController`) → DB ([database.md](../db/database.md) §Quiz) → Screen MH-LEARN-02/03/04 ([phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md)).

---

## 1. Quyết định

| # | Quyết định | Nội dung canonical |
| :--- | :--- | :--- |
| D1 | API base path | `/api`. Không dùng `/api/v1`; không tự version hóa API. |
| D2 | Quiz data resolution | Resolve dữ liệu qua `TemplateField.semanticRole`. Không hard-code `SchemaAttribute.name` (`word`, `meaning`, `translation`, `example`, `audio`...). |
| D3 | Grading lifecycle | **Incremental check**: chấm và phản hồi ngay sau mỗi interaction. Không có mô hình "làm hết → submit toàn bộ → mới biết đúng/sai". |
| D4 | MCQ | Chọn một đáp án → check câu đó → đúng/sai ngay → câu tiếp theo. |
| D5 | FILL_BLANK | Nhập đáp án → submit câu hiện tại → đúng/sai ngay → câu tiếp theo. |
| D6 | MATCHING | **Per-pair attempt**: chọn trái → chọn phải → check cặp đó ngay → đúng/sai ngay. Không check theo cả round. |
| D7 | Complete | `complete` chỉ **finalize** session (tổng hợp kết quả, side effects cuối session). Không phải thời điểm chấm đầu tiên. |
| D8 | Mode vs Direction | `QuizMode` = `MCQ` \| `MATCHING` \| `FILL_BLANK`. `QuizDirection` = `EN_VI` \| `VI_EN`. Hai khái niệm tách biệt. |
| D9 | LISTENING | `LISTENING` là `Template.code` của flashcard, **không phải** `QuizMode`. Không có LISTENING Quiz Mode khi chưa có requirement mới. |
| D10 | NATIVE_TRANSLATION | Schema mặc định: `main.translation` (nghĩa tiếng Việt của từ) → `NATIVE_TRANSLATION`; `meaning` (định nghĩa tiếng Anh) giữ `DEFINITION`. **Không** fallback `NATIVE_TRANSLATION` → `DEFINITION`. |
| D11 | definition_translation | Là bản dịch tiếng Việt của DEFINITION; **không** map vào `NATIVE_TRANSLATION`, **không** tạo role mới. Quiz không dùng. |
| D12 | Eligibility | Item chỉ hợp lệ cho một câu hỏi khi mọi role mà mode/direction đó yêu cầu resolve ra giá trị non-blank. Pool sau lọc không đủ → Create Quiz lỗi domain rõ ràng. |
| D13 | FILL_BLANK | Prompt = `EXAMPLE_SENTENCE` với vị trí xuất hiện `TARGET_WORD` bị che; đáp án = `TARGET_WORD`. Không dùng `NATIVE_TRANSLATION`/`DEFINITION` làm đáp án; không có biến thể đáp án tiếng Việt; direction không ảnh hưởng chấm điểm. |
| D14 | Quiz size | `requestedCount` ∈ {5, 10, 20} (Quiz Settings mobile). Quiz luôn có đúng số unit đã chọn; không có dải 4..50. |
| D15 | Quiz duration | Quiz Play **MAY** hiển thị stopwatch đếm lên phía client: UX-only, không ảnh hưởng chấm điểm/XP, không persist. Khi quiz COMPLETED, Backend tính `durationSeconds = completedAt - createdAt` và expose trong `QuizResultDTO`; MH-LEARN-04 hiển thị giá trị này. Client **không** tự tính duration, **không** gửi stopwatch lên server. |
| D16 | Review vs Retry mistakes | **Review mistakes: đã hỗ trợ** — Result hiển thị `prompt`, `learnerAnswer`, `correctAnswer`, `outcome = INCORRECT` từ `QuizResultDTO.items`, không tạo session mới, không cần API mới. **Retry Mistakes** (tạo QuizSession chỉ gồm câu sai) **defer M4**; không được expose thành CTA active cho tới khi có quy tắc question-count / reward / session-generation. |
| D17 | Session expiration | Session `IN_PROGRESS` chỉ resume được trong **24h kể từ `createdAt`**. Lifecycle: `IN_PROGRESS → COMPLETED` (learner nộp bài) \| `→ CANCELLED` (learner chủ động bỏ) \| `→ EXPIRED` (hệ thống đóng vì quá TTL). `CANCELLED` **không** dùng thay `EXPIRED`. Lazy expiration (không cron/scheduler); expired không resume/answer/complete/cancel → `QUIZ_SESSION_EXPIRED` (HTTP **410 Gone**). |
| D18 | Resume vs Create | **Làm tiếp** (Quiz History) = resume đúng `QuizSession` cũ, không sinh lại câu hỏi. **Bắt đầu** (Quiz Setup) = luôn tạo Quiz mới (`id` khác); **không** ngầm hiểu "Trắc nghiệm 5 câu" là bài cũ. Khi Setup phát hiện session `IN_PROGRESS` còn hạn **cùng ngữ cảnh** (`topicId` + `mode` + `direction` + `requestedCount`) thì hỏi learner: *Tiếp tục bài cũ* → resume; *Bắt đầu bài mới* → `CANCELLED` bài cũ rồi create. Không giữ đồng thời hai `IN_PROGRESS` cùng ngữ cảnh. |
| D19 | Topic Selection (Hybrid) | Nguồn Quiz là thực thể `Topic` (`topicId`). Tuyệt đối **không** dùng `deckId` làm alias và **không** fallback giả `topicId = 1`. Quiz Setup hỗ trợ luồng hybrid: nếu nhận `topicId` từ màn trước (Topic Detail / bài học dở) thì preselect, nhưng learner vẫn có thể đổi Topic bất cứ lúc nào qua Bottom Sheet; nếu mở từ Learn Hub thì chưa chọn Topic, CTA Start Quiz bị disabled cho tới khi chọn Topic hợp lệ. |

> D10–D13: Human Architectural Decision 2026-09-23, dựa trên dữ liệu production đã kiểm chứng (15,663 bản ghi schema mặc định: `word`, `meaning`, `translation`, `definition_translation`, `phonetic` đủ 100%; 40,926 example `content`). D19: Human Architectural Decision 2026-09-24, chuẩn hoá domain model Quiz Source theo thực thể Topic.

---

## 2. Canonical Quiz Session Lifecycle

```text
Create Quiz (POST /api/me/quizzes)
    ↓
Backend generate & trả Quiz session (câu hỏi / cặp ghép, không kèm đáp án đúng)
    ↓
Learner làm từng interaction — mỗi interaction được Backend chấm ngay:
    MCQ         → check từng câu  → Correct / Incorrect → câu tiếp
    FILL_BLANK  → check từng câu  → Correct / Incorrect → câu tiếp
    MATCHING    → check từng pair → Correct / Incorrect → tiếp tục ghép
    ↓
Backend ghi nhận kết quả từng interaction (persistence contract: database.md §Quiz)
    ↓
Complete Quiz (POST /api/me/quizzes/{quizId}/complete)
    ↓
Finalize session: tổng hợp summary từ các kết quả đã chấm
    ↓
Result (GET /api/me/quizzes/{quizId}/result)
    ↓
XP / Coin / Mission / Progress — theo requirement hiện hành (FR-06.07, FR-09)
```

Quy tắc:

1. Mọi kết quả đúng/sai do **Backend** quyết định. Client không tự chấm, không gửi điểm.
2. Payload `complete` không chứa đáp án. Backend chỉ tổng hợp từ các check đã ghi nhận.
3. Thoát quiz giữa chừng qua modal thoát: session bị hủy (`cancel`), không lưu draft (FR-06, [specs.md](../spec/specs.md)). Session dở dang không qua modal (app bị đóng, back cứng) giữ `IN_PROGRESS` và resume được trong 24h, sau đó `EXPIRED` (D17).
4. Kết quả Quiz không cập nhật tham số FSRS.
5. Retry cùng request (mạng lỗi) không được cộng trùng kết quả/XP — dùng `Idempotency-Key` (xem API contract).

### 2.0. Session state machine (D17)

```text
IN_PROGRESS -> COMPLETED    learner nộp bài (complete)
IN_PROGRESS -> CANCELLED    learner chủ động bỏ (cancel qua modal thoát)
IN_PROGRESS -> EXPIRED      hệ thống đóng session dở dang sau TTL 24h kể từ createdAt
```

- TTL cố định `createdAt + 24h`; **không** dùng `lastActivityAt` ở phiên bản này.
- **Lazy expiration**: kiểm tra ngay trong request (resume / check / pair attempt / complete / cancel / result / history) và persist `EXPIRED` vì expiration là irreversible. Không có cron job.
- Mọi tương tác với session đã expired trả `QUIZ_SESSION_EXPIRED` → HTTP **410 Gone**.
- History (`GET /api/me/quizzes`) không được trình bày session quá 24h như còn resume được; expire batch trước khi map DTO, client không phải là source of truth của status.

### 2.0.1. Resume vs Create (D18)

```text
Thoát app đột ngột → Quiz giữ IN_PROGRESS
    ├── History → "Làm tiếp"  → resume Quiz cũ (cùng quizId, không generate lại)
    └── Setup   → "Bắt đầu"   → có IN_PROGRESS còn hạn cùng ngữ cảnh?
                                  không → create Quiz mới
                                  có    → hỏi learner
                                            Tiếp tục bài cũ → resume
                                            Bắt đầu bài mới → cancel bài cũ + create
```

- Ngữ cảnh = `topicId` + `mode` + `direction` + `requestedCount`. "5 câu trắc nghiệm" của Topic A **không** resume nhầm sang Topic B.
- Phát hiện nằm ở client (Quiz Setup đọc trang đầu `GET /api/me/quizzes`, nơi backend đã expire batch theo D17) — không thêm endpoint. Nếu cần chặt hơn thì bổ sung filter `status` cho history hoặc gate ngay trong `POST /api/me/quizzes`.
- "Bắt đầu bài mới" đi qua `POST .../cancel` sẵn có; create chỉ chạy sau khi cancel thành công.

### 2.1. MCQ

```text
Hiển thị câu hỏi → Learner chọn 1 đáp án → POST check → Correct / Incorrect → UI feedback → Next question
```

Mỗi câu được check đúng một lần; check lại câu đã chấm là lỗi conflict.

### 2.2. FILL_BLANK

```text
Learner nhập đáp án → Submit câu hiện tại → POST check → Correct / Incorrect → UI feedback → Next question
```

Không gom đáp án Fill Blank để nộp cuối Quiz. Quy tắc so khớp: trim + không phân biệt hoa/thường (F-QUIZ-04).

### 2.3. MATCHING — Per-pair attempt

```text
Select left item → Select right item → POST pair attempt → Correct / Incorrect ngay → Continue matching
```

- **Đúng:** cặp được xác nhận và khóa (không chọn lại được).
- **Sai:** UI báo sai ngay; cặp không bị khóa; Learner tiếp tục chọn cặp khác hoặc thử lại.
- Backend không tiết lộ cặp đúng khi trả kết quả sai.
- Các cặp được hiển thị theo nhóm (`roundNo`) để vừa màn hình; `roundNo` chỉ là nhóm hiển thị, **không** phải đơn vị chấm.

Quy tắc Matching v1 (quyết định Q4, 2026-09-23):
- 5 cặp mỗi round hiển thị (`PAIRS_PER_ROUND = 5`): 5 → [5], 10 → [5,5], 20 → [5,5,5,5]; thử lại không giới hạn trong session.
- Điểm theo **lần thử đầu**: đúng ngay → `CORRECT`; từng sai trước khi ghép đúng → `INCORRECT`.
- Điểm (`outcome`) và trạng thái hoàn tất (`resolved`) là hai khái niệm riêng; `complete` kiểm tra `resolved`.

---

## 3. SemanticRole Resolution

### 3.1. Chuỗi resolve canonical

```text
Topic
 → Topic.activeTemplate                          (topics.active_template_id)
 → Template.elements   (TemplateElement, type = FIELD)
 → TemplateField        (template_fields)
 → TemplateField.semanticRole = <ROLE>
 → TemplateField.schemaAttribute                 (template_fields.schema_attribute_id)
 → TopicItemAttributeValue của TopicItem có attribute = SchemaAttribute đó
```

Quiz Engine hỏi "giá trị có vai trò `TARGET_WORD` của TopicItem này là gì", **không** hỏi "giá trị của attribute tên `word`". Một Topic không bắt buộc phải có attribute tên `word`/`meaning`/`translation`/`example`.

`CardSide` (`FRONT`/`BACK`) không tham gia resolve: Quiz chỉ dựa vào `semanticRole`.

### 3.2. SemanticRole canonical

Chỉ dùng enum thực tế `vn.ptit.snapvocab.domain.enumeration.SemanticRole`:

`TARGET_WORD` · `DEFINITION` · `NATIVE_TRANSLATION` · `EXAMPLE_SENTENCE` · `AUDIO` · `IMAGE`

Không thêm role mới (ví dụ không tạo `PHONETIC`) khi chưa có quyết định riêng.

### 3.3. Vai trò theo chiều hỏi (QuizDirection)

| Phía | Role |
| :--- | :--- |
| Phía tiếng Anh | `TARGET_WORD` |
| Phía tiếng Việt | `NATIVE_TRANSLATION` — không fallback (D10) |

| Mode | Direction | Role bắt buộc | Prompt | Đáp án đúng |
| :--- | :--- | :--- | :--- | :--- |
| MCQ / MATCHING | `EN_VI` | `TARGET_WORD`, `NATIVE_TRANSLATION` | `TARGET_WORD` | `NATIVE_TRANSLATION` |
| MCQ / MATCHING | `VI_EN` | `TARGET_WORD`, `NATIVE_TRANSLATION` | `NATIVE_TRANSLATION` | `TARGET_WORD` |
| FILL_BLANK | (không ảnh hưởng chấm) | `TARGET_WORD`, `EXAMPLE_SENTENCE` | `EXAMPLE_SENTENCE` có chỗ trống thay cho `TARGET_WORD` | `TARGET_WORD` |

- `DEFINITION` và `definition_translation` không tham gia Quiz ở phase này (D10, D11).
- `AUDIO`, `IMAGE`: dữ liệu bổ trợ tùy chọn; không tạo Quiz Mode riêng.

### 3.4. Eligibility khi thiếu dữ liệu (D12)

| Tình huống | Quy tắc |
| :--- | :--- |
| Thiếu `TARGET_WORD` | Loại item khỏi mọi Quiz |
| Thiếu `NATIVE_TRANSLATION` | Loại khỏi MCQ/MATCHING (`EN_VI`, `VI_EN`); không fallback `DEFINITION` |
| Thiếu `EXAMPLE_SENTENCE` | Loại khỏi FILL_BLANK |
| Thiếu `AUDIO` / `IMAGE` | Không ảnh hưởng eligibility |
| Pool sau lọc không đủ | Create Quiz thất bại `QUIZ_INSUFFICIENT_ELIGIBLE_ITEMS` (`requestedCount`, `eligibleCount`); không tạo Quiz nhỏ hơn, không sinh câu hỏi lỗi, không đổi role ngầm |
| Role singleton (`TARGET_WORD`, `NATIVE_TRANSLATION`, `DEFINITION`) bind vào >1 attribute trong active template | Template ambiguous → Create Quiz lỗi `SEMANTIC_ROLE_AMBIGUOUS` (Q2); không chọn field đầu tiên |
| `EXAMPLE_SENTENCE` bind nhiều field / nhóm lặp | Mọi câu non-blank đều là ứng viên (Q2) |
| Role singleton có >1 giá trị khác nhau cho cùng một item (attribute trong nhóm lặp) | Item không hợp lệ (không đoán giá trị) |
| `phonetic` có cần SemanticRole riêng | **REQUIRES PRODUCT DECISION** |

> **Ràng buộc binding hiện trạng** (seed `DataInitializer`): không template nào wire `main.translation` vào `NATIVE_TRANSLATION`; `EXAMPLE_SENTENCE` được wire vào `meaning.example`, **không** phải `examples.content` (nơi chứa 40,926 câu ví dụ). Kiến trúc `TemplateField` hiện không có cách khai báo binding ngữ nghĩa mà không render field đó (chỉ có `hideIfEmpty`). Xem Implementation Plan — Phase 0.

---

## 4. Mode & Direction

| Khái niệm | Giá trị | Ghi chú |
| :--- | :--- | :--- |
| `QuizMode` | `MCQ`, `MATCHING`, `FILL_BLANK` | Không có `LISTENING`, không có `EN_VI`/`VI_EN` |
| `QuizDirection` | `EN_VI`, `VI_EN` | Không phải mode |
| `Template.code` | `STANDARD`, `LISTENING`, `REVERSE` (seed hiện có) | Chế độ hiển thị flashcard, không phải Quiz Mode |

---

## 5. Phạm vi không thuộc quyết định này

- Chính sách XP/Coin cụ thể cho Quiz: theo [specs.md](../spec/specs.md) FR-09 và [daily_mission.md](./daily_mission.md).
- Tên bảng/entity Quiz: PLANNED, xem [database.md](../db/database.md) §Quiz.
- Tính năng skip câu / skip cặp: **TBD** — chưa có requirement canonical.
- **Retry Mistakes session** (quiz mới chỉ gồm câu sai): defer **M4** (D16). Cần quyết định question-count (rule hiện tại {5, 10, 20} không cho 2–4 câu), reward và cách sinh session trước khi implement.
- **XP:** `complete` ghi `10 XP × số câu đúng` qua `XpService.award(QUIZ, eventKey = QUIZ:{quizId})` — XP Booster ×2 xét tại thời điểm complete; `QuizResultDTO.xp {baseAmount, bonusAmount, amount}` (null nếu 0 câu đúng). **Coin:** chưa kích hoạt, UI hiển thị `+0 Coin`.

---

## 6. Tài liệu đã thay thế

Các mô tả sau trong tài liệu cũ **không còn hiệu lực**: `POST /quizzes/{id}/submit` (nộp toàn bộ bài), `POST .../rounds/{roundNo}/check` (chấm cả round Matching), base path `/api/v1`, resolver theo tên attribute (`word`, `meaning`, `example`...). Nhánh backup `backup/pre-reset-docs` (4722a10) chứa proposal Quiz cũ — chỉ tham khảo, không phải implementation.

---

## 7. Implementation Gap / Follow-up

Không thực hiện code change trong phase tài liệu này. Các gap cần xử lý khi implement:

| # | Vị trí | Hiện trạng | Cần đổi theo canonical |
| :--- | :--- | :--- | :--- |
| G1 | `snap-vocab-backend` | **DONE** 2026-09-23 | — |
| G2 | `SnapVocab/src/features/quiz/api/quiz-api.ts` | **DONE** — dùng `/api/...` | — |
| G3 | `quiz-api.ts` / `types.ts` | **DONE** — per-pair attempt (`POST .../rounds/{roundNo}/attempts`) | — |
| G4 | `types.ts` | **DONE** — chỉ `MCQ` \| `MATCHING` \| `FILL_BLANK` | — |
| G5 | `quiz-session-screen.tsx`, `*-quiz-view.tsx`, `quiz-setup-screen.tsx` | **DONE** — gọi create/check/attempt/complete/cancel/result, đúng/sai từ Backend | — |
| G6 | `SnapVocab/src/features/learn/learning-state.ts` | Comment tham chiếu `/api/v1/learning-hub/summary` | Cập nhật về `/api/...` khi endpoint được định nghĩa |
| G7 | Seed `DataInitializer` + DB hiện có | Không wire `main.translation` → `NATIVE_TRANSLATION`; `EXAMPLE_SENTENCE` trỏ `meaning.example` thay vì `examples.content` | **DONE** (Q1): `template_fields.hidden` + binding ẩn do `SemanticBindingMigration` thêm cho DEFAULT_ENGLISH; fork chỉ REPORT mặc định |
| G8 | `SnapVocab/src/features/template-builder/mappers/backend-to-client.ts` | Field có `semanticRole = null` bị gán mặc định `TARGET_WORD`; lưu lại template sẽ tạo nhiều field `TARGET_WORD` | Giữ `null` khi round-trip |
