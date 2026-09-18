# Phân Rã Phân Hệ Hệ Thống — SnapVocab

> Hub: [specs.md](./specs.md) · Source of truth cho mọi FR/BF/SS/MH.  
> Tài liệu mô tả cách hệ thống SnapVocab được phân rã thành các phân hệ (module/subsystem), trách nhiệm, entities, API chính và quan hệ phụ thuộc giữa chúng.  
> Quyết định canonical: xem bảng §66–76 trong specs.md.

**Quy ước ID:** `SS-{nn}` — mỗi SS là một phân hệ/module trong kiến trúc backend hoặc hệ thống.
**Quy ước Endpoint (REST API Naming Convention):** Tất cả các public API đều ngầm định có **Base Path là `/api/v1`**. Luôn sử dụng danh từ số nhiều (plural nouns) và gom nhóm theo prefix (VD: `/api/v1/quizzes`, `/api/v1/reviews`). Tài liệu này là Single Source of Truth cho toàn bộ URL của hệ thống.

---

## Tổng quan phân hệ

Hệ thống SnapVocab được chia thành **18 phân hệ** thuộc 5 lớp chức năng:

```text
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│                                   HỆ THỐNG SNAPVOCAB                                     │
│                                                                                          │
│  ═══════════════════════ LỚP PRESENTATION (Client) ══════════════════════════            │
│  ┌────────────────────────┐  ┌────────────────────────┐                                  │
│  │  SS-01: MOBILE APP     │  │  SS-02: ADMIN CMS      │                                  │
│  │  (React Native/Expo)   │  │  (Web App Dashboard)   │                                  │
│  └────────────┬───────────┘  └────────────┬───────────┘                                  │
│               │                            │                                              │
│  ═══════════════════════ LỚP IDENTITY & SECURITY ════════════════════════════            │
│  ┌────────────────────────────────────────────────────────┐                               │
│  │  SS-03: IDENTITY (Auth, User, Profile, JWT, OTP)       │                               │
│  └────────────────────────┬───────────────────────────────┘                               │
│                            │                                                              │
│  ═══════════════════════ LỚP DOMAIN CHÍNH ═══════════════════════════════════            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐              │
│  │  SS-04:      │  │  SS-05:      │  │  SS-06:      │  │  SS-07:          │              │
│  │  DICTIONARY  │  │  TOPIC       │  │  RECOGNITION │  │  AI SERVICE      │              │
│  │  (Từ điển)   │  │  (Chủ đề)    │  │  (Orchestr.) │  │  (Florence-2)    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────────────────┘              │
│         │                 │                  │                                            │
│  ┌──────┴─────────────────┴──────────────────┴────────────────────────────┐               │
│  │  SS-08: VOCABULARY (Topic / TopicItem — Personal Vocabulary)           │               │
│  └───────────────────────────┬────────────────────────────────────────────┘               │
│                               │                                                          │
│  ═══════════════════════ LỚP LEARNING ENGINE ════════════════════════════════            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐              │
│  │  SS-09:      │  │  SS-10:      │  │  SS-11:      │  │  SS-12:          │              │
│  │  FLASHCARD   │  │  QUIZ        │  │  SRS         │  │  PROGRESS        │              │
│  │  & TEMPLATE  │  │              │  │  (FSRS)      │  │  TRACKING        │              │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘              │
│                                                                                          │
│  ═══════════════════════ LỚP ENGAGEMENT & INFRA ═════════════════════════════            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐              │
│  │  SS-13:      │  │  SS-14:      │  │  SS-15:      │  │  SS-16:          │              │
│  │  GAMIFICATION│  │  SHOP        │  │  NOTIFICATION│  │  STORAGE         │              │
│  │  (XP,Coin,   │  │  (Vật phẩm)  │  │  (Push/      │  │  (Object         │              │
│  │  Badge,      │  │              │  │   In-app)    │  │   Storage)       │              │
│  │  Mission)    │  │              │  │              │  │                  │              │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘              │
│                                                                                          │
│  ┌──────────────┐  ┌──────────────┐                                                      │
│  │  SS-17:      │  │  SS-18:      │                                                      │
│  │  ADMIN       │  │  API DOCS    │                                                      │
│  │  (Dashboard) │  │  (OpenAPI)   │                                                      │
│  │  └──────────────┘  └──────────────┘                                                      │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## SS-01: Mobile App — Ứng dụng di động

### Mô tả

Giao diện người dùng chính của hệ thống. Cung cấp trải nghiệm học từ vựng trên thiết bị di động: camera/gallery, tra cứu từ điển, flashcard, quiz, SRS, gamification và quản lý hồ sơ cá nhân.

### Công nghệ

| Thành phần       | Công nghệ                          |
| ---------------- | ---------------------------------- |
| Framework        | React Native / Expo                |
| Ngôn ngữ         | TypeScript                         |
| Navigation       | Expo Router                        |
| State management | Tùy chọn (Context / Zustand / TQ)  |
| API Client       | Axios / Fetch + interceptor JWT    |

### Chức năng chính

- Màn hình Welcome/Onboarding, đăng ký, đăng nhập, OTP verify
- Hồ sơ cá nhân, avatar upload
- Camera capture, gallery pick, gửi ảnh nhận diện
- Nút nổi quét màn hình (Android overlay bubble — Could)
- Tra cứu từ điển (text + voice), duyệt Collection/Topic
- My Vocabulary (danh sách Topic/TopicItem cá nhân), word detail
- Flashcard study session, render theo cấu hình Topic Template
- Quiz (MCQ, matching, fill blank)
- SRS review session, FSRS rating
- Progress dashboard, streak, accuracy
- Leaderboard, missions, badges, shop
- Thông báo in-app, cấu hình push notification
- Settings

### Trace

- FR: FR-01 → FR-12 (consumer)
- BF: BF-01 → BF-13

### Milestone

- M1: Auth, Profile, Dictionary, Topic, Vocab, Flashcard cơ bản
- M2: Camera/Scan, Scan result
- M3: Quiz, SRS, Progress, Notification
- M4: Gamification, Leaderboard, Shop

---

## SS-02: Admin CMS — Dashboard quản trị

### Mô tả

Web application nội bộ (tách biệt với Mobile App) dành cho Admin quản lý người dùng, từ vựng, chủ đề, gamification, card template và xem thống kê hệ thống.

### Công nghệ

| Thành phần | Công nghệ          |
| ---------- | ------------------- |
| Platform   | Web App nội bộ      |
| Auth       | JWT (role ROLE_ADMIN) |

### Chức năng chính

- Quản lý người dùng: list, search, ban/unban, reset password
- Quản lý từ điển: CRUD Word/Definition/Translation/Pronunciation, soft-delete
- Quản lý Collection/Topic: cấu trúc chủ đề, TopicItem, thuộc tính EAV
- Import dữ liệu từ vựng hàng loạt (CSV/Excel)
- Quản lý Topic Template hệ thống
- Quản lý gamification: Missions, Badges, Shop Items, XP config
- Xem Feedback/báo lỗi từ người dùng
- Dashboard thống kê: users active, lượt dùng AI, dung lượng R2/S3

### Trace

- FR: FR-13
- BF: BF-14

### Milestone

- M4 (có thể parallel, không block M1–M3)

---

## SS-03: Identity — Xác thực & Quản lý người dùng

### Mô tả

Quản lý toàn bộ vòng đời tài khoản: đăng ký, xác thực email/OTP, đăng nhập (mật khẩu + sinh trắc học), quản lý phiên JWT, khôi phục mật khẩu và hồ sơ cá nhân. Cung cấp identity context cho tất cả phân hệ khác.

### Entities

| Entity         | Mô tả                                                                       |
| -------------- | ---------------------------------------------------------------------------- |
| `User`         | Tài khoản người dùng (name, email, hashedPassword, avatarUrl, status, role)  |
| `Authority`    | Quyền/vai trò: ROLE_LEARNER, ROLE_ADMIN                                     |
| `RefreshToken` | Token làm mới phiên đăng nhập (token, userId, expiresAt, revoked)            |
| `OtpToken`     | Mã OTP xác thực (code, userId, type, expiresAt, used, attemptCount)          |

### Chức năng chính

- Đăng ký tài khoản (email, password, tên hiển thị)
- Xác thực email/OTP (TTL ≤ 10 phút, max 5 lần, resend ≥ 60s, one-time)
- Đăng nhập (email/password → Access Token + Refresh Token)
- Đăng nhập vân tay/sinh trắc học (Should)
- Refresh token (rotate, revoke on logout)
- Đăng xuất (revoke refresh token)
- Khôi phục mật khẩu (OTP/link reset)
- Xem/cập nhật hồ sơ cá nhân (tên, avatar)
- Cung cấp user identity + role cho Spring Security filter chain

### API Endpoints

| Method | Endpoint                    | Mô tả                                    | Auth     |
| ------ | --------------------------- | ----------------------------------------- | -------- |
| GET    | `/app-config`               | Cấu hình app (minSupportedAppVersion)    | Public   |
| POST   | `/auth/register`            | Đăng ký tài khoản mới                    | Public   |
| POST   | `/auth/verify-otp`          | Xác thực OTP                              | Public   |
| POST   | `/auth/resend-otp`          | Gửi lại OTP                               | Public   |
| POST   | `/auth/login`               | Đăng nhập                                 | Public   |
| POST   | `/auth/refresh`             | Làm mới access token                      | Public   |
| POST   | `/auth/logout`              | Đăng xuất (revoke refresh token)          | Learner  |
| POST   | `/auth/forgot-password`     | Yêu cầu reset mật khẩu                   | Public   |
| POST   | `/auth/reset-password`      | Đặt mật khẩu mới                          | Public   |
| GET    | `/users/me`                 | Xem hồ sơ cá nhân                         | Learner  |
| PUT    | `/users/me`                 | Cập nhật hồ sơ                             | Learner  |

### Business Rules

1. Mật khẩu hash ở backend, không lưu plaintext.
2. OTP TTL ≤ 10 phút, max 5 lần thử, resend cooldown ≥ 60s, không tái sử dụng.
3. JWT + role-based access. API cá nhân yêu cầu JWT hợp lệ.
4. Refresh token revoke khi logout hoặc rủi ro bảo mật.
5. Sai credential → generic message, không tiết lộ email tồn tại.

### Trace

- FR: FR-01
- BF: BF-01, BF-02, BF-03, BF-04

### Milestone: M1

---

## SS-04: Dictionary — Từ điển & Tra cứu từ vựng

### Mô tả

Quản lý cơ sở dữ liệu từ vựng Anh-Việt (357,729+ từ). Cung cấp API tra cứu, word detail và ánh xạ nhãn AI sang từ vựng. Là nguồn dữ liệu chính cho toàn bộ luồng học tập.

### Entities

| Entity           | Mô tả                                                                |
| ---------------- | --------------------------------------------------------------------- |
| `Word`           | Từ vựng gốc trong dictionary (word, pos)                              |
| `Definition`     | Định nghĩa/giải thích của từ                                          |
| `Translation`    | Bản dịch/nghĩa tiếng Việt                                            |
| `Pronunciation`  | Phiên âm IPA, audio URL                                              |
| `WordDefinition` | Liên kết word ↔ definition (hỗ trợ nhiều nghĩa/loại từ)              |
| `WordRelation`   | Quan hệ synonym/antonym/related words                                 |
| `ObjectWordMapping` | Ánh xạ nhãn AI pipeline → Word (xử lý từ điển Anh-Việt thiếu mục) |

### Chức năng chính

- Tra cứu từ vựng bằng văn bản (text search, p95 < 500ms cho từ phổ biến)
- Tra cứu giọng nói — Should: mobile dùng **STT on-device** (`expo-speech-recognition`) capture tiếng Việt → text → gửi lên `/words/search`; backend chỉ cần reverse-lookup bảng Translation (ý Việt → Word). **Không cần STT engine, translate API phía backend.** (ARC-12)
- Xem word detail: nghĩa tiếng Việt, phiên âm IPA, phát âm, loại từ
  - **Phát âm:** API trả `audioUrl` (nullable, từ DB minhqnd); nếu null → mobile fallback TTS on-device (`expo-speech`). Không cần TTS server, không thay đổi DB schema. (ARC-12)
- Hiển thị nhiều nghĩa/POS theo nhóm
- Hiển thị synonym/antonym/ví dụ (nếu dữ liệu hỗ trợ) — Could
- Ánh xạ label từ AI service sang Word (tra cứu trực tiếp + mapping/synonym)
- Admin CRUD từ vựng, definition, translation, pronunciation (soft-delete)
- Import dữ liệu từ vựng hàng loạt (CSV/Excel)

### API Endpoints

| Method    | Endpoint                        | Mô tả                                      | Auth    |
| --------- | ------------------------------- | ------------------------------------------- | ------- |
| GET       | `/words/search`                 | Tra cứu từ vựng (text, voice result)        | Learner |
| GET       | `/words/{id}`                   | Chi tiết từ vựng                             | Learner |
| GET       | `/words/by-label/{label}`       | Ánh xạ nhãn AI → word detail                | System  |
| GET/POST/PUT/DELETE | `/admin/words`        | Admin CRUD từ vựng                           | Admin   |
| POST      | `/admin/words/import`           | Import hàng loạt                             | Admin   |

### Sub-components

```text
Dictionary
  ├── WordSearchService         — Text search, ranking, cache Redis top words
  ├── VoiceLookupService        — Reverse-lookup bảng Translation (tiếng Việt → Word); **không** gọi STT hay translate API; STT được thực hiện on-device bằng expo-speech-recognition (ARC-12)
  ├── WordDetailService         — Aggregate word + definitions + translations + pronunciations; `audioUrl` nullable
  ├── ObjectWordMappingService  — Ánh xạ AI label → Word (synonym/mapping table)
  ├── DictionaryImportService   — Batch import CSV/Excel
  └── DictionaryAdminService    — CRUD + soft-delete
```

### Trace

- FR: FR-03, FR-13.02, FR-13.03
- BF: BF-05, BF-06 (ánh xạ), BF-14

### Milestone: M1

---

## SS-05: Topic — Chủ đề & Bộ sưu tập học tập

### Mô tả

Quản lý bộ sưu tập (Collections) và chủ đề (Topics) từ vựng theo mô hình EAV linh hoạt. Cho phép Learner duyệt và học từ theo chủ đề có sẵn.

### Entities

| Entity                    | Mô tả                                                                         |
| ------------------------- | ------------------------------------------------------------------------------ |
| `Collection`              | Tập hợp chủ đề lớn (VD: TOEIC Words, Animals)                                 |
| `Topic`                   | Chủ đề cụ thể, hỗ trợ phân cấp (parent_id), thuộc Collection                  |
| `TopicItem`               | Phần tử nội dung (từ vựng/cụm từ) thuộc Topic                                 |
| `TopicAttributeGroup`     | Nhóm thuộc tính cho một Topic                                                  |
| `TopicAttribute`          | Định nghĩa thuộc tính (VD: Nghĩa tiếng Việt, Phiên âm, Ví dụ, Audio)         |
| `TopicItemAttributeValue` | Giá trị thực tế của thuộc tính cho từng TopicItem                              |

### Chức năng chính

- Duyệt danh sách Collections
- Xem Topics theo Collection (hỗ trợ phân cấp parent/child)
- Xem TopicItems kèm thuộc tính EAV (nghĩa, phiên âm, ví dụ, audio)
- Sao chép/lưu TopicItem vào Topic cá nhân (source = TOPIC)
- Admin CRUD Collection/Topic/TopicItem và thuộc tính

### API Endpoints

| Method    | Endpoint                                | Mô tả                                    | Auth    |
| --------- | --------------------------------------- | ----------------------------------------- | ------- |
| GET       | `/collections`                          | Danh sách Collections                     | Learner |
| GET       | `/collections/{id}/topics`              | Topics trong Collection                   | Learner |
| GET       | `/topics/{id}`                          | Chi tiết Topic + children                 | Learner |
| GET       | `/topics/{id}/items`                    | TopicItems + attributes                   | Learner |
| GET/POST/PUT/DELETE | `/admin/collections`          | Admin CRUD Collection                     | Admin   |
| GET/POST/PUT/DELETE | `/admin/topics`               | Admin CRUD Topic                          | Admin   |
| GET/POST/PUT/DELETE | `/admin/topic-items`          | Admin CRUD TopicItem + attributes         | Admin   |

### Trace

- FR: FR-03 (topic browse), FR-04.01 (save from topic), FR-13.02
- BF: BF-05 (luồng chủ đề)

### Milestone: M1

---

## SS-06: Recognition — Orchestration nhận diện ảnh

### Mô tả

Phân hệ backend phía Spring Boot chịu trách nhiệm orchestrate luồng nhận diện ảnh: cấp URL upload, kiểm tra quota, xếp hàng job, gọi AI service, lọc kết quả, ánh xạ sang từ vựng và trả kết quả khi mobile poll. **Không** chạy model AI trực tiếp.

### Entities

| Entity | Mô tả |
| ------ | ----- |
| `ScanRequest` (`scan_requests`) | Một lượt scan: user, objectKey, status, errorCode, result_json (box từ AI), modelVersion, detectionCount, processingTimeMs, startedAt/finishedAt, version. Cũng là nguồn đếm quota. Xem [database.md §5](../db/database.md). |
| `ScanDetectionsDTO` | Nội dung `result_json`: `imageWidth`, `imageHeight`, `detections[]` (label, headword, source, reliability, box). Không phải bảng riêng. |

### Chức năng chính

- Cấp presigned URL upload dưới `scans/{userId}/` (chỉ jpeg/png/webp)
- Kiểm tra `objectKey` thuộc Learner, ảnh đã upload, ≤ 10MB — **trước** khi tính quota
- Kiểm tra quota/ngày bằng cách đếm `scan_requests` (status ≠ FAILED) dưới khóa Redisson theo user
- Xếp hàng job (1 worker, tối đa 3 job chờ); đầy → `AI_QUEUE_FULL`
- Gọi FastAPI AI service qua HTTP nội bộ (`X-Request-Id`, `X-Service-Token`, timeout 60s)
- Ánh xạ lỗi AI thành mã lỗi scan (`INVALID_IMAGE`, `AI_UNAVAILABLE`, `AI_TIMEOUT`, `AI_ERROR`)
- Lọc lần cuối theo `scan.min-reliability`; lưu box vào `result_json`
- Khi poll: ánh xạ label → Word (tra nhãn, rồi `headword`) và trả kết quả
- Bộ quét: đánh dấu `INTERRUPTED` job mất khi restart hoặc quá thời gian tối đa
- Log `requestId` (MDC), `modelVersion`, số object, thời gian xử lý

### API Endpoints

| Method | Endpoint | Mô tả | Auth |
| ------ | -------- | ----- | ---- |
| POST | `/api/scan/upload-url` | Body `{contentType}` → `{uploadUrl, objectName, contentType, expiredAt}` | Learner |
| POST | `/api/scan` (`application/json`) | Body `{objectKey}` → `202 {requestId, status: PENDING}` | Learner |
| GET | `/api/scan/{requestId}` | `{requestId, status, errorCode, createdAt, finishedAt, result}`; `result = {imageWidth, imageHeight, items[]}` khi `DONE`. Scan của người khác trả 404 | Learner |
| GET | `/api/scan/quota` | `{limit, used, remaining, resetAt}` | Learner |
| POST | `/api/scan` (`multipart/form-data`) | **Deprecated** — endpoint đồng bộ cũ, field `file`, trả kết quả ngay (kèm ảnh vẽ sẵn box nếu AI bật). Vẫn tính quota | Learner |
| GET | `/api/scan/history` | Lịch sử scan của Learner (Should — chưa triển khai) | Learner |

Mỗi phần tử `items[]`: `label`, `score`, `source`, `reliability`, `box {x1, y1, x2, y2}`, `word` (LookupResult, `null` nếu không có trong từ điển). Mã lỗi trả trong `RestResponse.error`; riêng `QUOTA_EXCEEDED` kèm `data = {limit, used, remaining, resetAt}`.

### Sub-components

```text
Recognition (Backend)
  ├── ScanController        — /api/scan: upload-url, submit, poll, quota, endpoint đồng bộ cũ
  ├── ScanServiceImpl       — Điều phối: kiểm tra key/size → quota → enqueue → 202 + requestId
  ├── ScanQuotaService      — Đếm scan_requests trong ngày dưới khóa Redisson; tạo ScanRequest
  ├── ScanJobQueue          — ThreadPoolTaskExecutor riêng (không phải Spring bean, để @Async mail không bị ảnh hưởng)
  ├── ScanWorker            — PENDING → PROCESSING → DONE/FAILED, mỗi bước một transaction ngắn
  ├── AiDetectionService    — HTTP client gọi AI, ánh xạ lỗi AI → ScanErrorCode
  ├── ScanResultAssembler   — Lọc theo reliability; ánh xạ label/headword → Word (delegate SS-04)
  ├── ScanRequestSweeper    — @Scheduled + lúc khởi động: đánh dấu INTERRUPTED job kẹt
  └── ScanObjectKeys        — Quy ước key scans/{userId}/{uuid}.{ext}, kiểm tra quyền sở hữu
```

### Trace

- FR: FR-02
- BF: BF-06

### Milestone: M2

---

## SS-07: AI Service — Pipeline nhận diện từ vựng mở

### Mô tả

Service **độc lập** (Python FastAPI) chạy pipeline nhận diện từ vựng mở (open-vocabulary) dựa trên Florence-2 ở chế độ zero-shot. Nhận ảnh từ backend, trả danh sách đối tượng đã lọc theo từ điển. CLIP chỉ dùng khi bật bước từ vựng nền; SAM đã gỡ bỏ.

### Công nghệ

| Thành phần | Công nghệ |
| ---------- | --------- |
| Framework | Python FastAPI, Uvicorn 1 process |
| Models | Florence-2-base (CPU/dev) hoặc Florence-2-large (GPU); CLIP ViT-B/32 tùy chọn |
| Pipeline mặc định | `<OD>` + self-grounding; tiled OD / dense caption / từ vựng nền bật qua env |
| Phần cứng | CPU chạy được (~35–45s/ảnh với base); GPU ≥ 4GB VRAM cho large |
| Đồng thời | 1 ảnh tại một thời điểm (semaphore) |

### Chức năng chính

- Nhận ảnh multipart từ backend (không nhận trực tiếp từ mobile); kiểm tra `X-Service-Token` nếu đặt `SERVICE_TOKEN`
- Xoay ảnh theo EXIF, resize
- Florence-2: `<OD>` + self-grounding
- Loại box quá nhỏ/quá lớn; lọc nhãn theo từ điển (WordNet); NMS 2 tầng; 1 box / nhãn
- Gắn `headword` (từ cuối, dạng số ít) và `reliability` (theo `source`)
- Không có vật thể → danh sách rỗng, **không** phải lỗi
- Lỗi có cấu trúc: `INVALID_REQUEST`, `INVALID_IMAGE`, `UNAUTHORIZED`, `MODEL_NOT_READY`, `MODEL_ERROR`
- Logging: `request_id`, `model_version`, số object, thời gian xử lý

### API Endpoints (Internal)

| Method | Endpoint | Mô tả |
| ------ | -------- | ----- |
| POST | `/api/v1/detect` | Nhận ảnh (field `file`), trả danh sách detected objects |
| GET | `/health` | Trạng thái nạp model, `model_version`, `gpu_available` |

### Output Schema

```json
{
  "request_id": "3f2a9c1e-...",
  "model_version": "Florence-2-base/od+self",
  "processing_time_ms": 21450,
  "detections": [
    {
      "label": "coffee mug",
      "headword": "mug",
      "score": 0.9,
      "source": "od",
      "reliability": "HIGH",
      "box": { "x1": 120, "y1": 80, "x2": 350, "y2": 420 }
    }
  ],
  "labels": ["coffee mug"],
  "annotated_image_base64": null,
  "annotated_image_mime": null,
  "image_width": 900,
  "image_height": 1200
}
```

Lỗi: `{"error": {"code": "INVALID_IMAGE", "message": "..."}}`. Chi tiết contract ở [server.md §5.3](../sa/server.md).

### Trace

- FR: FR-02.05, FR-02.06
- BF: BF-06 (bước 8–10)
- Phụ lục A (specs.md §12)

### Milestone: M2

### Ghi chú

- AI service nằm trong mạng nội bộ; dùng `SERVICE_TOKEN` nếu cổng ra được mạng ngoài.
- Endpoint nội bộ tách rõ với endpoint public/mobile.
- MVP dùng zero-shot; fine-tune LoRA là hướng mở rộng (ngoài phạm vi MVP).

---

## SS-08: Vocabulary — Từ vựng cá nhân (Topic / TopicItem)

### Mô tả

Phân hệ quản lý từ vựng cá nhân của Learner theo mô hình `Collection (loại USER) → Topic → TopicItem`. UI "My Vocabulary / từ đã lưu" = danh sách TopicItem trong Topic của Learner. Không duy trì entity song song `Deck`/`Note`/`Card`/`SavedWord`/`UserWord`.

### Entities

| Entity                         | Mô tả                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------- |
| `Collection`                   | Bộ sưu tập chủ đề (type USER hoặc SYSTEM)                                    |
| `Topic`                        | Chủ đề học tập cá nhân của Learner, có thể phân cấp cha-con                  |
| `TopicItem`                    | Đơn vị từ vựng lưu vào Topic (wordId, text, phonetic, audio, meaning...)     |
| `TopicItemAttributeValue`      | Giá trị thuộc tính động EAV của TopicItem                                     |

### Chức năng chính

- Tạo/xem/xóa Topic trong Collection cá nhân (owner-only)
- Lưu từ → tạo TopicItem (từ scan, dictionary, hoặc topic hệ thống) + tự động khởi tạo FsrsRecord (SS-09/SS-11)
- Unique per Topic: không lưu từ trùng trong cùng Topic
- Xem danh sách từ (My Vocabulary)
- Lọc/sắp xếp theo UI state (new/learning/reviewing/mastered) suy từ FSRS, ngày lưu, độ khó, ngày ôn
- Xóa/archive TopicItem (FsrsRecord tương ứng bị xóa/ẩn, Word gốc trong từ điển không bị xóa)
- Gắn nguồn (source): SCAN, DICT, TOPIC

### API Endpoints

| Method | Endpoint                    | Mô tả                                      | Auth    |
| ------ | --------------------------- | ------------------------------------------ | ------- |
| GET    | `/collections`              | Danh sách Collection của Learner           | Learner |
| POST   | `/collections`              | Tạo Collection mới                         | Learner |
| GET    | `/topics`                   | Danh sách Topic của Learner                | Learner |
| POST   | `/topics`                   | Tạo Topic mới                              | Learner |
| PUT    | `/topics/{id}`              | Cập nhật Topic                             | Learner |
| DELETE | `/topics/{id}`              | Xóa Topic                                  | Learner |
| GET    | `/topics/{id}/items`        | Danh sách TopicItem trong Topic (lọc/sắp xếp) | Learner |
| POST   | `/topics/{id}/items`        | Lưu từ mới (tạo TopicItem + FsrsRecord)    | Learner |
| GET    | `/topic-items/{id}`         | Chi tiết TopicItem                         | Learner |
| DELETE | `/topic-items/{id}`         | Xóa TopicItem                              | Learner |

### Trace

- FR: FR-04, FR-05.01
- BF: BF-07

### Milestone: M1 (cơ bản), M2 (từ scan)

---

## SS-09: Flashcard & Template — Thẻ học & Topic Template

### Mô tả

Quản lý cấu hình hiển thị thẻ học thông qua `Template` gắn với `Topic` (loại `TOPIC_CUSTOM` hoặc kế thừa từ hệ thống `SYSTEM`), bao gồm các phần tử giao diện `TemplateElement` và trường dữ liệu `TemplateField` với vai trò ngữ nghĩa `SemanticRole` (FRONT, BACK, EXAMPLE, AUDIO, IMAGE, PHONETIC, TRANSLATION, HINT, TAG, EXTRA). Phiên học flashcard nạp các TopicItem và cập nhật FsrsRecord theo đánh giá của Learner.

### Entities

| Entity            | Mô tả                                                                             |
| ----------------- | ---------------------------------------------------------------------------------- |
| `Template`        | Mẫu thẻ học gắn theo Topic (`topic_id`), loại SYSTEM hoặc TOPIC_CUSTOM             |
| `TemplateElement` | Phần tử giao diện (type: FIELD, DIVIDER, BUTTON; order_index, flex, alignment)    |
| `TemplateField`   | Trường dữ liệu ánh xạ TopicAttribute với SemanticRole và cấu hình hiển thị         |
| `FsrsRecord`      | Trạng thái và tham số SRS của từ học (card_state, due, stability, difficulty...)  |

### Chức năng chính

- Khởi tạo FsrsRecord (card_state=NEW) khi TopicItem được tạo trong Topic
- System Templates: Classic, Listening, Spelling, Image Vocab... (seeded, không sửa/xóa)
- Custom Templates: Cấu hình phần tử giao diện (TemplateElement) và trường dữ liệu (TemplateField) với SemanticRole gắn cho Topic
- Render thẻ học linh hoạt theo cấu hình template của Topic (ẩn field thiếu dữ liệu, không vỡ layout)
- Study session: hiển thị TopicItem theo template → Learner tương tác → submit FSRS rating
- Cập nhật trực tiếp thông số FSRS trên FsrsRecord (card_state, due, stability, difficulty, reps, lapses)
- Hỗ trợ học hàng loạt và sync queue cục bộ (batch rating)

### API Endpoints

| Method | Endpoint                                  | Mô tả                                        | Auth    |
| ------ | ----------------------------------------- | -------------------------------------------- | ------- |
| GET    | `/topics/{id}/templates`                  | Lấy cấu hình template gắn với Topic          | Learner |
| PUT    | `/topics/{id}/templates`                  | Cập nhật cấu hình template cho Topic         | Learner |
| GET    | `/topics/{id}/items/study-session`        | Lấy danh sách TopicItem cần học (new + due)  | Learner |
| POST   | `/topic-items/{id}/review`                | Submit FSRS rating cho TopicItem             | Learner |
| POST   | `/reviews/batch`                          | Sync batch rating từ local queue             | Learner |
| GET    | `/admin/templates`                        | Admin quản lý System Templates               | Admin   |

### Sub-components

```text
Flashcard & Template
  ├── TemplateService         — Quản lý Template, TemplateElement, TemplateField (SemanticRole)
  ├── StudySessionService     — Build study queue (new + due), quản lý phiên học
  ├── FsrsService             — Thuật toán FSRS: card_state, due, stability, difficulty
  └── CardRendererConfig      — Ánh xạ SemanticRole sang UI elements cho mobile
```

### Trace

- FR: FR-05, FR-13.07
- BF: BF-08

### Milestone: M1 (flashcard cơ bản), M3 (template system)

---

## SS-10: Quiz — Kiểm tra từ vựng

### Mô tả

Sinh bài kiểm tra từ vựng từ TopicItem trong Topic của Learner hoặc Topic hệ thống. Hỗ trợ nhiều dạng câu hỏi, chấm điểm và lưu lịch sử attempt.

### Entities

| Entity         | Mô tả                                                                                   |
| -------------- | ---------------------------------------------------------------------------------------- |
| `Quiz`         | Bài kiểm tra (topicId, type, questionCount, createdAt)                                  |
| `Question`     | Câu hỏi trong quiz (type, topicItemId, correctAnswer, distractors)                      |
| `QuizAttempt`  | Lượt làm quiz (quizId, userId, score, correctCount, wrongCount, duration, completedAt) |
| `QuizAnswer`   | Chi tiết câu trả lời của người dùng trong lượt làm                                      |

### Chức năng chính

- Sinh quiz từ TopicItem trong Topic
- Dạng câu hỏi: Multiple choice (Must), Matching (Should), Fill blank (Could)
- Sinh đáp án nhiễu (lấy từ TopicItem cùng Topic/POS, không trùng nghĩa)
- Yêu cầu số từ tối thiểu để sinh quiz
- Chấm điểm: score, correctCount, wrongCount, accuracy
- Lưu QuizAttempt (idempotent — event key, retry không cộng trùng)
- Cập nhật Progress, XP nếu gamification bật (qua event)

### API Endpoints

| Method | Endpoint                             | Mô tả                                    | Auth    |
| ------ | ------------------------------------ | ----------------------------------------- | ------- |
| POST   | `/topics/{id}/quizzes/generate`      | Sinh quiz mới từ Topic                   | Learner |
| GET    | `/quizzes/{id}`                      | Lấy quiz + câu hỏi                        | Learner |
| POST   | `/quizzes/{id}/submit`               | Nộp bài, chấm điểm (idempotent)          | Learner |
| GET    | `/quizzes/history`                   | Lịch sử quiz attempts                     | Learner |

### Trace

- FR: FR-06
- BF: BF-09

### Milestone: M3

---

## SS-11: SRS — Spaced Repetition System (FSRS)

### Mô tả

Quản lý hàng đợi ôn tập theo thuật toán FSRS (Free Spaced Repetition Scheduler). Tính toán lịch ôn dựa trên kết quả recall của FsrsRecord, ưu tiên từ quá hạn.

### Chức năng chính

- Tính Daily Review Queue: FsrsRecord có `due <= now`, ưu tiên overdue
- Learner đánh giá recall (Again, Hard, Good, Easy)
- Cập nhật FSRS trên FsrsRecord: card_state, due, stability, difficulty, reps, lapses
- Recall tốt → interval tăng; recall kém → interval giảm hoặc đưa về LEARNING/RELEARNING theo FSRS
- Hiển thị số từ cần ôn trên Home (daily due count)
- Reset/archive từ vựng (Could)

### API Endpoints

| Method | Endpoint                             | Mô tả                                       | Auth    |
| ------ | ------------------------------------ | -------------------------------------------- | ------- |
| GET    | `/reviews/queue`                     | Daily review queue (due FsrsRecords)         | Learner |
| GET    | `/reviews/summary`                   | Tổng quan: due count, overdue count          | Learner |
| POST   | `/topic-items/{id}/review`           | Submit rating (shared with SS-09)            | Learner |
| POST   | `/reviews/batch`                     | Sync batch rating từ local queue             | Learner |
| POST   | `/topic-items/{id}/reset`            | Reset SRS về NEW (Could)                     | Learner |

### Ghi chú

- SRS logic tích hợp chặt với SS-09 (FsrsRecord entity + FsrsService). Tách SS vì trách nhiệm nghiệp vụ khác nhau: SS-09 quản lý study session / template, SS-11 quản lý review scheduling.
- FSRS parameters: card_state (NEW/LEARNING/REVIEW/RELEARNING/SUSPENDED), due, stability, difficulty, reps, lapses.
- UI/progress state dùng map chuẩn FR-04: NEW→new; LEARNING/RELEARNING→learning; REVIEW interval <21 ngày→reviewing; REVIEW interval ≥21 ngày→mastered.

### Trace

- FR: FR-07
- BF: BF-10

### Milestone: M3

---

## SS-12: Progress — Tiến độ học tập

### Mô tả

Tổng hợp và hiển thị tiến độ học tập cá nhân: số từ, streak, accuracy, lịch sử hoạt động. Dữ liệu aggregate từ FsrsRecord, QuizAttempt, TopicItem count.

### Entities

| Entity             | Mô tả                                                                     |
| ------------------ | -------------------------------------------------------------------------- |
| `LearningProgress` | Aggregate: totalWords, learnedCount, dueCount, masteredCount, streak, accuracy theo learning-state map |
| `LearningEvent`    | Sự kiện học (type, timestamp, metadata) — rebuild từ review event / QuizAttempt |

### Chức năng chính

- Tổng quan: số từ đã lưu, đã học, đang ôn, mastered theo learning-state map
- Streak: chuỗi ngày học liên tiếp (tăng khi hoàn thành điều kiện tối thiểu/ngày)
- Accuracy: tỷ lệ chính xác quiz/review
- Lịch sử hoạt động: ngày/tuần/tháng
- Home summary widget (progress ngắn gọn)
- Goal tracking (Could)
- Cập nhật sau mỗi hoạt động: lưu từ, flashcard review, quiz submit

Quy tắc aggregate:

- `learnedCount` = số TopicItem có UI state khác `new` (`learning + reviewing + mastered`).
- `dueCount` / đang ôn = số FsrsRecord có `due <= now`.
- `masteredCount` = số FsrsRecord có `card_state = REVIEW` và interval ≥ 21 ngày.

### API Endpoints

| Method | Endpoint                      | Mô tả                                     | Auth    |
| ------ | ----------------------------- | ------------------------------------------ | ------- |
| GET    | `/progress/summary`           | Tổng quan tiến độ                          | Learner |
| GET    | `/progress/streak`            | Chi tiết streak                             | Learner |
| GET    | `/progress/history`           | Lịch sử hoạt động (daily/weekly/monthly)   | Learner |
| GET    | `/progress/home-widget`       | Summary ngắn gọn cho Home screen           | Learner |

### Trace

- FR: FR-08
- BF: BF-11

### Milestone: M3

---

## SS-13: Gamification — XP, Coin, Mission, Badge, Leaderboard

### Mô tả

Hệ thống tăng động lực học tập: điểm kinh nghiệm (XP), tiền ảo (Coin), nhiệm vụ (Mission), huy hiệu (Badge) và bảng xếp hạng (Leaderboard). Tất cả reward phải idempotent (event key).

### Entities

| Entity            | Mô tả                                                        |
| ----------------- | ------------------------------------------------------------- |
| `Mission`         | Nhiệm vụ ngày/tuần/thành tựu (type, target, reward, period)  |
| `MissionProgress` | Tiến độ nhiệm vụ của Learner (current, completed, claimedAt) |
| `Badge`           | Định nghĩa huy hiệu (name, condition, iconUrl)               |
| `UserBadge`       | Huy hiệu Learner đã đạt (earnedAt)                           |
| `ExperienceLog`   | Lịch sử cộng XP (amount, source, eventKey, timestamp)        |
| `CoinTransaction` | Lịch sử cộng/trừ coin (amount, type, eventKey, balance)      |
| `LeaderboardEntry`| Bản ghi xếp hạng (userId, score, period, rank)               |

### Chức năng chính

- **XP:** Cộng XP khi hoàn thành hoạt động học (idempotent — event key)
- **Coin:** Cộng coin theo mission/milestone; trừ coin khi mua item
- **Mission:** Nhiệm vụ ngày/tuần/thành tựu; tự động cập nhật progress; claim reward
- **Badge:** Trao huy hiệu khi đạt điều kiện cụ thể
- **Leaderboard:** Xếp hạng theo Weekly XP; Redis sorted set / snapshot cache
- Admin cấu hình missions, badges, XP rules

### API Endpoints

| Method | Endpoint                              | Mô tả                                       | Auth    |
| ------ | ------------------------------------- | -------------------------------------------- | ------- |
| GET    | `/gamification/xp`                    | Tổng XP và lịch sử                           | Learner |
| GET    | `/gamification/coins`                 | Balance coin và lịch sử giao dịch            | Learner |
| GET    | `/gamification/missions`              | Danh sách missions + progress                | Learner |
| POST   | `/gamification/missions/{id}/claim`   | Claim reward nhiệm vụ (idempotent)           | Learner |
| GET    | `/gamification/badges`                | Danh sách badges (earned + available)        | Learner |
| GET    | `/leaderboards`                       | Bảng xếp hạng (period, type)                | Learner |
| GET/POST/PUT/DELETE | `/admin/missions`       | Admin CRUD missions                          | Admin   |
| GET/POST/PUT/DELETE | `/admin/badges`         | Admin CRUD badges                            | Admin   |

### Sub-components

```text
Gamification
  ├── XpService             — Tính, cộng XP (idempotent event key)
  ├── CoinService           — Cộng/trừ coin, balance tracking (balance ≥ 0)
  ├── MissionService        — CRUD mission, auto-update progress, claim
  ├── BadgeService          — Evaluate conditions, grant badges
  ├── LeaderboardService    — Redis sorted set / snapshot cache, ranking
  └── RewardEventHandler    — Lắng nghe learning events → trigger reward logic
```

### Trace

- FR: FR-09
- BF: BF-11, BF-12
- Detail: [daily_mission.md](./daily_mission.md)

### Milestone: M4

---

## SS-14: Shop — Cửa hàng vật phẩm

### Mô tả

Cửa hàng vật phẩm ảo trong ứng dụng. Learner dùng Coin mua vật phẩm (theme, avatar frame, booster). Không xử lý thanh toán tiền thật.

### Entities

| Entity     | Mô tả                                                           |
| ---------- | ---------------------------------------------------------------- |
| `ShopItem` | Vật phẩm trong cửa hàng (name, price, type, iconUrl, status)    |
| `UserItem` | Vật phẩm Learner sở hữu/đang sử dụng (purchasedAt, equipped)   |

### Chức năng chính

- Duyệt danh sách vật phẩm
- Mua vật phẩm bằng Coin (balance ≥ price)
- Áp dụng vật phẩm (đổi theme, avatar frame, booster)
- Admin CRUD vật phẩm

### API Endpoints

| Method    | Endpoint                        | Mô tả                               | Auth    |
| --------- | ------------------------------- | ------------------------------------ | ------- |
| GET       | `/shop/items`                   | Danh sách vật phẩm                   | Learner |
| POST      | `/shop/items/{id}/buy`          | Mua vật phẩm                         | Learner |
| POST      | `/shop/inventory/{id}/equip`    | Áp dụng vật phẩm                     | Learner |
| GET       | `/shop/inventory`               | Danh sách vật phẩm sở hữu           | Learner |
| GET/POST/PUT/DELETE | `/admin/shop-items`   | Admin CRUD vật phẩm                  | Admin   |

### Trace

- FR: FR-09.06, FR-09.07
- BF: BF-12

### Milestone: M4 (Could)

---

## SS-15: Notification — Hệ thống thông báo

### Mô tả

Gửi thông báo đẩy (Push) và thông báo trong ứng dụng (In-app) cho Learner: nhắc nhở ôn SRS, thông báo badge/coin, tin hệ thống.

### Entities

| Entity         | Mô tả                                                                   |
| -------------- | ------------------------------------------------------------------------ |
| `Notification` | Thông báo in-app (userId, title, body, type, readAt, createdAt)          |
| `DeviceToken`  | Token thiết bị cho push notification (userId, token, platform, updatedAt) |

### Chức năng chính

- **Push Notification:** Gửi qua Expo Push hoặc Firebase FCM
- **In-app Notification:** Lưu database, Learner xem lại khi mở app
- Đánh dấu đã đọc
- Cấu hình thông báo: Learner bật/tắt push trong Settings
- Đăng ký/cập nhật device token
- Tối đa 1 push nhắc SRS/ngày khung 19–21h; tuân thủ giờ nhận (nếu cấu hình)

### API Endpoints

| Method | Endpoint                                | Mô tả                                    | Auth    |
| ------ | --------------------------------------- | ----------------------------------------- | ------- |
| GET    | `/notifications`                        | Danh sách in-app notifications            | Learner |
| PUT    | `/notifications/{id}/read`              | Đánh dấu đã đọc                          | Learner |
| POST   | `/notifications/device-token`           | Đăng ký device token                      | Learner |
| PUT    | `/notifications/settings`               | Cấu hình thông báo                        | Learner |

### Sub-components

```text
Notification
  ├── PushNotificationService   — Gửi push qua Expo Push / FCM
  ├── InAppNotificationService  — CRUD in-app notification
  ├── DeviceTokenService        — Quản lý device token
  ├── NotificationScheduler     — Job nhắc SRS daily review
  └── NotificationPreferences   — Cấu hình bật/tắt, giờ nhận
```

### Trace

- FR: FR-10
- BF: BF-13

### Milestone: M3

---

## SS-16: Storage — Object Storage & Media

### Mô tả

Quản lý lưu trữ media (ảnh scan, avatar, tài nguyên vật phẩm) qua S3-compatible Object Storage. Cung cấp presigned upload/download URL. Bucket private.

### Công nghệ

| Môi trường | Công nghệ                                 |
| ---------- | ------------------------------------------ |
| Dev/Local  | MinIO hoặc S3-compatible storage            |
| Production | Cloudflare R2 qua S3-compatible API        |

### Chức năng chính

- Presigned upload: backend cấp URL để mobile upload trực tiếp
- Upload complete: client báo → backend validate MIME/size + lưu metadata
- Presigned download/access: URL tạm thời (TTL ≤ 15 phút)
- Avatar upload flow (≤ 5MB)
- Scan image storage (≤ 10MB, optional — tuân thủ privacy)
- Tài nguyên vật phẩm gamification
- Object key do backend sinh (không dùng tên file user)
- Orphan cleanup job (chỉ xóa object type=CROP, state=TEMP đã tạo quá 24h)

### API Endpoints

| Method | Endpoint                          | Mô tả                                         | Auth    |
| ------ | --------------------------------- | ---------------------------------------------- | ------- |
| POST   | `/storage/upload-init`            | Khởi tạo upload, trả presigned PUT URL         | Learner |
| POST   | `/storage/upload-complete`        | Xác nhận upload xong, validate + lưu metadata  | Learner |
| GET    | `/storage/access-url/{objectKey}` | Lấy presigned GET URL (TTL ≤ 15m)              | Learner |

### Sub-components

```text
Storage
  ├── S3StorageService         — Ký presigned PUT/GET URL, HEAD/delete object
  ├── UploadValidationService  — Validate MIME allowlist, kích thước
  ├── StorageMetadataService   — Lưu object key, owner, MIME, size, type, state (TEMP/PERMANENT), timestamp
  └── OrphanCleanupJob         — Scheduled job xóa object type=CROP, state=TEMP quá 24h
```

### Trace

- FR: FR-11
- BF: BF-04 (avatar), BF-06 (scan image)

### Milestone: M1 (avatar), M2 (scan), M4 (production R2)

---

## SS-17: Admin — Dashboard quản trị backend

### Mô tả

API backend phục vụ CMS/Dashboard quản trị. Cung cấp các endpoint quản lý user, dictionary, gamification, template và thống kê hệ thống. Tất cả yêu cầu `ROLE_ADMIN`.

### Chức năng chính

- Quản lý người dùng: list, search, detail + tiến độ, ban/unban, reset password
- Quản lý từ điển: CRUD Word/Definition (soft-delete)
- Quản lý Collection/Topic
- Import dữ liệu hàng loạt
- Quản lý Topic Template hệ thống
- Quản lý gamification config
- Xem Feedback/báo lỗi từ Learner
- Dashboard thống kê: users active, AI usage, R2/S3 storage

### API Endpoints

| Method | Endpoint                              | Mô tả                                 | Auth  |
| ------ | ------------------------------------- | -------------------------------------- | ----- |
| GET    | `/admin/users`                        | Danh sách users                        | Admin |
| GET    | `/admin/users/{id}`                   | Chi tiết user + tiến độ                | Admin |
| POST   | `/admin/users/{id}/ban`               | Khóa tài khoản                        | Admin |
| POST   | `/admin/users/{id}/unban`             | Mở khóa                                | Admin |
| POST   | `/admin/users/{id}/reset-password`    | Reset mật khẩu                         | Admin |
| GET    | `/admin/dashboard/summary`            | Thống kê tổng quan                     | Admin |
| GET    | `/admin/dashboard/ai-usage`           | Mức sử dụng AI service                 | Admin |
| GET    | `/admin/dashboard/storage`            | Dung lượng R2/S3                       | Admin |
| GET    | `/admin/feedback`                     | Danh sách báo lỗi từ người dùng       | Admin |

### Ghi chú

- Admin không can thiệp tiến độ học tập cá nhân cụ thể của Learner.
- CRUD từ điển, topic, template, gamification config → delegate sang SS-04, SS-05, SS-09, SS-13, SS-14.
- Thống kê có thể dùng aggregate queries hoặc materialized view.

### Trace

- FR: FR-13
- BF: BF-14

### Milestone: M4 (parallel, không block M1–M3)

---

## SS-18: API Documentation — Tài liệu hóa API

### Mô tả

Cung cấp Swagger/OpenAPI tự động cho toàn bộ backend API, phục vụ kiểm thử và tích hợp mobile ↔ backend ↔ AI service.

### Chức năng chính

- Swagger UI (env-gated: off hoặc restrict trên production)
- API grouping theo tags: auth, user, word, storage, recognition, learning, gamification, admin
- DTO schema rõ trong OpenAPI spec
- Error response format thống nhất (`success/data/error/requestId`)
- Endpoint nội bộ (backend ↔ AI) tách rõ với endpoint public/mobile

### Trace

- FR: FR-12

### Milestone: M1 (bắt đầu), ongoing

---

## Dependency Graph — Quan hệ phụ thuộc

```text
                              ┌────────────────┐
                              │    SS-01:       │
                              │  MOBILE APP     │
                              └───────┬────────┘
                                      │ HTTP / REST API
                              ┌───────▼────────┐
                              │    SS-03:       │
                              │   IDENTITY      │◄─────────── SS-02: ADMIN CMS
                              │  (Auth/JWT)     │
                              └───┬──┬──┬──┬───┘
           ┌──────────────────────┘  │  │  └──────────────────────┐
           ▼                         ▼  ▼                          ▼
    ┌───────────────┐       ┌──────────────────┐          ┌────────────────┐
    │   SS-04:      │       │    SS-06:         │          │   SS-08:       │
    │  DICTIONARY   │◄──────│  RECOGNITION     │─────────►│  VOCABULARY    │
    │               │       │  (Orchestrator)   │          │  (Topic/Item)  │
    └───────┬───────┘       └────────┬──────────┘          └──┬──────┬─────┘
            │                        │                         │      │
    ┌───────┴───────┐       ┌────────▼──────────┐     ┌───────▼──┐  ┌▼────────────┐
    │   SS-05:      │       │    SS-07:         │     │ SS-09:   │  │ SS-10:      │
    │   TOPIC       │       │  AI SERVICE      │     │ FLASHCARD│  │ QUIZ        │
    │               │       │  (Florence-2)     │     │ &TEMPLATE│  │             │
    └───────────────┘       └───────────────────┘     └──┬───────┘  └──┬──────────┘
                                                          │             │
                                                     ┌────▼─────────────▼──┐
                                                     │       SS-11:        │
                                                     │    SRS (FSRS)       │
                                                     └─────────┬───────────┘
                                                               │
                                                     ┌─────────▼───────────┐
                                                     │       SS-12:        │
                                                     │    PROGRESS         │
                                                     └─────────┬───────────┘
                                                               │
                               ┌────────────────┐    ┌─────────▼───────────┐
                               │    SS-14:       │◄───│       SS-13:        │
                               │    SHOP         │    │   GAMIFICATION      │
                               └────────────────┘    └─────────┬───────────┘
                                                               │
                               ┌────────────────┐    ┌─────────▼───────────┐
                               │    SS-16:       │    │       SS-15:        │
                               │    STORAGE      │    │   NOTIFICATION      │
                               └────────────────┘    └─────────────────────┘

    Crosscutting: SS-16 (Storage) ← SS-03, SS-06, SS-09, SS-14
                  SS-17 (Admin) → SS-03, SS-04, SS-05, SS-09, SS-13, SS-14
                  SS-18 (API Docs) → All backend SS
```

### Mermaid Dependency Diagram

```mermaid
graph TD
    SS01["SS-01: Mobile App"] --> SS03["SS-03: Identity"]
    SS02["SS-02: Admin CMS"] --> SS03
    SS03 --> SS04["SS-04: Dictionary"]
    SS03 --> SS05["SS-05: Topic"]
    SS03 --> SS06["SS-06: Recognition"]
    SS03 --> SS08["SS-08: Vocabulary"]
    SS06 --> SS07["SS-07: AI Service"]
    SS06 --> SS04
    SS06 --> SS08
    SS05 --> SS08
    SS04 --> SS08
    SS08 --> SS09["SS-09: Flashcard & Template"]
    SS08 --> SS10["SS-10: Quiz"]
    SS09 --> SS11["SS-11: SRS"]
    SS10 --> SS11
    SS11 --> SS12["SS-12: Progress"]
    SS10 --> SS12
    SS09 --> SS12
    SS12 --> SS13["SS-13: Gamification"]
    SS13 --> SS14["SS-14: Shop"]
    SS13 --> SS15["SS-15: Notification"]
    SS11 --> SS15
    SS03 --> SS16["SS-16: Storage"]
    SS06 --> SS16
    SS09 --> SS16
    SS17["SS-17: Admin"] --> SS03
    SS17 --> SS04
    SS17 --> SS05
    SS17 --> SS09
    SS17 --> SS13
    SS17 --> SS14
```

---

### Quy tắc phụ thuộc chi tiết

| Từ (Source)            | Đến (Target)          | Quan hệ                                                                                   |
| ---------------------- | --------------------- | ------------------------------------------------------------------------------------------ |
| Mobile App → Identity  | SS-01 → SS-03         | Mọi API call đều qua JWT authentication                                                   |
| Admin CMS → Identity   | SS-02 → SS-03         | CMS dùng JWT với ROLE_ADMIN                                                                |
| Recognition → AI       | SS-06 → SS-07         | Recognition worker gọi FastAPI AI service qua HTTP nội bộ (timeout 60s); mobile theo dõi job bằng requestId |
| Recognition → Dictionary | SS-06 → SS-04       | Ánh xạ label AI → Word qua ObjectWordMappingService                                        |
| Recognition → Vocabulary | SS-06 → SS-08       | Learner lưu kết quả scan → tạo TopicItem + FsrsRecord                                     |
| Recognition → Storage  | SS-06 → SS-16         | Upload/access ảnh scan qua Object Storage                                                  |
| Dictionary → Vocabulary | SS-04 → SS-08       | Từ dictionary tra cứu → lưu thành TopicItem                                                |
| Topic → Vocabulary     | SS-05 → SS-08         | Từ topic hệ thống → sao chép/lưu thành TopicItem cá nhân (source=TOPIC)                    |
| Vocabulary → Flashcard | SS-08 → SS-09         | TopicItem là nguồn nạp thẻ học; Topic gán Template cấu hình render                         |
| Vocabulary → Quiz      | SS-08 → SS-10         | TopicItem là nguồn câu hỏi quiz                                                           |
| Flashcard → SRS        | SS-09 → SS-11         | Review session cập nhật trực tiếp FsrsRecord (card_state, due, stability, difficulty)      |
| Quiz → SRS             | SS-10 → SS-11         | Kết quả quiz không cập nhật thông số FSRS (chỉ ghi nhận QuizAttempt, progress, XP)          |
| SRS → Progress         | SS-11 → SS-12         | FsrsRecord state/due → aggregate progress                                                  |
| Quiz → Progress        | SS-10 → SS-12         | QuizAttempt → aggregate accuracy, XP                                                       |
| Flashcard → Progress   | SS-09 → SS-12         | Study session → cập nhật streak, learned count                                             |
| Progress → Gamification | SS-12 → SS-13       | Learning events trigger XP/coin/mission/badge rules                                       |
| Gamification → Shop    | SS-13 → SS-14         | Coin balance; item purchase                                                                |
| Gamification → Notification | SS-13 → SS-15   | Badge/mission complete → in-app + push notification                                       |
| SRS → Notification     | SS-11 → SS-15         | Due review → push notification nhắc nhở                                                   |
| Identity → Storage     | SS-03 → SS-16         | Avatar upload/access                                                                       |
| Flashcard → Storage    | SS-09 → SS-16         | Ảnh crop flashcard từ AI                                                                    |
| Admin → All domains    | SS-17 → SS-03..14     | CRUD user, dictionary, topic, template, gamification, shop                                 |
| API Docs → All backend | SS-18 → All           | OpenAPI spec cho tất cả endpoint                                                           |

### Ghi chú coupling

- **Loose coupling qua events:** Các phân hệ nên dùng domain event nội bộ (VD: `TopicItemCreated`, `ReviewCompleted`, `QuizSubmitted`, `MissionCompleted`) để tránh coupling trực tiếp.
- **Shared entities:** `TopicItem` và `FsrsRecord` được chia sẻ giữa SS-05/SS-08 (Topic & Item), SS-09 (Flashcard & Template) và SS-11 (SRS). Trách nhiệm tách rõ qua service layer.
- **AI Service tách deploy:** SS-07 là service độc lập (Python FastAPI), giao tiếp HTTP. Không chia sẻ database với backend Spring Boot.
- **Storage crosscutting:** SS-16 là infrastructure service, được nhiều domain sử dụng qua cùng interface.

---

## Mapping SS → FR → Milestone

| SS    | Tên phân hệ               | FR chính                     | Milestone     |
| ----- | -------------------------- | ---------------------------- | ------------- |
| SS-01 | Mobile App                 | FR-01 → FR-12 (consumer)    | M1 → M4      |
| SS-02 | Admin CMS                  | FR-13                        | M4            |
| SS-03 | Identity                   | FR-01                        | M1            |
| SS-04 | Dictionary                 | FR-03, FR-13.02              | M1            |
| SS-05 | Topic                      | FR-14                        | M1            |
| SS-06 | Recognition (Orchestrator) | FR-02                        | M2            |
| SS-07 | AI Service                 | FR-02.05, FR-02.06           | M2            |
| SS-08 | Vocabulary (Topic/Item)    | FR-04, FR-05.01              | M1–M2         |
| SS-09 | Flashcard & Template       | FR-05, FR-13.07              | M1, M3        |
| SS-10 | Quiz                       | FR-06                        | M3            |
| SS-11 | SRS (FSRS)                 | FR-07                        | M3            |
| SS-12 | Progress                   | FR-08                        | M3            |
| SS-13 | Gamification               | FR-09.01–05                  | M4            |
| SS-14 | Shop                       | FR-09.06–07                  | M4            |
| SS-15 | Notification               | FR-10                        | M3            |
| SS-16 | Storage                    | FR-11                        | M1, M2, M4    |
| SS-17 | Admin (Backend)            | FR-13                        | M4            |
| SS-18 | API Documentation          | FR-12                        | M1 (ongoing)  |

---

## Mapping với Package Structure (Backend — Spring Boot)

```text
vn.ptit.snapvocab
├── config/                         (SecurityConfig, ApplicationProperties, CloudflareR2Properties, CorsConfig...)
├── controller/                     (AuthenticationController, CollectionController, TopicController, WordController, ScanController, StorageController...)
├── domain/                         (Entity definitions & mappings)
│   ├── Authority, User, RefreshToken
│   ├── Word, Definition, Translation, Pronunciation, WordDefinition, WordRelation
│   ├── Collection, Topic, TopicItem, TopicAttributeGroup, TopicAttribute, TopicItemAttributeValue
│   ├── Template, TemplateElement, TemplateField
│   ├── FsrsRecord, Level, ShopItem, UserInventory, Notification, UserNotification
│   ├── common/                    (BaseTimeEntity, BaseCreatedAtEntity)
│   ├── enumeration/               (CardState, ReviewRating, SemanticRole, TemplateElementType, CollectionType...)
│   └── mapper/                    (Entity mappers & DTO converters)
├── repository/                    (JPA Repositories cho 25 entity tables)
├── security/                      (JwtTokenProvider, CustomUserDetailsService, SecurityUtils)
├── service/                       (AuthenticationService, CollectionService, TopicService, WordService, ScanService, StorageService...)
│   ├── dto/                       (Request/Response DTOs)
│   └── impl/                      (Service implementations)
└── util/                          (StringUtil, HeaderUtil, PaginationUtil...)
```

---

## Checklist tài liệu

- [x] 18 phân hệ bao phủ toàn bộ FR-01 → FR-14 trong [specs.md](./specs.md).
- [x] Mỗi SS có: mô tả, entities, chức năng chính, API endpoints, trace, milestone.
- [x] Actor đúng canonical: Guest, Learner, Admin.
- [x] Canonical model: Collection → Topic → TopicItem + Template + FsrsRecord. Không `SavedWord`/`UserWord`/`Deck`/`Note`/`Card`.
- [x] AI pipeline: Florence-2 zero-shot (CLIP tùy chọn). Không YOLO, không SAM.
- [x] SRS: FSRS trên FsrsRecord gắn cặp (user_id, topic_item_id).
- [x] SS-07 (AI Service) tách deploy, giao tiếp HTTP nội bộ.
- [x] Dependency graph + coupling notes.
- [x] Package structure mapping.
- [x] Milestone mapping M1–M4.
