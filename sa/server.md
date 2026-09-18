# Server & Deployment — SnapVocab

> Tài liệu server, môi trường triển khai và vận hành cho SnapVocab, được xây dựng dựa trên [specs.md](../spec/specs.md), [sa.md](./sa.md), [techstack.md](./techstack.md), [buss_mainflow.md](../spec/buss_mainflow.md), [phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md), [phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md) và [phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md).

>
> **Canonical sync (2026-08-23):** Source = [`../spec/specs.md`](../spec/specs.md). AI = Florence-2 zero-shot (+ CLIP tùy chọn; SAM đã gỡ) · Learning = Collection/Topic/TopicItem + Template + FsrsRecord · SRS = FSRS · Actors = Guest/Learner/Admin · FR: Game=09, Noti=10, Storage=11, OpenAPI=12, Admin=13 · 4 milestones.

---

## 1. Mục tiêu tài liệu

Mô tả cách tổ chức, triển khai và vận hành các thành phần server-side của SnapVocab:

- **Backend API** (Spring Boot) — nghiệp vụ chính, auth, learning, gamification.
- **AI Service** (FastAPI) — pipeline Florence-2 (CLIP tùy chọn) cho nhận diện vật thể.
- **Database** (MySQL/MariaDB) — source of truth cho toàn bộ dữ liệu nghiệp vụ.
- **Cache** (Redis/Redisson) — cache dictionary, leaderboard sorted set (chỉ dùng từ M3/M4).
- **Object Storage** (Cloudflare R2 / MinIO) — lưu avatar, ảnh scan, tài nguyên vật phẩm.
- **Mail Provider** — OTP/email verification, reset password.
- **API Documentation** (Swagger/OpenAPI) — kiểm thử và tích hợp mobile ↔ backend ↔ AI.
- **Observability** — logging, monitoring, health checks, backup, cleanup.

> **Quy tắc:** Tài liệu **không** chứa secrets, passwords, tokens hoặc endpoint nội bộ nhạy cảm thật.

---

## 2. Tổng quan thành phần server

### 2.1 Kiến trúc triển khai

```text
┌──────────────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS (Public)                                     │
│                                                                                  │
│  ┌──────────────────────────────┐    ┌──────────────────────────────┐            │
│  │  Mobile App (iOS / Android)  │    │  Admin CMS (Web Dashboard)   │            │
│  │  React Native / Expo         │    │  ROLE_ADMIN only              │            │
│  └──────────────┬───────────────┘    └──────────────┬───────────────┘            │
│                 │ HTTPS / JSON / JWT                 │ HTTPS / JWT                │
└─────────────────┼───────────────────────────────────┼────────────────────────────┘
                  │                                   │
                  ▼                                   ▼
┌────────────────────────────────────────────────────────────────────────────────────┐
│                     BACKEND API SERVER (Public via HTTPS)                           │
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │  Spring Boot REST API (Java 17)                                               │  │
│  │  Spring Security + JWT + Refresh Token + OTP                                  │  │
│  │  JPA/Hibernate + MapStruct + Springdoc OpenAPI                                │  │
│  │                                                                               │  │
│  │  Modules: Identity, Dictionary, Topic, Recognition, Vocabulary,               │  │
│  │           Flashcard, Quiz, SRS, Progress, Gamification, Shop,                 │  │
│  │           Notification, Storage, Admin, API Docs                              │  │
│  └──────┬──────────┬──────────┬──────────┬──────────┬────────────────────────────┘  │
│         │          │          │          │          │                                │
└─────────┼──────────┼──────────┼──────────┼──────────┼────────────────────────────────┘
          │          │          │          │          │
   ┌──────▼──────┐ ┌─▼────────┐│ ┌────────▼────────┐│ ┌─────────────────────────┐
   │ MySQL/      │ │ Redis    ││ │ Object Storage  ││ │ FastAPI AI Service      │
   │ MariaDB     │ │ (Redisson│││ │ (R2 / MinIO)   ││ │ (Florence-2,CLIP opt.) │
   │             │ │         )│││ │ S3-compatible   ││ │ Python · GPU T4+       │
   │ Source of   │ │ Cache,   │││ │ Private bucket  ││ │ Internal/private       │
   │  Truth      │ │ Ldrboard ││ │                 ││ │                         │
   └─────────────┘ └──────────┘│ └─────────────────┘│ └─────────────────────────┘
                               │                    │
                        ┌──────▼─────┐              │
                        │ Mail       │              │
                        │ Provider   │──────────────┘
                        │ (SMTP/API) │
                        └────────────┘
```

### 2.2 Bảng thành phần

| # | Thành phần | Runtime / Công nghệ | Vai trò | Public? |
| --- | --- | --- | --- | :---: |
| 1 | Backend API | Java 17, Spring Boot | REST API nghiệp vụ cho mobile & admin CMS | Có qua HTTPS |
| 2 | AI Service | Python, FastAPI, Florence-2 (base/large) + CLIP ViT-B/32 (tùy chọn) | Nhận diện vật thể từ ảnh (zero-shot) | Không (Internal) |
| 3 | Database | MySQL/MariaDB | Lưu dữ liệu nghiệp vụ — source of truth | Không (Private) |
| 4 | Cache | Redis/Redisson | Cache dictionary, leaderboard, home summary, rate limit | Không (Private) |
| 5 | Object Storage | Cloudflare R2 (prod) / MinIO (dev), S3-compatible | Avatar, ảnh scan, tài nguyên vật phẩm | Không (Private bucket) |
| 6 | Mail Provider | SMTP hoặc transactional email API | OTP, xác thực email, reset password, security notice | Backend → outbound |
| 7 | API Docs | Swagger/OpenAPI (Springdoc) | Tài liệu API backend cho mobile & AI tích hợp | Dev/staging; restricted prod |
| 8 | Observability | Logs/metrics/alerts | Theo dõi lỗi, hiệu năng, bảo mật | Không (Internal) |

---

## 3. Môi trường triển khai

### 3.1 Local Development

**Mục tiêu:** Lập trình, kiểm thử API và chạy thử mobile app trên máy phát triển.

```text
Expo Dev Client (mobile)
  → Backend API (localhost:8080)
  → MySQL/MariaDB local
  → Redis local
  → MinIO local (S3-compatible storage)
  → FastAPI AI Service local (Florence-2-base, OD + self-grounding — chạy được trên CPU)
  → SMTP sandbox (Mailtrap / MailHog / Ethereal)
  → Swagger UI: enabled
```

**Đặc điểm:**

| Hạng mục | Cấu hình |
| --- | --- |
| Swagger UI | Bật — test API tự do |
| CORS | Cho phép origin dev (Expo tunnel / LAN IP) |
| Object Storage | MinIO hoặc S3-compatible local |
| AI Service | Florence-2-base trên CPU (`.env.example`); prod dùng `.env.gpu` trên GPU |
| Dictionary | Import subset để test nhanh; full import khi cần test dictionary flow |
| Mail | Sandbox — không gửi email thật |
| Secrets | File `.env` local, **không** commit vào VCS |

### 3.2 Staging / UAT

**Mục tiêu:** Kiểm thử gần production trước khi demo/nghiệm thu.

```text
Mobile build staging (Expo preview / APK)
  → HTTPS Backend API staging
  → Staging MySQL/MariaDB
  → Staging Redis
  → Staging object bucket (R2 test hoặc MinIO staging)
  → AI Service staging (GPU T4 — full pipeline)
  → Mail provider staging/test mode
  → Swagger UI: enabled
```

**Đặc điểm:**

| Hạng mục | Cấu hình |
| --- | --- |
| HTTPS | Bắt buộc (cert staging) |
| Dữ liệu | Test data gần thật, **không** chứa thông tin nhạy cảm thật |
| Dictionary | Full import (357K+ từ) |
| AI Service | Cấu hình `.env.gpu` (Florence-2-large) trên GPU |
| Log level | Chi tiết hơn production nhưng **không** log secret/password/token/OTP |
| Swagger UI | Bật — kiểm thử tích hợp end-to-end |
| E2E test flows | Auth → scan → save word → flashcard → quiz → SRS → gamification → storage |

### 3.3 Production

**Mục tiêu:** Vận hành ổn định cho người dùng thật.

```text
Mobile App Store / APK
  → HTTPS Backend API (reverse proxy / load balancer)
  → Managed MySQL/MariaDB (private network)
  → Redis service (private network)
  → Cloudflare R2 private bucket
  → FastAPI AI Service container/VM (private network, GPU T4+)
  → Mail provider production (transactional email)
  → Centralized logs / metrics / alerts
  → Swagger UI: off hoặc restricted
```

**Đặc điểm:**

| Hạng mục | Cấu hình |
| --- | --- |
| HTTPS | Bắt buộc cho Backend API |
| Private | Database, Redis, AI service **không** public trực tiếp |
| Object Storage | Cloudflare R2 private bucket; truy cập qua presigned URL (TTL ≤ 15 phút) |
| Secrets | Quản lý bằng secret manager hoặc env variables an toàn |
| Swagger UI | Tắt hoặc bảo vệ bằng IP allowlist / auth |
| CORS | Chỉ allow origin cần thiết; không wildcard với credential |
| Log | Structured logging (JSON); centralized aggregation |
| Backup | Backup DB định kỳ; R2 lifecycle nếu cần |
| Monitoring | Error rate, latency, AI timeout, DB/Redis availability |

---

## 4. Backend API Server

### 4.1 Runtime

| Thuộc tính | Giá trị |
| --- | --- |
| Runtime | Java 17 |
| Framework | Spring Boot |
| API style | RESTful JSON API |
| Security | Spring Security + JWT (access + refresh token) |
| ORM | Spring Data JPA / Hibernate |
| DTO Mapping | MapStruct |
| Validation | Spring Validation (jakarta.validation) |
| API Docs | Springdoc OpenAPI (Swagger UI) |
| Mail | Spring Mail / JavaMailSender |
| Build | Maven |

### 4.2 Module và API endpoints

Tên endpoint cụ thể (bao gồm HTTP method, path, request/response payload) vui lòng tham khảo nguồn duy nhất (Single Source of Truth) tại tài liệu `phan_ra_phan_he_he_thong.md`. Bảng dưới đây mô tả mapping module theo phân hệ.

| Module (SS) | Mục đích chính | Auth | Milestone |
| --- | --- | --- | --- |
| **Identity (SS-03)** | Register, login, refresh, OTP, forgot/reset password | Public | M1 |
| | Profile view/edit | Learner | M1 |
| **Dictionary (SS-04)** | Search word, word detail, pronunciation, translation | Learner | M1 |
| **Topic (SS-05)** | Collections, Topics, TopicItems browse | Learner | M1 |
| **Storage (SS-16)** | Presigned upload, upload complete, access URL | Learner | M1 |
| **Recognition (SS-06)** | `/api/scan`: upload-url, submit (quota + hàng đợi), poll kết quả, quota | Learner | M2 |
| **Vocabulary (SS-08)** | CRUD Topic cá nhân, TopicItem, gán EAV attributes | Learner | M1–M2 |
| **Flashcard (SS-09)** | Flashcard study session theo Template, recall rating | Learner | M1, M3 |
| **Quiz (SS-10)** | Quiz setup, play, result, history | Learner | M3 |
| **SRS (SS-11)** | Review queue, submit FSRS rating trên FsrsRecord | Learner | M3 |
| **Progress (SS-12)** | Home summary, streak, accuracy, activity history | Learner | M3 |
| **Gamification (SS-13)** | Missions, badges, XP, leaderboard | Learner | M4 |
| **Shop (SS-14)** | Shop browse/buy, inventory, coin balance | Learner | M4 |
| **Notification (SS-15)** | List/read notifications, device token register | Learner | M3 |
| **Admin (SS-17)** | User mgmt, dict CRUD, topic, template, game config, dashboard | Admin | M4 |
| **AI Service (SS-07)** | Backend → AI (Internal: `POST /api/v1/detect`, `GET /health`) | Internal | M2 |

### 4.3 API response envelope

Tất cả backend API trả JSON envelope thống nhất (NFR §12 — specs.md):

```json
{
  "success": true,
  "data": { },
  "error": null,
  "requestId": "uuid-v4"
}
```

Error response:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "RECOGNITION_TIMEOUT",
    "message": "AI service xử lý quá thời gian cho phép.",
    "details": null
  },
  "requestId": "uuid-v4"
}
```

### 4.4 Backend configuration

| Nhóm | Biến cấu hình (ví dụ) | Ghi chú |
| --- | --- | --- |
| **App** | `APP_ENV`, `APP_BASE_URL`, `API_BASE_PATH` | Theo môi trường (mặc định `API_BASE_PATH=/api/v1`) |
| **Database** | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD` | Không commit secret |
| **Redis** | `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` | Password nếu môi trường yêu cầu |
| **JWT** | `JWT_SECRET`, `JWT_ACCESS_TTL`, `JWT_REFRESH_TTL` | Secret đủ mạnh, rotate được |
| **Mail** | `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_FROM` | SMTP hoặc API provider |
| **Storage** | `STORAGE_ENDPOINT`, `STORAGE_BUCKET`, `STORAGE_ACCESS_KEY`, `STORAGE_SECRET_KEY`, `STORAGE_REGION`, `PRESIGNED_URL_TTL` | R2/MinIO/S3-compatible |
| **AI Service** | `AI_SERVICE_URL`, `AI_SERVICE_READ_TIMEOUT_MS`, `AI_SERVICE_TOKEN` | Timeout 60s (prod) / 120s (dev) |
| **Scan** | `scan.quota-per-day`, `scan.zone`, `scan.workers`, `scan.queue-capacity`, `scan.stale-margin-seconds`, `scan.min-reliability` | Trong `application.yaml`; mặc định 20 / Asia/Ho_Chi_Minh / 1 / 3 / 30 / LOW |
| **Upload** | `MAX_AVATAR_SIZE`, `MAX_SCAN_SIZE`, `ALLOWED_IMAGE_TYPES` | Avatar ≤ 5MB, scan ≤ 10MB |
| **OpenAPI** | `OPENAPI_ENABLED`, `SWAGGER_UI_ENABLED` | Tắt/hạn chế production |
| **CORS** | `CORS_ALLOWED_ORIGINS` | Không wildcard với credential |
| **OTP** | `OTP_TTL_MINUTES`, `OTP_MAX_ATTEMPTS`, `OTP_RESEND_COOLDOWN_SECONDS` | TTL ≤ 10m, max 5, cooldown ≥ 60s |

**Quy tắc:**
- Không commit secrets vào VCS.
- Không copy password/token thật vào tài liệu.
- Tách config local/staging/production.
- Production secrets quản lý bằng secret manager hoặc env an toàn.

---

## 5. AI Service Server

### 5.1 Runtime

| Thuộc tính | Giá trị |
| --- | --- |
| Runtime | Python 3.10+ |
| Framework | FastAPI |
| Server | Uvicorn, **1 process** (model nạp một lần, dùng chung) |
| Model | Florence-2-base (CPU/dev) hoặc Florence-2-large (GPU, `.env.gpu`); CLIP ViT-B/32 chỉ nạp khi bật từ vựng nền |
| Mode | Zero-shot (không fine-tune trong MVP) |
| Thời gian | Florence-2-base, OD + self-grounding: ~35–45s/ảnh trên CPU; GPU nhanh hơn nhiều |
| Đồng thời | 1 ảnh tại một thời điểm (semaphore); request thêm xếp hàng trong AI service |
| API visibility | Internal/private — **không** public trực tiếp |

### 5.2 Pipeline xử lý

```text
Input Image (multipart từ backend)
  → Xoay theo EXIF, resize cạnh dài ≤ MAX_INPUT_SIZE
  → Florence-2: <OD> + self-grounding            (mặc định)
      [+ tiled OD, dense caption, từ vựng nền + CLIP — bật qua biến môi trường]
  → Loại box < MIN_BOX_AREA_RATIO (0,4%) hoặc > MAX_BOX_AREA_RATIO (85%)
  → Lọc nhãn theo từ điển (WordNet)
  → NMS 2 tầng + mỗi nhãn giữ 1 box
  → Gắn headword + reliability
  → [Vẽ box lên 1 ảnh nếu RETURN_ANNOTATED_IMAGE=true — chỉ phục vụ endpoint đồng bộ cũ]
```

### 5.3 Internal API contract

**POST `/api/v1/detect`** — Nhận diện vật thể

Input: `multipart/form-data`, field `file` (`image/jpeg`, `image/png`, `image/webp`, ≤ `MAX_UPLOAD_MB`).

| Header | Bắt buộc | Mô tả |
| --- | --- | --- |
| `X-Request-Id` | Không | `requestId` của backend (UUID của `ScanRequest`); thiếu thì AI tự sinh |
| `X-Service-Token` | Khi AI đặt `SERVICE_TOKEN` | Phải trùng `SERVICE_TOKEN` |

Output:

| Field | Type | Mô tả |
| --- | --- | --- |
| `request_id` | string | Trả lại `X-Request-Id` |
| `model_version` | string | Model + các bước bật, vd `Florence-2-base/od+self` |
| `processing_time_ms` | int | Thời gian chạy model, không tính thời gian xếp hàng |
| `detections[]` | array | Danh sách object (rỗng nếu không nhận diện được — **không** phải lỗi) |
| `detections[].label` | string | Nhãn tiếng Anh, đã qua lọc từ điển |
| `detections[].headword` | string | Từ cuối của nhãn ở dạng số ít, để tra dự phòng |
| `detections[].score` | float | Hằng số theo nguồn, chỉ để xếp hạng nội bộ |
| `detections[].source` | string | `od`, `od_tile`, `self`, `dense`, `base` |
| `detections[].reliability` | string | `HIGH` (od, od_tile) / `MEDIUM` (self, dense) / `LOW` (base); đổi được qua `RELIABILITY_BY_SOURCE` |
| `detections[].box` | object | `{x1, y1, x2, y2}` theo pixel |
| `labels` | string[] | Danh sách nhãn (tiện cho debug) |
| `image_width`, `image_height` | int | Hệ toạ độ của mọi box: ảnh **đã xoay EXIF** rồi resize |
| `annotated_image_base64`, `annotated_image_mime` | string \| null | Ảnh vẽ sẵn box, chỉ có khi bật `RETURN_ANNOTATED_IMAGE` |

Error response — luôn có dạng `{"error": {"code": "...", "message": "..."}}`:

| HTTP | `code` | Mô tả | Backend ánh xạ thành |
| --- | --- | --- | --- |
| 400 | `INVALID_REQUEST` | Request sai dạng (thiếu `file`) | `AI_ERROR` (lỗi phía backend) |
| 400 / 413 / 415 | `INVALID_IMAGE` | Không giải mã được / quá dung lượng / sai định dạng | `INVALID_IMAGE` |
| 401 | `UNAUTHORIZED` | Thiếu hoặc sai `X-Service-Token` | `AI_ERROR` (sai cấu hình) |
| 503 | `MODEL_NOT_READY` | Model đang nạp | `AI_UNAVAILABLE` |
| 500 | `MODEL_ERROR` | Lỗi khi chạy model (vd. CUDA out of memory) | `AI_ERROR` |

Không kết nối được hoặc hết thời gian kết nối → `AI_UNAVAILABLE`; hết thời gian đọc (`ai-service.read-timeout-ms`) → `AI_TIMEOUT`.

**GET `/health`** — Health check (không cần token)

```json
{
  "status": "ok",
  "device": "cuda",
  "gpu_available": true,
  "model_version": "Florence-2-base/od+self",
  "florence_loaded": true,
  "clip_loaded": false
}
```

`status = "loading"` khi model chưa nạp xong.

### 5.4 AI service hardening

| Hạng mục | Yêu cầu |
| --- | --- |
| Network | Không public trực tiếp cho mobile; chỉ backend gọi. CORS mặc định tắt |
| Auth | `SERVICE_TOKEN` (AI) = `ai-service.service-token` (backend, env `AI_SERVICE_TOKEN`); bắt buộc nếu cổng AI ra được mạng ngoài |
| Input | Giới hạn 10MB ở cả backend (`ai-service.max-upload-mb`) và AI (`MAX_UPLOAD_MB`) |
| Timeout | Backend chờ AI tối đa 60s ở prod (120s ở dev), đổi qua `AI_SERVICE_READ_TIMEOUT_MS`; mobile poll tối đa 90s |
| Logging | Log `request_id`, `model_version`, số object, `time_ms`, lỗi; backend log cùng `requestId` (MDC) |
| Storage | **Không** lưu ảnh trong AI service |
| Scaling | 1 process/GPU; khuyến nghị Spot Instance hoặc Serverless GPU (scale-to-0) |
| Failover | Mọi lỗi AI thành mã lỗi scan có nghĩa; job `FAILED` không tính quota |

### 5.5 AI configuration

Chi tiết và số đo ở `ai-service/README.md`. Các biến chính:

| Biến | Mặc định | Mô tả |
| --- | --- | --- |
| `DEVICE` | `cuda` | `cuda` / `cpu` / `auto` |
| `FLORENCE_MODEL_ID` | `microsoft/Florence-2-base` | `Florence-2-large` trong `.env.gpu` |
| `CLIP_MODEL_ID` | `openai/clip-vit-base-patch32` | Chỉ nạp khi `ENABLE_BASE_VOCAB=true` |
| `ENABLE_SELF_GROUNDING` | `true` | Giá trị tốt nhất trên thời gian |
| `ENABLE_TILED_OD` / `ENABLE_DENSE_CAPTION` / `ENABLE_BASE_VOCAB` | `false` | Tăng độ phủ, tốn thêm thời gian |
| `FLORENCE_NUM_BEAMS` | `3` | Hạ xuống 1 nhanh hơn 24% nhưng mất 46% nhãn đúng |
| `FLORENCE_BATCH_SIZE` | `4` | Hạ trước tiên khi gặp CUDA OOM |
| `MAX_INPUT_SIZE` | `1400` | Chỉ có tác dụng khi bật tiled OD |
| `MAX_UPLOAD_MB` | `10` | Giới hạn ảnh đầu vào |
| `RELIABILITY_BY_SOURCE` | xem §5.3 | JSON map `source → reliability` |
| `RETURN_ANNOTATED_IMAGE` | `true` | Tắt khi mobile đã chuyển hẳn sang luồng mới |
| `SERVICE_TOKEN` | rỗng | Token xác thực backend |
| `CORS_ORIGINS` | `[]` | Chỉ cần cho UI debug cục bộ |

---

## 6. Database Server

### 6.1 Công nghệ

| Thuộc tính | Giá trị |
| --- | --- |
| Engine | MySQL hoặc MariaDB |
| Access | Backend truy cập qua JPA/Hibernate |
| Role | Source of truth cho toàn bộ dữ liệu nghiệp vụ |
| Migration | SQL scripts hoặc Flyway/Liquibase nếu bổ sung |

### 6.2 Nhóm bảng dữ liệu

| Nhóm | Bảng/Entity tiêu biểu | SS tham chiếu |
| --- | --- | --- |
| **Identity** | `users`, `authorities`, `refresh_tokens`, `otp_tokens` | SS-03 |
| **Dictionary** | `words`, `definitions`, `translations`, `pronunciations`, `word_relations`, `object_word_mappings` | SS-04 |
| **Topic & Vocabulary** | `collections`, `topics`, `topic_items`, `topic_attribute_groups`, `topic_attributes`, `topic_item_attribute_groups`, `topic_item_attribute_values` | SS-05, SS-08 |
| **Recognition** | `image_recognition_requests`, `recognition_results`, `detected_objects`, `scan_histories` | SS-06 |
| **Template & SRS** | `templates`, `template_elements`, `template_fields`, `fsrs_records` | SS-09, SS-11 |
| **Quiz** | `quizzes`, `quiz_questions`, `quiz_attempts` | SS-10 |
| **Progress** | `learning_events`, `learning_progress` | SS-12 |
| **Gamification** | `missions`, `mission_progress`, `badges`, `user_badges`, `experience_logs`, `coin_transactions`, `leaderboard_entries` | SS-13 |
| **Economy** | `shop_items`, `user_inventories`, `levels` | SS-14 |
| **Media** | `storage_metadata` (object key, owner, MIME, size, type, state, timestamp) | SS-16 |
| **Notification** | `notifications`, `user_notifications` | SS-15 |

### 6.3 Indexing đề xuất

| Use case | Index gợi ý | Lý do |
| --- | --- | --- |
| Login | UNIQUE `users.email` | Lookup nhanh + chống trùng |
| Dictionary search | INDEX trên `words.word` (normalized); FULLTEXT nếu DB hỗ trợ | Lookup p95 < 500ms |
| Personal vocabulary | UNIQUE `(topic_id, word_id)` trên `topic_items` | Chống trùng từ vựng trong cùng một Topic |
| SRS review queue | INDEX `(user_id, due_at, card_state)` trên `fsrs_records` | Daily review query |
| Quiz history | INDEX `(user_id, created_at)` trên `quiz_attempts` | Pagination |
| Learning events | INDEX `(user_id, event_type, created_at)` | Progress aggregate |
| Leaderboard | INDEX `(scope, score)` hoặc Redis sorted set | Ranking query |
| Media owner | INDEX `(owner_id, media_type)` trên `storage_metadata` | User media lookup |
| Notifications | INDEX `(user_id, is_read, created_at)` trên `user_notifications` | Notification list |
| Object word mapping | INDEX `(label)` trên `object_word_mappings` | AI label → Word lookup |

### 6.4 Dictionary import

| Thuộc tính | Chi tiết |
| --- | --- |
| Nguồn | SQLite từ minhqnd/dictionary |
| Quy mô | ~357,729+ từ vựng, ~443,116+ định nghĩa |
| Xử lý | Chuẩn hóa, batch import vào MySQL/MariaDB, ánh xạ sang Word/Definition/Translation/Pronunciation |
| Chất lượng | Xử lý trùng lặp, thiếu nghĩa, thiếu phiên âm, bản dịch không phù hợp |
| Import quy trình | Test trên subset trước → kiểm tra encoding/duplicate/performance → import full |
| Index | Tạo index **sau** import batch để tối ưu tốc độ import |

### 6.5 Backup & migration

| Hạng mục | Chiến lược |
| --- | --- |
| Backup | Backup database định kỳ theo môi trường production |
| Restore test | Kiểm thử restore trước demo/production |
| Migration | SQL scripts hoặc Flyway/Liquibase quản lý version schema |
| Dictionary source | Lưu import script/source để tái import khi cần |
| Rebuild aggregate | LearningEvent/Progress/Leaderboard rebuild được từ dữ liệu nguồn khi aggregate lệch |

---

## 7. Redis Server

### 7.1 Use cases

| Use case | Cơ chế Redis | TTL đề xuất | Ghi chú |
| --- | --- | --- | --- |
| Dictionary cache | Key-value (word detail/translation) | 1–24h | Cache từ phổ biến, giảm DB query |
| Leaderboard | Sorted set (ZADD/ZRANGEBYSCORE) | 5–15 phút | Ranking global theo Weekly XP |
| Home summary | Key-value aggregate cache | 1–5 phút | Giảm query aggregate nặng |
| Mission state | Key-value tạm thời | 5 phút | Không thay thế DB |
| AI label → Word mapping | Key-value | 24h | Cache mapping label → Word ID |
| Rate limiting | Counter (INCR/EXPIRE) | Window-based | OTP/login/upload nếu triển khai |
| Distributed lock | Redisson lock | Auto-release | Tránh cộng reward trùng trong concurrent |

### 7.2 Quy tắc

- Redis **không** là source of truth cho dữ liệu quan trọng.
- Mọi dữ liệu Redis phải rebuild được từ database/events.
- Key prefix theo môi trường: `snapvocab:{env}:{module}:{key}`.
- TTL rõ ràng cho mọi cache tạm thời.
- Redis unavailable → backend degrade gracefully (query DB trực tiếp), không crash.

### 7.3 Configuration

| Biến | Mô tả |
| --- | --- |
| `REDIS_HOST` | Redis server host |
| `REDIS_PORT` | Redis server port (default: 6379) |
| `REDIS_PASSWORD` | Password nếu yêu cầu |
| `REDIS_DATABASE` | DB index (0–15) |
| `REDIS_KEY_PREFIX` | Prefix key theo môi trường |

---

## 8. Object Storage Server

### 8.1 Topology

| Môi trường | Công nghệ | Ghi chú |
| --- | --- | --- |
| Dev/Local | MinIO hoặc S3-compatible storage | Mô phỏng R2/S3 |
| Production | Cloudflare R2 qua S3-compatible API | Bucket private |

### 8.2 Bucket policy

| Quyết định | Mô tả |
| --- | --- |
| Private bucket | **Không** public toàn bucket |
| Presigned upload | Mobile upload qua URL ngắn hạn do backend cấp |
| Presigned read | Media private được đọc qua URL ngắn hạn (TTL ≤ 15 phút) |
| Object key | Backend sinh UUID-based key, **không** dùng tên file user |
| Metadata DB | Database `storage_metadata` lưu owner, type, MIME, size, object key |

### 8.3 Loại media

| Media type | Owner | Giới hạn | Source flow | Milestone |
| --- | --- | --- | --- | --- |
| Avatar | User | ≤ 5MB, image/* | Edit profile | M1 |
| Scan image | User | ≤ 10MB, jpeg/png/webp | `scans/{userId}/{uuid}.{ext}` qua `POST /api/scan/upload-url`; chưa có job dọn | M2 |
| Item asset | System/ShopItem | — | Admin upload | M4 |

### 8.4 Upload flow chi tiết

```text
1. Mobile → POST /storage/upload-init
   Request: { mediaType, mimeType, fileSize }
   Response: { uploadId, objectKey, presignedPutUrl, expiresAt }

2. Mobile → PUT presignedPutUrl (upload trực tiếp tới Object Storage)

3. Mobile → POST /storage/upload-complete
   Request: { uploadId, objectKey }
   Backend:
     → HEAD object (kiểm tra tồn tại)
     → Validate MIME allowlist (image/jpeg, image/png, image/webp)
     → Validate file size (≤ limit theo media type)
     → Lưu StorageMetadata (owner, objectKey, mime, size, type, createdAt)
     → Link object key tới entity (User.avatarKey, TopicItem.attributeValues, ShopItem.iconKey)
   Response: { objectKey, accessUrl (presigned GET, TTL ≤ 15m) }

4. GET /storage/access-url/{objectKey}
   → Backend sinh presigned GET URL (TTL ≤ 15 phút)
   → Validate owner/permission
```

### 8.5 Upload constraints & validation

| Hạng mục | Yêu cầu |
| --- | --- |
| MIME allowlist | `image/jpeg`, `image/png`, `image/webp` (cấu hình được) |
| Size limit | Avatar ≤ 5MB, Scan ≤ 10MB (cấu hình được) |
| Object key | UUID-based, backend sinh — không tin tên file client |
| Server-side validate | Backend HEAD + validate ở upload-complete; **không** tin metadata client |
| Presigned TTL | Upload URL: 10–15 phút; Download URL: ≤ 15 phút |

### 8.6 Cleanup

| Job | Mô tả | Frequency |
| --- | --- | --- |
| Orphan cleanup | Xóa object không còn tham chiếu trong DB | Daily/scheduled |
| Upload session cleanup | Xóa upload session hết hạn (chưa complete) | Hourly |
| Old avatar cleanup | Xóa avatar cũ sau khi user đổi avatar thành công | Event-driven |

### 8.7 Configuration

| Biến | Mô tả |
| --- | --- |
| `STORAGE_ENDPOINT` | R2/MinIO endpoint URL |
| `STORAGE_BUCKET` | Tên bucket |
| `STORAGE_ACCESS_KEY` | Access key |
| `STORAGE_SECRET_KEY` | Secret key |
| `STORAGE_REGION` | Region (VD: `auto` cho R2) |
| `PRESIGNED_UPLOAD_TTL` | TTL presigned upload URL (s) |
| `PRESIGNED_ACCESS_TTL` | TTL presigned download URL (s) |
| `MAX_AVATAR_SIZE_BYTES` | Giới hạn avatar |
| `MAX_SCAN_SIZE_BYTES` | Giới hạn scan image |
| `ALLOWED_IMAGE_TYPES` | MIME allowlist |

---

## 9. Mail Server / Provider

### 9.1 Use cases

| Use case | Nội dung | BF tham chiếu |
| --- | --- | --- |
| Signup verification | Gửi OTP/link xác thực tài khoản | BF-01 |
| Forgot password | Gửi OTP/link reset password | BF-03 |
| Security notice | Thông báo đổi mật khẩu hoặc login bất thường (nếu triển khai) | BF-02 |

### 9.2 Quy tắc

| Quy tắc | Chi tiết |
| --- | --- |
| OTP TTL | ≤ 10 phút |
| OTP max attempts | 5 lần thử trước khi khóa OTP hiện tại |
| Resend cooldown | ≥ 60 giây |
| OTP one-time | Không tái sử dụng sau xác thực thành công |
| Security | Không gửi password qua email |
| Logging | **Không** log OTP plaintext; production tuyệt đối tránh |
| Rate limit | Giới hạn gửi OTP theo email/IP/user nếu triển khai |
| Sender | Dùng domain chính thức, có SPF/DKIM nếu production |

### 9.3 Configuration

| Biến | Mô tả |
| --- | --- |
| `MAIL_HOST` | SMTP host |
| `MAIL_PORT` | SMTP port (587/465) |
| `MAIL_USERNAME` | SMTP username |
| `MAIL_PASSWORD` | SMTP password |
| `MAIL_FROM` | Sender email address |
| `MAIL_TLS_ENABLED` | TLS/STARTTLS |

---

## 10. Network Topology

### 10.1 Luồng public

```text
Mobile App ──(HTTPS)──→ Backend API Server
Admin CMS  ──(HTTPS)──→ Backend API Server
```

Backend API là **cổng public duy nhất**. Mobile và CMS **không** gọi trực tiếp database, Redis, AI service hoặc object storage (ngoại trừ presigned URL do backend cấp).

### 10.2 Luồng nội bộ

```text
Backend API ──→ MySQL/MariaDB     (JDBC, private network)
Backend API ──→ Redis              (Redis protocol, private network)
Backend API ──→ Object Storage     (S3-compatible API, private network hoặc public endpoint R2)
Backend API ──→ AI Service         (HTTP, private network hoặc service token)
Backend API ──→ Mail Provider      (SMTP/API, outbound)
```

### 10.3 Luồng presigned (mobile → storage trực tiếp)

```text
Mobile ──→ Backend: POST /storage/upload-init (nhận presigned PUT URL)
Mobile ──→ Object Storage: PUT presignedUrl (upload binary)
Mobile ──→ Backend: POST /storage/upload-complete (xác nhận)

Mobile ──→ Backend: GET /storage/access-url/{key} (nhận presigned GET URL)
Mobile ──→ Object Storage: GET presignedUrl (download binary)
```

### 10.4 Quy tắc network

| Quy tắc | Chi tiết |
| --- | --- |
| Backend là gateway | Mobile không gọi trực tiếp DB/Redis/AI |
| AI service private | Ưu tiên internal network; service token nếu public |
| Database private | Chỉ backend truy cập |
| Redis private | Chỉ backend truy cập |
| Storage | Private bucket; presigned URL ngắn hạn cho mobile upload/download |
| Mail | Backend gọi ra ngoài (outbound only) |

---

## 11. Security Configuration

### 11.1 Ma trận bảo mật theo NFR

| # | Hạng mục | Yêu cầu (truy vết NFR specs.md) |
| --- | --- | --- |
| 1 | HTTPS | Production bắt buộc HTTPS cho Backend API (NFR §1) |
| 2 | JWT | Access token bảo vệ API cá nhân; secret đủ mạnh, lưu env/secret manager, rotate được (NFR §1) |
| 3 | Refresh token | TTL hữu hạn, revoke khi logout/reset password/rủi ro bảo mật (NFR §1) |
| 4 | Password | Hash bằng bcrypt/argon2; **không** plaintext; **không** log (NFR §1) |
| 5 | OTP safety | TTL ≤ 10m, max 5 attempts, resend ≥ 60s, one-time use, **không** log (NFR §3) |
| 6 | Authorization | Learner chỉ dữ liệu cá nhân; Admin API yêu cầu `ROLE_ADMIN` (NFR §2) |
| 7 | File upload | Validate MIME allowlist ảnh; avatar ≤ 5MB, scan ≤ 10MB; object key backend sinh (NFR §5) |
| 8 | Privacy | Bucket private; presigned URL TTL ≤ 15 phút; ảnh scan/avatar không public mặc định (NFR §6) |
| 9 | AI service | Internal/private; service token nếu public network (NFR §1) |
| 10 | Idempotency | Reward/XP/coin/mission claim + quiz submit dùng event key; retry không cộng trùng (NFR §4) |
| 11 | Swagger | Bật dev/staging; production tắt hoặc bảo vệ IP/auth (NFR §1) |
| 12 | CORS | Chỉ allow origin cần thiết; không wildcard với credential |
| 13 | Logs | **Không** log password, token, OTP, secrets, presigned URL dài hạn (NFR §17) |
| 14 | Error response | Sai credential → generic message (chống email enumeration); error code có cấu trúc (NFR §10) |

### 11.2 Authorization matrix

| Resource | Guest | Learner | Admin |
| --- | --- | --- | --- |
| Auth endpoints (register/login/OTP/reset) | Có | — | — |
| Profile (own) | — | Có (owner) | — |
| Dictionary search/detail | — | Có | Có |
| Recognition scan | — | Có | — |
| Vocabulary Topic cá nhân (own) | — | Có (owner) | — |
| Flashcard/Quiz/SRS (own) | — | Có (owner) | — |
| Progress/Gamification (own) | — | Có (owner) | — |
| Shop/Wallet (own) | — | Có (owner) | — |
| Notifications (own) | — | Có (owner) | — |
| Storage upload (own media) | — | Có | Có |
| Admin CMS APIs | — | — | Có (ROLE_ADMIN) |

---

## 12. Health Check & Readiness

### 12.1 Health endpoints

| Check | Endpoint/Cơ chế | Thành phần | Mục đích |
| --- | --- | --- | --- |
| Backend alive | `GET /health` hoặc `/actuator/health` | Backend | Process sống |
| Database | DB query/connection test | Backend | Kết nối database |
| Redis | Redis PING | Backend | Kết nối Redis |
| Storage | Presigned URL test hoặc bucket HEAD | Backend | Object storage accessible |
| AI service | Backend → `GET /api/health` | Backend + AI | AI model loaded + ready |
| Mail | Config validation / test send (dev/staging) | Backend | Mail provider hợp lệ |

### 12.2 Readiness policy

- Trạng thái **ready** của Backend API được quyết định duy nhất bởi **Hard Dependency**:
  - Database (Bắt buộc)
- Các dịch vụ phụ trợ là **Soft Dependencies**. Lỗi kết nối đến các dịch vụ này sẽ đánh dấu trạng thái *Degraded/Warning* trên health indicator (`/health`) để giám sát, nhưng KHÔNG làm sập readiness:
  - Redis (Lỗi → degrade gracefully, fallback xuống DB)
  - Object storage (Lỗi → trả về mã lỗi nghiệp vụ khi upload/download)
  - AI service (Lỗi → trả về mã lỗi nghiệp vụ thân thiện + cho phép retry)
  - Mail provider (Lỗi → log cảnh báo, retry gửi mail sau)

### 12.3 Liveness vs Readiness

| Probe | Ý nghĩa | Dependency check |
| --- | --- | --- |
| Liveness | Backend process sống, không bị hang | Không check dependency |
| Readiness | Backend sẵn sàng nhận request | Chỉ check Database (Hard dependency) |

---

## 13. Observability

### 13.1 Logging

| Nhóm | Nội dung cần log |
| --- | --- |
| **API request** | `requestId`, method, path, status, latency, userId (nếu phù hợp) |
| **Auth** | Login fail/success, OTP fail/expire, refresh fail, account ban/unban, registration |
| **Recognition** | `requestId`, image metadata (size/type), AI modelVersion, processingTimeMs, object count, errors |
| **Storage** | upload-init, upload-complete, validation fail (MIME/size), orphan cleanup, presigned URL generation |
| **Dictionary** | Lookup latency, not-found rate, import job result (count, errors, duration) |
| **Learning** | Save word (source), flashcard recall (rating), quiz attempt (score), SRS review (state change) |
| **Gamification** | XP/coin transaction, mission complete, badge award, duplicate event ignored (idempotent) |
| **Leaderboard** | Cache refresh job, ranking update, Redis error/failover |
| **Notification** | Push sent/fail, device token register/expire, in-app notification created |
| **System** | DB/Redis/Storage/AI availability change, exception rate, unhandled errors |

**Quy tắc logging:**
- Structured format (JSON recommended) cho production.
- Mỗi request có `requestId` xuyên suốt.
- **KHÔNG** log: password, JWT token, OTP, secret key, presigned URL.
- Log level: DEBUG (dev), INFO (staging), WARN+ERROR (production default).

### 13.2 Metrics đề xuất

| Metric | Nguồn | Ý nghĩa |
| --- | --- | --- |
| API latency p50/p95/p99 | Backend | Hiệu năng API |
| API error rate | Backend | Tỷ lệ 4xx/5xx |
| AI recognition latency | AI Service | Thời gian inference |
| AI timeout rate | Backend | Tần suất AI timeout |
| Dictionary cache hit rate | Redis | Hiệu quả cache |
| Leaderboard cache freshness | Redis | Tuổi cache leaderboard |
| Upload success/fail rate | Storage | Tình trạng upload |
| DB connection pool usage | Backend | Sức khỏe connection pool |
| DB slow queries | Database | Performance issues |
| Redis memory/connections | Redis | Sức khỏe Redis |

### 13.3 Alert gợi ý

| Alert | Điều kiện | Severity |
| --- | --- | --- |
| Backend error rate cao | 5xx rate > 5% trong 5 phút | Critical |
| AI service unavailable | Health check fail > 3 lần liên tiếp | Critical |
| AI timeout rate cao | Timeout rate > 20% trong 10 phút | Warning |
| Database connection failure | Connection pool exhausted hoặc DB unreachable | Critical |
| Redis unavailable | PING fail > 3 lần liên tiếp | Warning |
| Storage upload failure | Upload-complete fail rate > 10% | Warning |
| Mail provider failure | OTP gửi lỗi liên tục > 5 lần | Warning |
| Dictionary import failure | Import job fail hoặc import count = 0 | Warning |
| Reward duplicate detected | Idempotent event key blocked > threshold | Info |

---

## 14. Backup, Restore & Cleanup

### 14.1 Backup

| Dữ liệu | Chiến lược |
| --- | --- |
| Database | Backup định kỳ (daily cho production); kiểm thử restore trước demo |
| Object storage | Versioning/lifecycle nếu R2 hỗ trợ; metadata DB là nguồn tham chiếu |
| Redis | **Không** bắt buộc backup (cache); leaderboard rebuild từ DB/events |
| Dictionary source | Lưu import script/source file để tái import khi cần |
| Configuration | VCS (không secrets); secrets trong secret manager |

### 14.2 Restore

| Bước | Hành động |
| --- | --- |
| 1 | Restore database trước |
| 2 | Kiểm tra media object còn tồn tại theo `storage_metadata.object_key` |
| 3 | Rebuild progress/leaderboard aggregates nếu cần |
| 4 | Rebuild Redis cache từ database/events |
| 5 | Verify health checks pass |
| 6 | Run smoke tests |

### 14.3 Cleanup jobs

| Job | Mô tả | Frequency |
| --- | --- | --- |
| Orphan media | Xóa object không còn tham chiếu trong DB | Daily |
| Upload session expired | Xóa upload session hết hạn (chưa complete) | Hourly |
| Old avatar | Xóa avatar cũ sau khi user đổi avatar | Event-driven |
| Expired OTP | Cleanup OTP tokens hết hạn | Hourly |
| Expired refresh token | Cleanup refresh tokens đã expire/revoke | Daily |
| Old notifications | Xóa notification cũ theo retention policy (nếu có) | Weekly |
| Logs | Theo retention policy của môi trường | Configurable |

---

## 15. CI/CD Pipeline

### 15.1 Backend API

```text
commit / PR
  → Lint + static analysis (optional)
  → mvn test (unit + integration)
  → mvn package (build JAR artifact)
  → Deploy staging
  → Smoke test: health → auth → word search → storage upload → recognition (nếu AI bật)
  → Manual QA / UAT
  → Deploy production (khi được duyệt)
```

### 15.2 AI Service

```text
commit / PR
  → Install Python dependencies
  → Pytest: model load smoke → inference sample image → error handling
  → Package container (Docker)
  → Deploy staging
  → Backend integration test (POST /api/v1/detect với ảnh test)
  → Deploy production (khi được duyệt)
```

### 15.3 Mobile App

```text
commit / PR
  → TypeScript typecheck
  → ESLint
  → Jest tests (unit + component)
  → Expo build preview/dev client
  → Test flows: auth → search → camera → flashcard → quiz
  → Build release (khi milestone ổn định)
```

### 15.4 Database

```text
Schema change
  → Migration script (SQL / Flyway / Liquibase)
  → Apply staging
  → Verify data integrity
  → Apply production
```

---

## 16. Deployment Checklist

### 16.1 Backend API

- [ ] Build artifact (JAR) thành công
- [ ] Environment variables đầy đủ (App, DB, Redis, JWT, Mail, Storage, AI, Upload, CORS, OpenAPI)
- [ ] Kết nối Database thành công
- [ ] Kết nối Redis thành công
- [ ] Kết nối Object Storage thành công (presigned URL test)
- [ ] Kết nối Mail Provider thành công (test send nếu dev/staging)
- [ ] Kết nối AI Service thành công (health check)
- [ ] Swagger/OpenAPI hoạt động (dev/staging)
- [ ] Health check endpoint trả đúng trạng thái
- [ ] HTTPS configured (staging/production)
- [ ] CORS configured đúng cho mobile origin
- [ ] Dictionary data đã import

### 16.2 AI Service

- [ ] Model weights (Florence-2; CLIP nếu bật từ vựng nền) loaded thành công — `GET /health` trả `status = ok`
- [ ] Health endpoint (`GET /api/health`) trả modelLoaded=true
- [ ] Inference endpoint hoạt động với ảnh test (trả objects)
- [ ] Timeout và max image size được cấu hình
- [ ] Log requestId/processingTimeMs hoạt động
- [ ] Backend gọi được AI service (integration test)
- [ ] GPU available (production)
- [ ] Service token configured (nếu public network)

### 16.3 Database

- [ ] Schema/migration đã chạy
- [ ] Dictionary data đã import (hoặc seed subset cho test)
- [ ] Index quan trọng đã tạo
- [ ] Backup policy production đã cấu hình
- [ ] Connection pool size phù hợp

### 16.4 Object Storage

- [ ] Bucket tồn tại
- [ ] Access key/secret đúng môi trường
- [ ] Private access policy đúng
- [ ] Presigned upload hoạt động (test upload)
- [ ] Upload-complete validate được object (MIME/size)
- [ ] Presigned download hoạt động

### 16.5 Redis

- [ ] Redis accessible từ backend
- [ ] Key prefix cấu hình theo môi trường
- [ ] Memory limit phù hợp

### 16.6 Mobile Integration

- [ ] `API_BASE_URL` mobile trỏ đúng backend
- [ ] Auth flow hoạt động (register → OTP → login → refresh → logout)
- [ ] Upload avatar hoạt động
- [ ] Search word hoạt động
- [ ] Camera/detection hoạt động (nếu AI service đã bật)

---

## 17. Smoke Test sau Deploy

### 17.1 Test matrix

| # | Test | Endpoint/Flow | Kết quả mong đợi | Milestone |
| --- | --- | --- | --- | --- |
| 1 | Backend health | `GET /health` | Backend trả OK / UP | M1 |
| 2 | Swagger (dev/staging) | `GET /swagger-ui.html` | Mở được API docs | M1 |
| 3 | Register | `POST /auth/register` | Tạo user PENDING, OTP gửi | M1 |
| 4 | OTP verify | `POST /auth/verify-otp` | Account → ACTIVE | M1 |
| 5 | Login | `POST /auth/login` | Trả accessToken + refreshToken | M1 |
| 6 | Refresh token | `POST /auth/refresh` | Trả new accessToken | M1 |
| 7 | Profile | `GET /users/me` | Trả user info | M1 |
| 8 | Word search | `GET /words?q=apple` | Trả word detail + translation | M1 |
| 9 | Avatar upload | Storage upload flow | Presigned upload + complete thành công | M1 |
| 10 | AI recognition | `POST /api/scan` → poll `GET /api/scan/{requestId}` | Ảnh test về `DONE`, items có label/reliability/box | M2 |
| 11 | Save word | `POST /topics/{topicId}/items` | Lưu TopicItem thành công | M1-M2 |
| 12 | Flashcard | Flashcard API | Render và học flashcard theo Topic và Template | M1 |
| 13 | Quiz | Quiz API | Tạo và submit quiz test | M3 |
| 14 | SRS review | Review API | Lấy review queue hoặc empty state | M3 |
| 15 | Progress | `GET /progress/summary` | Trả summary hợp lệ | M3 |
| 16 | Notification | Notification API | List/read notification hoạt động | M3 |
| 17 | Leaderboard | Leaderboard API | Ranking trả dữ liệu/cache hợp lệ | M4 |
| 18 | Mission | Mission API | Missions list hoạt động | M4 |
| 19 | Shop | Shop API | ShopItem list hoạt động | M4 |
| 20 | Admin | `GET /admin/users` (ROLE_ADMIN) | Trả user list (admin only) | M4 |

### 17.2 E2E flows cần kiểm tra

| Flow | Mô tả | Milestones |
| --- | --- | --- |
| Auth lifecycle | Register → OTP → Login → Refresh → Logout | M1 |
| Scan-to-learn | Camera → Upload → AI detect → Map word → Save TopicItem | M2 |
| Dictionary lookup | Search → Word detail → Save to personal Topic | M1 |
| Topic browse | Collection → Topic → TopicItem → Save to personal Topic | M1 |
| Flashcard study | Open Topic → Study session → Template render → FSRS rating | M1, M3 |
| Quiz | Setup → Play → Submit → Result → Progress update | M3 |
| SRS review | Due queue → Review → Rating → FsrsRecord update | M3 |
| Gamification | Learning activity → XP/Coin → Mission → Badge → Leaderboard | M4 |
| Storage | Avatar upload → Edit profile → Presigned access | M1 |

---

## 18. Rủi ro vận hành

| # | Rủi ro | Ảnh hưởng | Giảm thiểu |
| --- | --- | --- | --- |
| 1 | AI Florence pipeline dùng GPU, ~15–30s/ảnh | Recognition chậm, UX kém | Hàng đợi AI giới hạn 1 worker/GPU mặc định, quota 20 scan/ngày/Learner, timeout worker→AI 60s (cấu hình), loading/queued UX, cancel button |
| 2 | Import dictionary lớn (357K+ từ) | DB chậm hoặc lỗi encoding | Import batch, index sau import, test subset trước |
| 3 | Redis mất dữ liệu (restart) | Leaderboard/cache mất tạm thời | Redis chỉ cache, rebuild từ DB/events |
| 4 | Object storage config sai giữa dev/prod | Upload avatar/scan thất bại | Smoke test presigned upload per env, S3-compatible abstraction |
| 5 | Mail provider lỗi | Không xác thực/reset password được | Retry logic, provider health check, error message thân thiện |
| 6 | Secret bị commit vào VCS | Rủi ro bảo mật nghiêm trọng | Dùng env/secret manager, git-secrets scan, pre-commit hooks |
| 7 | Swagger public trên production | Lộ API surface cho attacker | Tắt hoặc restrict Swagger production (IP/auth) |
| 8 | Reward cộng trùng do retry | Sai coin/XP/leaderboard | Idempotent event key + unique transaction |
| 9 | AI label không khớp dictionary | Không tạo được TopicItem từ scan | ObjectWordMapping + synonym table, dictionary miss state UI |
| 10 | Database connection pool exhausted | API timeout hàng loạt | Monitor pool size, optimize slow queries, connection timeout |
| 11 | R2/MinIO CORS misconfigured | Mobile upload fail | Bucket CORS policy cho presigned URL origin |
| 12 | JWT secret weak/leaked | Token giả mạo | Secret đủ mạnh, rotate được, monitor unusual auth patterns |

---

## 19. Server theo Milestone

| Milestone | Thành phần server cần sẵn sàng |
| --- | --- |
| **M1 — Core Auth & Vocabulary** | Backend API (auth, user, word, topic, storage, vocabulary, flashcard basic) · MySQL + dictionary import · Redis (cache) · MinIO/S3 (avatar) · Mail provider · Swagger |
| **M2 — Camera/Recognition** | AI Service (Florence-2 trên GPU) · Recognition API · Storage (scan image) · ObjectWordMapping |
| **M3 — Learning Engine** | Quiz API · SRS engine (FSRS) · Progress aggregate · Notification service (Push + In-app) · Template CRUD |
| **M4 — Gamification & Production** | Gamification/Shop/Leaderboard APIs · Redis sorted set · Admin CMS APIs · Cloudflare R2 production · Hardening (security, observability, CI/CD) |

---

## 20. Tài liệu liên quan

| Tài liệu | Đường dẫn | Vai trò |
| --- | --- | --- |
| Đặc tả yêu cầu (SRS) | [specs.md](../spec/specs.md) | Source of truth |
| System Architecture | [sa.md](./sa.md) | Kiến trúc tổng thể |
| Tech Stack | [techstack.md](./techstack.md) | Công nghệ theo lớp |
| Luồng nghiệp vụ | [buss_mainflow.md](../spec/buss_mainflow.md) | BF — actor, bước, ngoại lệ |
| Phân rã phân hệ | [phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md) | SS — ranh giới module, entities, API |
| Phân rã tính năng | [phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md) | F — sub-feature + AC theo FR |
| Phân rã màn hình | [phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md) | MH — UI map, navigation, states |

---

## 21. Checklist nghiệm thu server/deployment

- [x] Backend API public qua HTTPS; DB/Redis/AI service/Object Storage **không** public trực tiếp.
- [x] Mỗi môi trường (dev/staging/prod) có cấu hình riêng, không hardcode secret.
- [x] Backend kết nối được database, Redis, object storage, mail provider và AI service.
- [x] AI service: Florence-2 pipeline ready, timeout/max image size cấu hình, health check.
- [x] Object storage: bucket private, upload qua presigned URL, MIME/size validation, orphan cleanup.
- [x] Mail provider: OTP gửi được, TTL/attempts/cooldown đúng rule.
- [x] JWT/refresh token: sign/verify/revoke hoạt động đúng.
- [x] Swagger/OpenAPI hoạt động ở dev/staging, restricted ở production.
- [x] Health/readiness checks cho tất cả dependency quan trọng.
- [x] Log/metric cho auth, recognition, storage, dictionary, learning, gamification, notification, system.
- [x] Backup database + restore test trước demo.
- [x] Cleanup jobs: orphan media, expired uploads, expired OTP/tokens.
- [x] Smoke test sau deploy bao phủ: auth lifecycle, word search, avatar upload, recognition, save vocabulary, flashcard/quiz/SRS.
- [x] Security: password hash, OTP safety, CORS, upload validation, presigned URL TTL, idempotent reward, no secret logging.
- [x] Dictionary import (357K+ từ) hoàn thành và index tối ưu.
- [x] Admin CMS APIs yêu cầu `ROLE_ADMIN`, tách deploy/route với mobile.
- [x] CI/CD pipeline cho backend, AI service, mobile app được định nghĩa.
- [x] Rủi ro vận hành đã nhận diện và có phương án giảm thiểu.
- [x] Tài liệu không chứa secrets, passwords, tokens hoặc endpoint nhạy cảm thật.
