# SnapVocab Docs

Thư mục này là trung tâm tài liệu sản phẩm, kiến trúc và thiết kế dữ liệu cho SnapVocab — ứng dụng mobile-first hỗ trợ học từ vựng tiếng Anh thông qua nhận diện hình ảnh, flashcard, quiz, SRS và gamification.

## Nên đọc từ đâu?

Nếu mới vào dự án, nên đọc theo thứ tự sau:

1. [spec/specs.md](./spec/specs.md) — tài liệu SRS tổng quan và source of truth cho phạm vi hệ thống.
2. [spec/buss_mainflow.md](./spec/buss_mainflow.md) — các luồng nghiệp vụ end-to-end theo BF.
3. [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md) — backlog tính năng, priority và acceptance criteria.
4. [spec/phan_ra_phan_he_he_thong.md](./spec/phan_ra_phan_he_he_thong.md) — phân hệ hệ thống và trách nhiệm từng module.
5. [spec/phan_ra_man_hinh.md](./spec/phan_ra_man_hinh.md) — danh sách màn hình, nhóm màn hình và mapping với chức năng.
6. [sa/sa.md](./sa/sa.md) — kiến trúc hệ thống tổng thể.
7. [sa/techstack.md](./sa/techstack.md) — công nghệ sử dụng cho mobile, backend, admin, AI service và hạ tầng.
8. [db/database.md](./db/database.md) — mô hình dữ liệu và ràng buộc database.

## Quyết định canonical

Các tài liệu trong thư mục này cần bám theo các quyết định canonical dưới đây:

| Chủ đề             | Quyết định                                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------------------- |
| Source of truth    | [spec/specs.md](./spec/specs.md)                                                                                |
| AI pipeline        | Florence-2 zero-shot (OD + self-grounding; CLIP tùy chọn; không SAM)                                            |
| Actor              | Guest, Learner, Admin                                                                                           |
| Learning domain    | Collection (system/user) → Topic → TopicItem (mô hình EAV); đã lược bỏ hoàn toàn Deck, Note, Card, ReviewLog cũ |
| Flashcard Template | Template gắn với Topic, gồm TemplateElement và TemplateField mang SemanticRole                                  |
| SRS                | Thuật toán FSRS quản lý qua bảng fsrs_records cho từng cặp (user, topic_item)                                   |
| Milestone          | M1 Auth+Dict → M2 Scan → M3 Learning → M4 Game+Prod                                                             |
| Community scope    | Lược bỏ hoàn toàn chức năng community/study group; chỉ giữ leaderboard cá nhân                                  |

## Cấu trúc tài liệu

```text
docs/
├── README.md
├── db/
│   └── database.md
├── decisions/
│   ├── badges.md
│   ├── custom_card.md
│   ├── daily_mission.md
│   ├── quiz.md
│   └── shop.md
├── ui/
│   └── collection_creation_flow_ui.md
├── ke_hoach_phan_chia_cong_viec.xlsx
├── sa/
│   ├── sa.md
│   ├── server.md
│   └── techstack.md
└── spec/
    ├── specs.md
    ├── buss_mainflow.md
    ├── phan_ra_tinh_nang.md
    ├── phan_ra_phan_he_he_thong.md
    └── phan_ra_man_hinh.md
```

## Nhóm tài liệu

### 1. Product specification

- [spec/specs.md](./spec/specs.md): đặc tả yêu cầu hệ thống, phạm vi, actor, milestone, functional requirements và non-functional requirements.
- [spec/buss_mainflow.md](./spec/buss_mainflow.md): các business flow chính như đăng ký, đăng nhập, tra cứu từ điển, scan-to-vocabulary, lưu từ, flashcard, quiz, SRS, progress và gamification.
- [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md): backlog tính năng theo nhóm AUTH, RECOG, DICT, TOPIC, VOCAB, FLASH, QUIZ, SRS, PROGRESS, GAME, NOTIF, STORAGE, ADMIN.
- [spec/phan_ra_phan_he_he_thong.md](./spec/phan_ra_phan_he_he_thong.md): phân rã hệ thống theo subsystem/service và trách nhiệm triển khai.
- [spec/phan_ra_man_hinh.md](./spec/phan_ra_man_hinh.md): phân rã màn hình mobile/admin, mapping màn hình với flow và module.

### 2. System architecture

- [sa/sa.md](./sa/sa.md): kiến trúc tổng thể gồm Mobile App, Admin CMS, Backend API, AI Service, MySQL/MariaDB, Redis và Object Storage.
- [sa/techstack.md](./sa/techstack.md): stack công nghệ cho React Native/Expo, Spring Boot, FastAPI AI, Next.js Admin, Redis, Cloudflare R2/MinIO và Swagger/OpenAPI.
- [sa/server.md](./sa/server.md): môi trường local/staging/production, server-side deployment, API envelope, cấu hình backend, AI service và vận hành.

### 3. Database

- [db/database.md](./db/database.md): schema dữ liệu cho Identity, Dictionary, Topic, Flashcard/SRS, Daily Mission, Gamification, Recognition, Storage và các ràng buộc/index quan trọng.

### 4. Decision records

- [decisions/custom_card.md](./decisions/custom_card.md): thiết kế hệ thống Template cho Topic, thành phần TemplateElement và TemplateField mang SemanticRole ánh xạ thuộc tính EAV thành thẻ học.
- [decisions/quiz.md](./decisions/quiz.md): quyết định kiến trúc Quiz — base path `/api`, incremental grading (MCQ/Fill Blank từng câu, Matching từng cặp), resolve dữ liệu theo SemanticRole, QuizMode vs QuizDirection.
- [decisions/daily_mission.md](./decisions/daily_mission.md): thiết kế Daily Mission (5 nhiệm vụ/ngày, reward theo từng nhiệm vụ), mission pool, event tracking và reward idempotency.
- [decisions/badges.md](./decisions/badges.md): Badges canonical — catalog 12 badge / 4 metric, ghi nhận khi app đồng bộ, nhận một lần không thu hồi, không thưởng, schema, API, mobile. PD-BDG duyệt 2026-10-07.
- [decisions/shop.md](./decisions/shop.md): Shop canonical — `THEME`/`AVATAR_FRAME`/`XP_BOOSTER`, UserItem ownership + equip, XP Booster ×2, purchase idempotency/concurrency, Admin boundary, API, schema.

### 5. Design

- [snapVocab-frontend/docs/design.md](../snapVocab-frontend/docs/design.md): design tokens/component (nguồn design duy nhất, được [shop.md](./decisions/shop.md) tham chiếu). File nằm trong thư mục prototype nhưng nội dung là ACTIVE.

## Milestone triển khai

| Milestone | Tên                                 | Mục tiêu chính                                                               | Tài liệu liên quan                                                                                                                                         |
| --------- | ----------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M1        | Core Auth & Vocabulary Lookup       | Auth, profile, dictionary, topic, saved vocabulary, flashcard cơ bản         | [spec/specs.md](./spec/specs.md), [spec/buss_mainflow.md](./spec/buss_mainflow.md)                                                                         |
| M2        | Camera/Object Recognition MVP       | Camera/gallery, upload, AI recognition, word mapping, save from scan         | [sa/sa.md](./sa/sa.md), [sa/server.md](./sa/server.md)                                                                                                     |
| M3        | Learning Engine                     | Custom card, quiz, SRS, progress, notification                               | [decisions/custom_card.md](./decisions/custom_card.md), [decisions/quiz.md](./decisions/quiz.md), [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md) |
| M4        | Gamification & Production Readiness | Mission, XP, coin, badge, shop, leaderboard, admin CMS, production hardening | [decisions/daily_mission.md](./decisions/daily_mission.md), [decisions/badges.md](./decisions/badges.md), [decisions/shop.md](./decisions/shop.md), [sa/server.md](./sa/server.md) |

## Trạng thái tài liệu (audit 2026-10-03)

Khi hai tài liệu khác nhau, tài liệu AUTHORITATIVE thắng; DERIVED phải sửa theo. Tài liệu ACTIVE không được tham chiếu SUPERSEDED/HISTORICAL như nguồn hiện hành.

| Tài liệu | Trạng thái | Ghi chú |
| :--- | :--- | :--- |
| `docs/decisions/daily_mission.md` | ACTIVE / AUTHORITATIVE (rule Daily Mission) | Daily Mission: scope, catalog, progress đồng bộ, DB, API, mobile, admin. PD-DM-01 duyệt 2026-10-04; còn mở không chặn: PD-DM-02/06, PD-LB-01 |
| `docs/decisions/badges.md` | ACTIVE / AUTHORITATIVE (rule Badge) | Badges: catalog, metric, vòng đời, DB, API, mobile. PD-BDG-01…12 duyệt 2026-10-07; CLEAN-BDG-01 hoàn tất và Admin mock đã gỡ khỏi release; DEP-BDG-01…03 chỉ chặn badge Deferred |
| `docs/decisions/shop.md` | ACTIVE / AUTHORITATIVE | Shop, Coin ledger, XP Booster, reward grant |
| `docs/decisions/quiz.md` | ACTIVE / AUTHORITATIVE | Quiz, XP quiz |
| `docs/decisions/custom_card.md` | ACTIVE / AUTHORITATIVE | Template/flashcard |
| `docs/spec/specs.md` | ACTIVE / AUTHORITATIVE (FR, scope) | Chi tiết nghiệp vụ theo decisions |
| `docs/db/database.md` | ACTIVE / AUTHORITATIVE (schema) | Phần mission khớp `daily_mission.md` §7; phần badge khớp `badges.md` §7 |
| `docs/spec/buss_mainflow.md`, `phan_ra_tinh_nang.md`, `phan_ra_phan_he_he_thong.md`, `phan_ra_man_hinh.md` | ACTIVE / DERIVED | |
| `docs/sa/sa.md`, `server.md`, `techstack.md` | ACTIVE / DERIVED | |
| `docs/ke_hoach_phan_chia_cong_viec.xlsx` | ACTIVE / DERIVED | Kế hoạch phân công; endpoint/bảng theo decisions |
| `docs/ui/collection_creation_flow_ui.md` | ACTIVE / DERIVED | Ảnh minh họa gốc chưa có trong repo |
| `docs/ui/achievements_ui.md` | ACTIVE / AUTHORITATIVE (visual + interaction Thành tựu) | Freeze 2026-10-07; rule nghiệp vụ thuộc `badges.md` |
| `docs/ui/admin_badges_ui.md` | ACTIVE / DERIVED (Admin Huy hiệu) | P1 dùng API thật; PD-BDG-13a–d duyệt 2026-10-08; phát hành cần staging và nghiệm thu |
| `snapVocab-frontend/docs/design.md` | ACTIVE / DERIVED | Design tokens duy nhất (shop.md tham chiếu); nằm trong thư mục prototype |
| `snapVocab-frontend/docs/mascot_snapy.md` | ACTIVE / DERIVED | Mascot |
| `SnapVocab/docs/*`, `SnapVocab/AGENTS.md`, `SnapVocab/claude.md` | ACTIVE / DERIVED | Quy ước code mobile |
| `snapAdmin/docs/design/design.md` | ACTIVE / DERIVED | Admin UI; còn nội dung ngoài scope (Gems, Streak Recovery) chưa audit nghiệp vụ |
| `snap-vocab-backend/README.md`, `snapAdmin/README.md`, `snapVocab-frontend/README.md`, `snap-vocab-backend/ai-service/README.md` | ACTIVE / DERIVED | README kỹ thuật |
| `plan/shop-item/*` | HISTORICAL | Kịch bản/sơ đồ Shop trước khi implement; `shop.md` thắng |
| `plan/engagement/implementation_plan.md` | SUPERSEDED | Daily Goal/Challenge/Weekly; không dùng làm nguồn |
| `.tmp/mission-review/REPORT.md` | HISTORICAL | Review mock admin 17/09; tham chiếu file không còn tồn tại |
| `docs/design.md` (đã xóa) | BROKEN / REMOVED | File rỗng; link đã trỏ sang `snapVocab-frontend/docs/design.md` |

## Quy ước khi cập nhật tài liệu

1. Nếu thay đổi phạm vi sản phẩm, cập nhật [spec/specs.md](./spec/specs.md) trước.
2. Nếu thêm/sửa luồng nghiệp vụ, cập nhật [spec/buss_mainflow.md](./spec/buss_mainflow.md) và trace về FR/BF/SS/MH liên quan.
3. Nếu thêm backlog implement, cập nhật [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md) với ID, priority và AC đo được.
4. Nếu thay đổi kiến trúc hoặc công nghệ, cập nhật [sa/sa.md](./sa/sa.md), [sa/techstack.md](./sa/techstack.md) hoặc [sa/server.md](./sa/server.md) tương ứng.
5. Nếu thay đổi entity/schema/ràng buộc, cập nhật [db/database.md](./db/database.md).
6. Nếu một quyết định có nhiều trade-off hoặc ảnh hưởng dài hạn, ghi thành tài liệu trong [decisions/](./decisions/).

## Traceability

Tài liệu dùng các nhóm ID sau để truy vết yêu cầu:

| Prefix | Ý nghĩa                | Ví dụ                                   |
| ------ | ---------------------- | --------------------------------------- |
| FR     | Functional Requirement | FR-02 Image Recognition Vocabulary Flow |
| BF     | Business Flow          | BF-06 Scan-to-Vocabulary                |
| SS     | Subsystem              | SS-07 AI Service                        |
| MH     | Màn hình               | Camera Scan, Detection Result           |
| F      | Feature backlog item   | F-RECOG-05 AI Florence-2 pipeline       |

Khi implement một chức năng, nên bắt đầu từ FR trong [spec/specs.md](./spec/specs.md), đi qua BF trong [spec/buss_mainflow.md](./spec/buss_mainflow.md), kiểm tra AC trong [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md), rồi mới đối chiếu kiến trúc và database.
