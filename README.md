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

| Chủ đề | Quyết định |
| --- | --- |
| Source of truth | [spec/specs.md](./spec/specs.md) |
| AI pipeline | Florence-2 + SAM + CLIP, cấu hình F2-v13 zero-shot |
| Actor | Guest, Learner, Admin |
| Learning domain | Deck → Note → Card + ReviewLog |
| Saved vocabulary | Note trong Deck của Learner; không dùng SavedWord/UserWord song song |
| SRS | FSRS trên Card |
| Milestone | M1 Auth+Dict → M2 Scan → M3 Learning → M4 Game+Prod |
| Community scope | Không có community/study group; chỉ giữ leaderboard cá nhân |

## Cấu trúc tài liệu

```text
docs/
├── README.md
├── design.md
├── db/
│   └── database.md
├── decisions/
│   ├── custom_card.md
│   └── daily_mission.md
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

- [db/database.md](./db/database.md): schema dữ liệu cho Identity, Dictionary, Topic, Flashcard/SRS, Daily Mission, Gamification, Storage và các ràng buộc/index quan trọng.

### 4. Decision records

- [decisions/custom_card.md](./decisions/custom_card.md): thiết kế Card Template System, custom template, system template, 1 Deck dùng 1 template và 1 Note sinh 1 Card.
- [decisions/daily_mission.md](./decisions/daily_mission.md): thiết kế Daily Mission, Daily Chest, Weekly Milestone, mission pool, event tracking và reward idempotency.

### 5. Design

- [design.md](./design.md): tài liệu thiết kế UI/UX. File hiện là nơi dự kiến tổng hợp guideline giao diện, design system, màu sắc, typography, component và screen states.

## Milestone triển khai

| Milestone | Tên | Mục tiêu chính | Tài liệu liên quan |
| --- | --- | --- | --- |
| M1 | Core Auth & Vocabulary Lookup | Auth, profile, dictionary, topic, saved vocabulary, flashcard cơ bản | [spec/specs.md](./spec/specs.md), [spec/buss_mainflow.md](./spec/buss_mainflow.md) |
| M2 | Camera/Object Recognition MVP | Camera/gallery, upload, AI recognition, word mapping, save from scan | [sa/sa.md](./sa/sa.md), [sa/server.md](./sa/server.md) |
| M3 | Learning Engine | Custom card, quiz, SRS, progress, notification | [decisions/custom_card.md](./decisions/custom_card.md), [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md) |
| M4 | Gamification & Production Readiness | Mission, XP, coin, badge, shop, leaderboard, admin CMS, production hardening | [decisions/daily_mission.md](./decisions/daily_mission.md), [sa/server.md](./sa/server.md) |

## Quy ước khi cập nhật tài liệu

1. Nếu thay đổi phạm vi sản phẩm, cập nhật [spec/specs.md](./spec/specs.md) trước.
2. Nếu thêm/sửa luồng nghiệp vụ, cập nhật [spec/buss_mainflow.md](./spec/buss_mainflow.md) và trace về FR/BF/SS/MH liên quan.
3. Nếu thêm backlog implement, cập nhật [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md) với ID, priority và AC đo được.
4. Nếu thay đổi kiến trúc hoặc công nghệ, cập nhật [sa/sa.md](./sa/sa.md), [sa/techstack.md](./sa/techstack.md) hoặc [sa/server.md](./sa/server.md) tương ứng.
5. Nếu thay đổi entity/schema/ràng buộc, cập nhật [db/database.md](./db/database.md).
6. Nếu một quyết định có nhiều trade-off hoặc ảnh hưởng dài hạn, ghi thành tài liệu trong [decisions/](./decisions/).

## Traceability

Tài liệu dùng các nhóm ID sau để truy vết yêu cầu:

| Prefix | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| FR | Functional Requirement | FR-02 Image Recognition Vocabulary Flow |
| BF | Business Flow | BF-06 Scan-to-Vocabulary |
| SS | Subsystem | SS-07 AI Service |
| MH | Màn hình | Camera Scan, Detection Result |
| F | Feature backlog item | F-RECOG-05 AI Florence-2 pipeline |

Khi implement một chức năng, nên bắt đầu từ FR trong [spec/specs.md](./spec/specs.md), đi qua BF trong [spec/buss_mainflow.md](./spec/buss_mainflow.md), kiểm tra AC trong [spec/phan_ra_tinh_nang.md](./spec/phan_ra_tinh_nang.md), rồi mới đối chiếu kiến trúc và database.
