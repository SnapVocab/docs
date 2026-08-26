# Tech Stack — SnapVocab

> Tài liệu tổng hợp công nghệ sử dụng trong SnapVocab, được xây dựng dựa trên [specs.md](../spec/specs.md), [sa.md](./sa.md), [server.md](./server.md), [phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md), [phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md) và [phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md).

>
> **Canonical sync (2026-08-23):** Source = [`../spec/specs.md`](../spec/specs.md). AI = Florence-2+SAM+CLIP (F2-v13) zero-shot · Learning = Deck/Note/Card · SRS = FSRS · Actors = Guest/Learner/Admin · FR: Game=09, Noti=10, Storage=11, OpenAPI=12, Admin=13 · 4 milestones.

---

## 1. Tổng quan stack

SnapVocab sử dụng kiến trúc mobile-first cho Learner, web admin riêng cho CMS/quản trị, backend API tách biệt, AI service riêng cho nhận diện hình ảnh, database quan hệ làm source of truth, Redis cho cache/ranking và object storage cho media binary.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TECH STACK OVERVIEW                            │
│                                                                             │
│  ┌─────────────────┐     ┌───────────────────────────────────────────┐     │
│  │ React Native    │     │ Java 17 + Spring Boot                     │     │
│  │ Expo            │────▶│ Spring Security + JWT                     │     │
│  │ TypeScript      │ API │ JPA/Hibernate + MapStruct                 │     │
│  │ Expo Router     │     │ Springdoc OpenAPI                         │     │
│  └─────────────────┘     └──┬──────┬──────┬──────┬──────────────────┘     │
│  ┌─────────────────┐        │      │      │      │                        │
│  │ Next.js Admin   │────────┘      │      │      │                        │
│  │ React + TS      │  API          │      │      │                        │
│  │ shadcn/ui       │              │      │      │                        │
│  └─────────────────┘              │      │      │                        │
│          ┌───────────────────┘      │      │      └──────────────┐        │
│          ▼                          ▼      ▼                     ▼        │
│  ┌──────────────┐  ┌──────────────┐ ┌──────────────┐ ┌─────────────────┐ │
│  │ MySQL/MariaDB│  │ Redis        │ │ Cloudflare R2│ │ Python FastAPI  │ │
│  │ JPA/Hibernate│  │ Redisson     │ │ MinIO (dev)  │ │ Florence-2      │ │
│  │              │  │              │ │ S3-compatible│ │ SAM + CLIP      │ │
│  │ Source of    │  │ Cache,       │ │ Private      │ │ GPU T4+         │ │
│  │  Truth       │  │ Leaderboard  │ │  Bucket      │ │ Zero-shot       │ │
│  └──────────────┘  └──────────────┘ └──────────────┘ └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

| Lớp | Công nghệ chính | Vai trò |
| --- | --- | --- |
| Mobile App | React Native, Expo, TypeScript, Expo Router | Ứng dụng iOS/Android: camera, learning, gamification, profile |
| Web Admin CMS | Next.js, React, TypeScript, Tailwind CSS, shadcn/ui | Web quản trị cho Admin: dashboard, user management, dictionary/topic/content, gamification, moderation |
| Backend API | Java 17, Spring Boot, Spring Security, JPA/Hibernate | REST API nghiệp vụ: auth, dictionary, recognition, learning, gamification, admin CMS |
| Security | Spring Security, JWT (access + refresh), OTP/email, SecureStore/biometric, WebAuthn optional | Xác thực, phân quyền, bảo vệ API, mở khóa cục bộ an toàn |
| Database | MySQL/MariaDB, JPA/Hibernate | Source of truth cho toàn bộ dữ liệu nghiệp vụ |
| Cache | Redis, Redisson | Cache dictionary, leaderboard sorted set, rate limiting |
| AI Service | Python, FastAPI, Florence-2 + SAM + CLIP | Pipeline nhận diện vật thể zero-shot từ ảnh |
| Object Storage | Cloudflare R2 (prod), MinIO (dev), S3-compatible API | Avatar, ảnh scan, ảnh crop SAM, tài nguyên vật phẩm |
| Dictionary Data | SQLite (minhqnd/dictionary) → import vào MySQL | Nguồn dữ liệu từ vựng Anh-Việt (357K+ từ, 443K+ định nghĩa) |
| API Docs | Swagger/OpenAPI (Springdoc) | Tài liệu hóa và kiểm thử backend API |
| Design | Figma, Design System | Thiết kế UI/UX, chuẩn hóa component |
| Build/Ops | Maven, Expo/EAS, Docker (tùy triển khai) | Build, test, deploy, vận hành |

---

## 2. Mobile App Stack

### 2.1 Công nghệ

| Công nghệ | Vai trò | Ghi chú |
| --- | --- | --- |
| **React Native** | Framework UI mobile đa nền tảng | iOS + Android từ cùng codebase |
| **Expo** | Tooling, runtime, native capabilities | Dev nhanh, camera/gallery, build pipeline, OTA update |
| **TypeScript** | Type safety | Kiểm soát kiểu cho API response phức tạp, giảm runtime error |
| **Expo Router** | File-based routing/navigation | Phù hợp app nhiều tab/nested: auth, learn, camera, profile |
| **Axios** (hoặc tương đương) | HTTP client | Gắn JWT interceptor, xử lý refresh token, error handling |
| **TanStack Query** (React Query) | Server state/cache | Cache API data: words, progress, leaderboard; background refetch |
| **Zustand** / **MMKV** | Client state/persistence | UI state, preferences, queue rating offline & sync batch; secret ưu tiên SecureStore |
| **Zod** | Form/input validation | Auth forms, profile, quiz, search validation |
| **Expo SecureStore** | Secure token storage | Lưu refresh token/secret nhỏ trong Keychain/Keystore thay vì plain storage |
| **Expo Local Authentication** | Biometric unlock | Mở khóa nhanh bằng vân tay/Face ID sau khi đã login; không gửi biometric data lên backend |
| **React Native MMKV** | Local persistence hiệu năng cao | Preferences, cache nhẹ; không dùng cho secret nếu chưa mã hóa/thiết kế rõ |
| **Expo Image Picker** | Chọn ảnh từ thư viện | Gallery picker cho recognition flow |
| **Expo Camera** | Camera capture | Chụp ảnh trực tiếp cho scan-to-learn |
| **expo-speech** | TTS on-device | Phát âm từ vựng khi `Pronunciation.audioUrl = null` — fallback không cần server; xem ARC-12 |
| **expo-speech-recognition** | STT on-device (Voice Search) | Capture giọng nói tiếng Việt → text → gửi lên `/words/search` reverse-lookup bảng Translation; xem ARC-12 |
| **i18n** (optional) | Đa ngôn ngữ UI | UI tiếng Việt chính, có thể mở rộng |

### 2.2 Cấu trúc module mobile

| Module | Màn hình chính (MH) | API backend tương ứng | Milestone |
| --- | --- | --- | --- |
| **Auth** | Onboarding, Login, Signup, OTP Verify, Forgot/Reset Password | Auth API (SS-03) | M1 |
| **Home** | Home Dashboard, Learn Hub | Progress, SRS, Gamification API | M1 |
| **Recognition** | Camera Scan, Detection Result | Recognition, Storage, Word API (SS-06) | M2 |
| **Dictionary** | Search, Word Detail, Voice Search | Dictionary API (SS-04) | M1 |
| **Topic** | Collections, Topic Items | Topic API (SS-05) | M1 |
| **Vocabulary** | My Vocabulary (Deck List), Deck Detail | Vocabulary API (SS-08) | M1–M2 |
| **Learning** | Flashcards, Quiz Setup/Play/Result, SRS Review, Template Management | Flashcard, Quiz, SRS API (SS-09/10/11) | M1, M3 |
| **Progress** | Stats, Level Progress | Progress API (SS-12) | M3 |
| **Gamification** | Missions, Achievements, Leaderboard | Gamification API (SS-13) | M4 |
| **Economy** | Wallet, Shop, Inventory | Shop/Wallet API (SS-14) | M4 |
| **Profile** | Profile, Edit Profile, Settings, Notifications | User, Storage, Notification API (SS-03/15/16) | M1, M3 |
| **System UI** | Design System, Empty/Error/Loading States | Cross-cutting, không phụ thuộc API cụ thể | Ongoing |

### 2.3 Navigation architecture

```text
App Root
├── (auth)                          ← Guest-only screens
│   ├── onboarding
│   ├── login
│   ├── signup
│   ├── otp-verify
│   ├── forgot-password
│   └── reset-password
│
└── (tabs)                          ← Learner screens (JWT required)
    ├── home                        ← Home Dashboard
    │   ├── srs-review
    │   ├── missions
    │   └── stats
    ├── learn                       ← Learn Hub
    │   ├── decks/[deckId]          ← Deck Detail
    │   ├── flashcards/[deckId]     ← Flashcard Session
    │   ├── quiz/setup
    │   ├── quiz/play
    │   ├── quiz/result
    │   ├── srs-review
    │   └── templates
    ├── camera                      ← Camera Scan
    │   └── detection-result
    ├── search                      ← Dictionary Search
    │   ├── word/[wordId]           ← Word Detail
    │   ├── voice-search
    │   ├── collections
    │   └── topics/[topicId]
    └── profile                     ← Profile
        ├── edit
        ├── settings
        ├── notifications
        ├── achievements
        ├── wallet
        ├── shop
        └── inventory
```

### 2.4 API client pattern

```text
API Client Layer:
  ├── axiosInstance                  ← Base URL, timeout, request/response interceptors
  │   ├── Request interceptor       ← Gắn JWT access token vào Authorization header
  │   └── Response interceptor      ← Catch 401 → auto refresh token → retry original request
  ├── TanStack Query hooks          ← useQuery / useMutation per API group
  │   ├── useWords()                ← Dictionary search/detail
  │   ├── useDecks()                ← Vocabulary CRUD
  │   ├── useCards()                ← Flashcard/SRS
  │   ├── useQuiz()                 ← Quiz lifecycle
  │   ├── useProgress()             ← Stats/summary
  │   ├── useRecognition()          ← Scan flow
  │   └── ...                       ← Gamification, notification, storage
  └── Error handling                ← Map error.code → user-friendly message + CTA
```

### 2.5 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| React Native / Expo | Đa nền tảng iOS/Android; phù hợp thời gian đồ án; Expo hỗ trợ camera/image picker nhanh |
| TypeScript | Type safety cho API response phức tạp (Word, Note, Card, Quiz); giảm lỗi runtime |
| Expo Router | File-based routing tự nhiên cho app nhiều màn hình (30 MH); deep linking dễ |
| TanStack Query | Phù hợp dữ liệu remote: words, progress, leaderboard — cache, background refetch, pagination |
| Zustand + MMKV | Client state nhẹ; MMKV nhanh hơn AsyncStorage cho preferences/cache; secret ưu tiên SecureStore |
| Expo SecureStore + Local Authentication | Cho phép biometric unlock thuận tiện mà không biến vân tay/Face ID thành credential backend |
| expo-speech (TTS) | Dataset minhqnd/dictionary hầu như không có file audio; `audioUrl` nullable; TTS on-device không phát sinh thêm cost/API/server; xem ARC-12 |
| expo-speech-recognition (STT) | Voice Search on-device: không cần STT API cloud, không phát sinh credential/billing; text kết quả gửi thẳng lên `/words/search`; xem ARC-12 |

---

## 3. Web Admin CMS Stack

Web Admin CMS là frontend riêng dành cho `ROLE_ADMIN`, chạy trên browser và gọi cùng Backend API qua các endpoint admin/domain đã phân quyền. Web admin không thay thế mobile app; mục tiêu là quản trị dữ liệu, theo dõi vận hành và thao tác nội dung nhanh hơn so với làm trực tiếp trên database.

### 3.1 Công nghệ

| Công nghệ / thư viện | Vai trò | Ghi chú |
| --- | --- | --- |
| **Next.js** | Framework web React | App Router, routing theo module admin, có thể deploy dạng Node hoặc static tuỳ mức dùng server features |
| **React** | UI library | Component-based, dễ chia màn hình dashboard/table/form |
| **TypeScript** | Type safety | Đồng bộ type với OpenAPI/backend DTO, giảm lỗi khi gọi API admin |
| **Tailwind CSS** | Styling utility-first | Tạo UI nhất quán nhanh, phù hợp dashboard/CMS |
| **shadcn/ui** | Bộ component mã nguồn mở | Table, dialog, form, select, tabs, toast; dựa trên Radix UI + Tailwind, dễ chỉnh theo design system |
| **Radix UI** | Accessible primitives | Dialog, dropdown, popover, select có accessibility tốt |
| **TanStack Query** | Server state/cache | Cache danh sách user/word/topic/stats, pagination, invalidation sau mutation |
| **TanStack Table** | Data table | Sorting, filtering, pagination, column visibility cho CMS table lớn |
| **React Hook Form + Zod** | Form + validation | Form tạo/sửa word, topic, mission, shop item; schema validation thống nhất |
| **Axios** (hoặc fetch wrapper) | HTTP client | Gắn JWT interceptor, refresh token, map lỗi backend |
| **Recharts** (optional) | Biểu đồ dashboard | Thống kê user, scan, learning events, error rate ở mức admin |
| **Lucide React** | Icon set | Icon nhất quán với shadcn/ui |

### 3.2 Module web admin

| Module | Màn hình / chức năng | API backend tương ứng | Milestone |
| --- | --- | --- | --- |
| **Admin Auth** | Login admin, logout, refresh session, đổi mật khẩu | Auth API + Admin role guard | M4 |
| **Dashboard** | Tổng quan user, học tập, scan, quiz, hệ thống | Admin stats API, Progress/Gamification aggregate | M4 |
| **User Management** | Danh sách user, xem hồ sơ, khóa/mở khóa, reset trạng thái cần thiết | Admin User API | M4 |
| **Dictionary CMS** | Tìm kiếm, xem/sửa word, definition, translation, pronunciation | Dictionary/Admin API | M4 |
| **Topic CMS** | Quản lý collection/topic/topic item/attributes | Topic/Admin API | M4 |
| **Object Mapping** | Mapping AI label/synonym → Word, xử lý dictionary miss | Recognition/Dictionary Admin API | M4 (Ngoài MVP) |
| **Learning Content** | Card template, quiz template/rule cơ bản | Flashcard/Quiz Admin API | M4 (Ngoài MVP) |
| **Gamification CMS** | Mission, badge, shop item, item asset | Gamification/Economy/Storage Admin API | M4 |
| **Notification CMS** | Gửi/thử nghiệm thông báo, xem log gửi | Notification Admin API | M4 (Ngoài MVP) |
| **Storage Management** | Xem metadata media, cleanup orphan, kiểm tra upload lỗi | Storage Admin API | M4 (Ngoài MVP) |
| **Audit/Logs** | Tra cứu hành động admin và lỗi hệ thống cơ bản | Admin Audit/Observability API | M4 (Ngoài MVP) |

### 3.3 Routing architecture

```text
Admin App Root
├── (public)
│   └── login
│
└── (admin)                         ← ROLE_ADMIN required
    ├── dashboard
    ├── users
    │   └── [userId]
    ├── dictionary
    │   └── [wordId]
    ├── topics
    │   ├── collections
    │   └── items/[topicItemId]
    ├── recognition
    │   └── object-mappings (ngoài MVP)
    ├── learning
    │   ├── card-templates
    │   └── quiz-rules (ngoài MVP)
    ├── gamification
    │   ├── missions
    │   ├── badges
    │   └── shop-items
    ├── notifications (ngoài MVP)
    ├── storage (ngoài MVP)
    └── audit-logs (ngoài MVP)
```

### 3.4 Admin API client pattern

```text
Admin API Client Layer:
  ├── httpClient                         ← Base URL, timeout, Authorization header
  │   ├── Request interceptor            ← Gắn admin JWT access token
  │   └── Response interceptor           ← 401 refresh/logout; 403 hiển thị forbidden state
  ├── TanStack Query hooks
  │   ├── useAdminUsers()                ← User list/detail/actions
  │   ├── useAdminWords()                ← Dictionary CMS
  │   ├── useAdminTopics()               ← Topic CMS
  │   ├── useAdminMissions()             ← Gamification CMS
  │   └── useAdminDashboardStats()       ← Dashboard aggregates
  └── Form schemas                       ← React Hook Form + Zod per create/update DTO
```

### 3.5 Quy tắc bảo mật web admin

1. Tất cả route admin phải yêu cầu `ROLE_ADMIN`; frontend guard chỉ để UX, backend guard là bắt buộc.
2. Không lưu JWT trong `localStorage` nếu có thể tránh; ưu tiên httpOnly cookie hoặc access token ngắn hạn trong memory + refresh token được bảo vệ.
3. Form CMS phải validate ở cả frontend (Zod) và backend (`@Valid`).
4. Dashboard/log không hiển thị secret, token, OTP hoặc thông tin nhạy cảm.
6. Production admin nên giới hạn origin/domain, bật HTTPS, CSRF protection nếu dùng cookie.

### 3.6 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| Next.js + React + TypeScript | Phổ biến, dễ phát triển CMS/dashboard, type-safe khi tích hợp OpenAPI |
| Tailwind CSS + shadcn/ui | Nhanh có UI admin đẹp, accessible, dễ tùy biến, không bị khóa vào component library đóng |
| TanStack Query + Table | Phù hợp màn hình quản trị nhiều dữ liệu, pagination/filter/sort/invalidation rõ ràng |
| React Hook Form + Zod | Form CMS nhiều field nhưng vẫn nhẹ, validate rõ, dễ map với backend DTO |
| Tách web admin khỏi mobile app | Admin có UX/table/form/security khác Learner; tránh làm nặng mobile bundle |

---

## 4. Backend API Stack

### 4.1 Công nghệ

| Công nghệ | Vai trò | Ghi chú |
| --- | --- | --- |
| **Java 17** | Runtime | Ổn định, LTS, phù hợp Spring Boot hiện đại |
| **Spring Boot** | Framework API | REST API, dependency injection, validation, config, auto-configuration |
| **Spring Web MVC** | HTTP layer | Mobile/CMS gọi backend qua JSON REST |
| **Spring Security** | Auth/security | JWT filter chain, role/authority, request protection |
| **Spring Data JPA / Hibernate** | ORM/persistence | Entity mapping, repository pattern, transaction management |
| **Spring Validation** | Request validation | `@Valid` + jakarta.validation trên request DTOs |
| **Spring Mail** | Email/OTP | Signup verification, forgot password |
| **Springdoc OpenAPI** | API docs | Swagger UI + OpenAPI 3.0 spec tự động từ controller/DTO |
| **Lombok** | Giảm boilerplate | Entity/DTO/Service: getter/setter/builder/constructor |
| **MapStruct** | DTO ↔ Entity mapping | Tách domain khỏi response model, compile-time mapping |
| **Maven** | Build/dependency | Build JAR artifact, dependency management |

### 4.2 Module backend (mapping SS → package)

| Module | Package | SS | Công nghệ/Service chính | Milestone |
| --- | --- | --- | --- | --- |
| **Identity** | `com.snapvocab.identity` | SS-03 | Spring Security, JWT, Mail/OTP | M1 |
| **Dictionary** | `com.snapvocab.dictionary` | SS-04 | JPA repositories, indexed queries, Redis cache | M1 |
| **Topic** | `com.snapvocab.topic` | SS-05 | JPA, EAV model (Collection/Topic/TopicItem) | M1 |
| **Recognition** | `com.snapvocab.recognition` | SS-06 | Spring @Async queue, HTTP client → FastAPI, RecognitionFilter, LabelDedup | M2 |
| **Vocabulary** | `com.snapvocab.vocabulary` | SS-08 | JPA transaction, Deck/Note CRUD | M1–M2 |
| **Flashcard** | `com.snapvocab.flashcard` | SS-09 | CardService, StudySessionService, FsrsService, CardTemplateService | M1, M3 |
| **Quiz** | `com.snapvocab.quiz` | SS-10 | Quiz generation, scoring, idempotent submit | M3 |
| **SRS** | `com.snapvocab.srs` | SS-11 | ReviewQueueService (dùng Card/ReviewLog từ flashcard) | M3 |
| **Progress** | `com.snapvocab.progress` | SS-12 | Aggregate service, LearningEvent processing | M3 |
| **Gamification** | `com.snapvocab.gamification` | SS-13+14 | XP/Coin/Mission/Badge/Leaderboard/ShopService | M4 |
| **Notification** | `com.snapvocab.notification` | SS-15 | PushService (Expo/FCM), InAppService, DeviceTokenService | M3 |
| **Storage** | `com.snapvocab.storage` | SS-16 | S3StorageService, UploadValidation, OrphanCleanup | M1, M2, M4 |
| **Admin** | `com.snapvocab.admin` | SS-17 | Admin controllers delegating to domain services | M4 |
| **Common** | `com.snapvocab.common` | — | ApiResponse envelope, GlobalExceptionHandler, JwtUtils, domain events | M1 (ongoing) |

### 4.3 Kiến trúc layer

```text
Controller Layer
  ├── @RestController                   ← Nhận HTTP request
  ├── Request DTO + @Valid              ← Validate input
  ├── @PreAuthorize / SecurityContext   ← Kiểm tra quyền
  └── Return ResponseEntity<ApiResponse<T>>

Service Layer
  ├── Business logic                    ← Orchestration, validation, rules
  ├── Transaction boundary (@Transactional)
  ├── Domain events (publish NoteCreated, ReviewCompleted, QuizSubmitted...)
  └── Integration calls (AI client, S3 client, mail client, Redis client)

Repository Layer
  ├── JpaRepository<Entity, ID>         ← CRUD + custom queries
  ├── @Query (JPQL / native SQL)        ← Complex queries
  └── Specification / QueryDSL (optional)

DTO / Mapper Layer
  ├── Request DTOs                      ← Input validation
  ├── Response DTOs                     ← Output shaping
  └── MapStruct @Mapper                 ← Entity ↔ DTO compile-time mapping

Security Layer
  ├── JwtAuthenticationFilter           ← Extract + validate JWT from header
  ├── SecurityConfig                    ← Route protection rules
  ├── CurrentUser                       ← Inject authenticated user context
  └── RoleGuard                         ← ROLE_LEARNER / ROLE_ADMIN checks
```

### 4.4 API response envelope

```json
{
  "success": true,
  "data": { },
  "error": null,
  "requestId": "uuid-v4"
}
```

### 4.5 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| Spring Boot | Phù hợp backend nghiệp vụ nhiều module; ecosystem mạnh cho security, validation, JPA, OpenAPI |
| Spring Security + JWT | Stateless API cho mobile; role-based access; refresh token flow |
| JPA/Hibernate | Mô hình hóa domain phức tạp (Word, Deck/Note/Card, Quiz, Gamification) nhanh; relationship mapping |
| Springdoc OpenAPI | Mobile/backend thống nhất contract qua Swagger UI tự động |
| MapStruct | Compile-time DTO mapping nhanh hơn reflection-based; type-safe |

---

## 5. Security Stack

### 5.1 Tổng quan

| Thành phần | Công nghệ / Cơ chế | Áp dụng | NFR tham chiếu |
| --- | --- | --- | --- |
| Authentication | JWT access token (short-lived) | API cá nhân Learner/Admin | §1 |
| Session continuation | Refresh token (rotate + revoke) | Làm mới access token | §1 |
| Mobile biometric unlock | Expo Local Authentication + SecureStore | Mở khóa app/token cục bộ bằng vân tay/Face ID sau khi đã login | §1 |
| Web admin strong login | WebAuthn/passkey (optional) hoặc OTP/MFA | Tăng bảo vệ tài khoản Admin nếu mở rộng beyond MVP | §1 |
| Password security | bcrypt / argon2 hash | Lưu mật khẩu an toàn | §1 |
| Email verification | OTP qua email (TTL ≤ 10m, max 5, cooldown ≥ 60s) | Signup, forgot password | §3 |
| Authorization | Role-based (ROLE_LEARNER, ROLE_ADMIN) + owner checks | Data isolation | §2 |
| Upload security | MIME allowlist + size validation (avatar ≤ 5MB, scan ≤ 10MB) | Media upload | §5 |
| Storage access | Presigned URL ngắn hạn (TTL ≤ 15m) | Upload/read private media | §6 |
| Service-to-service | Internal network hoặc service token | Backend → AI service | §1 |
| API docs | Swagger env-gated (off/restricted production) | Tránh lộ surface | §1 |
| Idempotency | Event key cho reward/XP/coin/quiz submit | Retry không cộng trùng | §4 |
| Error safety | Generic auth error messages | Chống email enumeration | §10 |

### 5.2 Quy tắc bảo mật

1. **Không** lưu/log password, token, OTP, secret key — ở bất kỳ lớp nào.
2. Mobile **không** gọi trực tiếp database, Redis hoặc AI service.
3. Biometric trên mobile chỉ mở khóa secret/token cục bộ; backend vẫn xác thực bằng JWT/refresh token, không nhận/lưu dữ liệu vân tay/Face ID.
4. Learner chỉ truy cập dữ liệu cá nhân (`owner_id = currentUser.id`).
5. Admin chỉ truy cập qua CMS APIs với `ROLE_ADMIN`.
6. Object key do backend sinh (UUID); **không** dùng tên file user.
7. Presigned URL phải có TTL ngắn và gắn với object key cụ thể.
8. MVP **không** xử lý thanh toán tiền thật; nếu có payment cần stack riêng.
9. Secrets quản lý qua env/secret manager; **không** commit vào VCS.

---

## 6. Database Stack

### 6.1 Công nghệ

| Công nghệ | Vai trò | Ghi chú |
| --- | --- | --- |
| **MySQL / MariaDB** | Database chính | Source of truth cho toàn bộ dữ liệu nghiệp vụ |
| **JPA / Hibernate** | ORM | Truy cập dữ liệu, relationship mapping, lazy/eager loading |
| **SQL index / FULLTEXT** | Tối ưu tra cứu | Dictionary search, SRS queue, leaderboard |
| **Flyway / Liquibase** (nếu bổ sung) | Schema migration | Version control cho schema changes |

### 6.2 Nhóm dữ liệu

| Nhóm | Entity / Bảng tiêu biểu | SS | Milestone |
| --- | --- | --- | --- |
| **Identity** | `users`, `authorities`, `refresh_tokens`, `otp_tokens` | SS-03 | M1 |
| **Dictionary** | `words`, `definitions`, `translations`, `pronunciations`, `word_relations`, `object_word_mappings` | SS-04 | M1 |
| **Topic** | `collections`, `topics`, `topic_items`, `topic_attribute_groups`, `topic_attributes`, `topic_item_attribute_values` | SS-05 | M1 |
| **Recognition** | `image_recognition_requests`, `recognition_results`, `detected_objects`, `scan_histories` | SS-06 | M2 |
| **Vocabulary** | `decks`, `notes`, `note_meanings`, `note_pronunciations` | SS-08 | M1–M2 |
| **Flashcard** | `cards`, `review_logs`, `card_templates`, `card_template_fields` | SS-09 | M1, M3 |
| **Quiz** | `quizzes`, `quiz_questions`, `quiz_attempts` | SS-10 | M3 |
| **Progress** | `learning_events`, `learning_progress` | SS-12 | M3 |
| **Gamification** | `missions`, `mission_progress`, `badges`, `user_badges`, `experience_logs`, `coin_transactions`, `leaderboard_entries` | SS-13 | M4 |
| **Economy** | `shop_items`, `user_items` | SS-14 | M4 |
| **Media** | `storage_metadata` | SS-16 | M1 |
| **Notification** | `notifications`, `device_tokens` | SS-15 | M3 |

### 6.3 Index quan trọng

| Use case | Index | Lý do |
| --- | --- | --- |
| Login | UNIQUE `users.email` | Lookup nhanh + chống trùng |
| Dictionary search | INDEX `words.word` normalized; FULLTEXT nếu phù hợp | Lookup p95 < 500ms |
| Personal vocabulary | UNIQUE `(user_id, word_id, deck_id)` trên `notes` | Chống trùng Note/Deck |
| SRS review queue | INDEX `(user_id, due_at, state)` trên `cards` | Daily review query |
| Quiz history | INDEX `(user_id, created_at)` trên `quiz_attempts` | Pagination |
| Learning events | INDEX `(user_id, event_type, created_at)` | Progress aggregate |
| Object-word mapping | INDEX `(label)` trên `object_word_mappings` | AI label → Word lookup |
| Media owner | INDEX `(owner_id, media_type)` trên `storage_metadata` | User media lookup |
| Notifications | INDEX `(user_id, read_at, created_at)` | Notification list |
| Leaderboard | INDEX `(scope, score)` hoặc Redis sorted set | Ranking query |

### 6.4 Dictionary import

| Thuộc tính | Chi tiết |
| --- | --- |
| Nguồn | SQLite từ minhqnd/dictionary |
| Quy mô | ~357,729+ từ vựng, ~443,116+ định nghĩa |
| Xử lý | Chuẩn hóa, batch import → MySQL/MariaDB |
| Mapping | → `Word` / `Definition` / `Translation` / `Pronunciation` |
| Chất lượng | Xử lý trùng lặp, thiếu nghĩa, thiếu phiên âm, encoding issues |
| Quy trình | Test subset → validate quality → import full → create indexes |

### 6.5 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| MySQL/MariaDB | Ổn định, mature ecosystem, phù hợp Spring Data JPA; FULLTEXT search cho dictionary |
| JPA/Hibernate | Relationship mapping phong phú cho domain phức tạp (Word ↔ Definition ↔ Translation) |
| Index strategy | Dictionary lookup p95 < 500ms; SRS queue cần sorted index theo due_at |

---

## 7. Redis / Cache Stack

### 7.1 Công nghệ

| Công nghệ | Vai trò | Ghi chú |
| --- | --- | --- |
| **Redis** | In-memory data store | Cache, sorted set, counter |
| **Redisson** | Redis client cho Java | Distributed lock, advanced data structures |

### 7.2 Use cases

| Use case | Cơ chế Redis | TTL đề xuất | Ghi chú |
| --- | --- | --- | --- |
| Dictionary cache | Key-value (word detail/translation) | 1–24h | Cache từ phổ biến, giảm DB query |
| Leaderboard | Sorted set (ZADD/ZRANGEBYSCORE) | 5–15m | Ranking global/period — không full-scan aggregate |
| Home dashboard summary | Key-value aggregate cache | 1–5m | Giảm query nặng progress/stats |
| Mission state | Key-value tạm thời | 5m | Không thay thế DB |
| AI label → Word mapping | Key-value | 24h | Cache mapping label → Word ID |
| Rate limiting | Counter (INCR/EXPIRE) | Window-based | OTP, login, upload nếu triển khai |
| Distributed lock | Redisson lock | Auto-release | Tránh cộng reward trùng concurrent |

### 7.3 Quy tắc

1. Redis **không** là source of truth — mọi dữ liệu rebuild được từ database/events.
2. Key prefix theo môi trường: `snapvocab:{env}:{module}:{key}`.
3. TTL rõ ràng cho mọi cache tạm thời.
4. Redis unavailable → backend degrade gracefully (query DB trực tiếp), không crash.
5. Redis chỉ bắt buộc từ Milestone 3/4 (Leaderboard, Rate limiting, Gamification). Trong M1/M2, hệ thống hoàn toàn có thể chạy không cần Redis nhờ DB index cho tra cứu từ điển.
5. Cache tạm thời phải có TTL; không cache vĩnh viễn.

---

## 8. AI Service Stack

### 8.1 Công nghệ

| Công nghệ | Vai trò | Ghi chú |
| --- | --- | --- |
| **Python 3.10+** | Runtime | Ecosystem ML/AI phong phú |
| **FastAPI** | HTTP API framework | Async, OpenAPI tự động, lightweight |
| **Uvicorn** | ASGI server (dev) | Hot-reload cho development |
| **Gunicorn + Uvicorn** | Production server | 1 worker/GPU (giới hạn VRAM, concurrency=1) |
| **Florence-2-large** | Object detection model | Open-vocabulary, multi-task VLM (F2-v13) |
| **SAM (ViT-H)** | Segment Anything Model | Cắt nền RGBA cho flashcard + xác thực hình học |
| **CLIP (ViT-B/32)** | Vision-language matching | Xác thực ngữ nghĩa label ↔ crop image |
| **PyTorch** | Model runtime | GPU inference (CUDA) |
| **OpenCV / Pillow** | Image preprocessing | Resize, crop, format conversion |
| **NLTK / WordNet** | Linguistic filter | Lọc danh từ cụ thể, kiểm tra thuộc từ điển |
| **NumPy** | Array operations | Bounding box math, score computation |

### 8.2 Pipeline chi tiết

```text
Input Image
  │
  ├── Florence-2 Multi-task Inference
  │   ├── <OD> — Object Detection (standard)
  │   ├── <DENSE_REGION_CAPTION> — Mô tả từng vùng
  │   ├── <CAPTION_TO_PHRASE_GROUNDING> — Self-grounding
  │   └── Tiled OD — 4 ô chồng lấn 60% (bắt vật nhỏ)
  │
  ├── NMS / WBF — Khử box trùng lặp
  │
  ├── Lọc ngôn ngữ (WordNet)
  │   ├── Kiểm tra label thuộc từ điển tiếng Anh
  │   ├── Lọc từ trừu tượng/động từ (chỉ giữ danh từ chỉ vật cụ thể)
  │   └── Chuẩn hóa label (lowercase, singular form)
  │
  ├── Xác thực CLIP
  │   ├── Sàn tuyệt đối: similarity score ≥ 0.23
  │   └── Biên độ tương đối: không thua từ khớp nhất > 0.02
  │
  ├── Xác thực hình học SAM
  │   ├── Generate mask từ bounding box
  │   └── Loại mask quá nhỏ (< 400px area)
  │
  ├── Cắt nền RGBA (SAM)
  │   └── Ảnh trong suốt cho flashcard
  │
  └── Output
      ├── label ∈ dict (đã qua chuỗi lọc)
      ├── detectionSource (nguồn phát hiện vật thể: OD, GROUNDING, SELF...)
      ├── clipScore (điểm xác thực ngữ nghĩa, float)
      ├── boundingBox [x1, y1, x2, y2]
      ├── cropBase64 (chuỗi base64 ảnh RGBA cắt nền)
      └── 1 entry / unique label (gom trùng)
```

### 8.3 Độ tin cậy (Reliability Scoring)

Florence-2 sinh chuỗi văn bản nên **không** có xác suất thật cho từng box. Để phản ánh đúng bản chất, hệ thống bóc tách độ tin cậy thành 2 trường:

1. **`detectionSource`**: Nguồn sinh ra nhãn phát hiện:
   - `OD` (standard OD): Độ tin cậy cao nhất (High)
   - `GROUNDING` (prompt-based grounding): Độ tin cậy cao (High)
   - `SELF` (self-description): Độ tin cậy trung bình (Medium)
   - `DENSE` (region description): Độ tin cậy thấp (Low)
   - `BASE` (fallback): Độ tin cậy cực thấp (Low)
2. **`clipScore`**: Điểm số Cosine Similarity thực sự khi bật xác thực CLIP toàn phần (ví dụ: `0.28`).

Backend sử dụng kết hợp `(sourceAllowlist, clipScoreFloor)` để làm lớp lọc cuối cùng. Mobile UI sử dụng `detectionSource` để map ra các thẻ màu High/Medium/Low.

### 8.4 Hiệu năng

| Metric | Giá trị | Ghi chú |
| --- | --- | --- |
| COCO128 box-F1 | 0.646 | Đánh giá theo IoU ≥ 0.5 |
| COCO128 word-F1 | 0.825 | Đánh giá theo word match (sát mục tiêu sản phẩm) |
| Internet-50 word-precision | 0.885 | Ảnh thực tế từ internet |
| Inference time | ~15–30s (full mode), <10s (fast mode) | GPU T4; fast mode bỏ Tiled OD và SAM |
| Hardware | GPU T4 trở lên | CUDA required |

### 8.5 Contract với backend

| Thuộc tính | Mô tả |
| --- | --- |
| Giao thức | HTTP nội bộ (backend → AI service) |
| Input | `requestId`, ảnh (file/URL), options (`sourceAllowlist`, `clipScoreFloor`, maxObjects) |
| Output | `requestId`, `objects[]` (label, detectionSource, clipScore, bbox, cropBase64), `modelVersion`, `processingTimeMs` |
| Error | Structured error: `INVALID_IMAGE`, `NO_OBJECT`, `MODEL_ERROR`, `TIMEOUT` |
| Timeout | Backend cấu hình (mặc định 60s) |
| Logging | requestId, processingTimeMs, object count, errors |

### 8.6 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| FastAPI | Nhẹ, async, OpenAPI tự động; phù hợp service AI độc lập |
| Florence-2 (zero-shot) | Open-vocabulary — gọi tên vật ngoài tập lớp đóng; đúng mục tiêu học từ mới |
| SAM | Cắt nền chất lượng cho ảnh flashcard + xác thực hình học (mask nhỏ = sai) |
| CLIP | Cửa xác thực ngữ nghĩa chống hallucination; biên độ tương đối hiệu quả |
| Tách service | Backend Java không phải quản lý Python/GPU runtime; scale AI độc lập |
| Zero-shot MVP | Tránh vocabulary collapse khi fine-tune; fine-tune LoRA là hướng mở rộng |

---

## 9. Object Storage Stack

### 9.1 Công nghệ

| Môi trường | Công nghệ | Ghi chú |
| --- | --- | --- |
| Dev/Local | MinIO hoặc S3-compatible storage | Mô phỏng R2/S3 trên máy dev |
| Production | Cloudflare R2 qua S3-compatible API | Bucket private, egress miễn phí |
| Access protocol | S3-compatible API | Backend dùng cùng abstraction cho R2/MinIO |
| Upload flow | Presigned URL (PUT) | Mobile upload trực tiếp, an toàn |
| Metadata | Database `storage_metadata` | owner, object key, MIME, size, type |

### 9.2 Media types

| Media | Source | Giới hạn | Milestone |
| --- | --- | --- | --- |
| Avatar | Edit profile | ≤ 5MB, image/* | M1 |
| Scan image | Camera/detection (optional) | ≤ 10MB, image/* | M2 |
| Crop image (SAM) | AI pipeline → flashcard | — | M2 |
| Item asset | Admin upload (shop/gamification) | — | M4 |

### 9.3 Quy tắc

1. Bucket **private** — không public trực tiếp.
2. Object key do backend sinh (UUID) — **không** dùng tên file user.
3. Presigned URL TTL ngắn (upload: 10–15m, download: ≤ 15m).
4. Validate object ở upload-complete (MIME allowlist + size check).
5. Cleanup orphan objects (object không còn tham chiếu DB) định kỳ.
6. Không xóa binary trước khi DB transaction xác nhận.

### 9.4 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| Cloudflare R2 | S3-compatible, egress miễn phí, private bucket, giá tốt |
| MinIO (dev) | S3-compatible local — cùng code path với production |
| Presigned URL | Mobile upload trực tiếp — giảm tải backend; an toàn (TTL ngắn) |

---

## 10. API Documentation Stack

### 10.1 Công nghệ

| Công nghệ | Vai trò |
| --- | --- |
| **Swagger UI** | Kiểm thử API trực quan trên browser |
| **OpenAPI 3.0** | Chuẩn schema API (JSON/YAML) |
| **Springdoc OpenAPI** | Sinh docs tự động từ Spring Boot controller/DTO |

### 10.2 API groups (tags)

| Tag | Module | Mô tả |
| --- | --- | --- |
| `auth` | Identity | Register, login, OTP, refresh, logout, forgot/reset password |
| `user` | Identity | Profile view/edit |
| `word` | Dictionary | Search, word detail, pronunciation, translation |
| `topic` | Topic | Collections, topics, topic items |
| `storage` | Storage | Presigned upload, upload complete, access URL |
| `recognition` | Recognition | Submit scan, detection result |
| `vocabulary` | Vocabulary | Deck/Note CRUD |
| `flashcard` | Flashcard | Cards, study session, recall rating, templates |
| `quiz` | Quiz | Quiz setup, play, result, history |
| `srs` | SRS | Review queue, FSRS rating submit |
| `progress` | Progress | Stats, home summary, streak, accuracy |
| `gamification` | Gamification | Missions, badges, XP, leaderboard |
| `economy` | Shop | Wallet, shop, inventory |
| `notification` | Notification | Notification list/read, device token |
| `admin` | Admin | User management, dict CRUD, dashboard stats |

### 10.3 Quy tắc

- API groups rõ ràng theo tags.
- DTO schema trong OpenAPI spec.
- Error response format thống nhất (`success/data/error/requestId`).
- Endpoint nội bộ (backend ↔ AI) tách rõ với endpoint public/mobile.
- Swagger UI bật ở dev/staging; tắt hoặc restrict production.
- **Không** hardcode secrets hoặc thông tin môi trường thật vào docs.

---

## 11. Design / UI Stack

### 11.1 Công cụ

| Công cụ / Thành phần | Vai trò |
| --- | --- |
| **Figma** | Thiết kế màn hình, prototype, interactive flow |
| **Design System** (MH-SYSTEM-01) | Chuẩn hóa component, tokens, states trong app |

### 11.2 Design System components

| Category | Components |
| --- | --- |
| **Colors** | Primary, secondary, accent, semantic (success/warning/error/info) |
| **Typography** | Font family, sizes, weights, line height |
| **Buttons** | Primary, secondary, ghost, disabled, loading |
| **Inputs** | Text, password, OTP (digit), search, error state |
| **Cards** | Vocabulary card, quiz card, mission card, topic card |
| **Badges/Chips** | Learning state badge (new/learning/reviewing/mastered suy từ FSRS + interval), source tag (SCAN/DICT/TOPIC), reliability badge |
| **Navigation** | Tab bar, header, back button |
| **Gamification** | XP bar, Coin badge, Streak flame, Level badge, Progress bar, Mission progress |
| **Leaderboard** | Leaderboard row, rank indicator, avatar |
| **Flashcard** | Front/back card, flip animation, FSRS rating buttons |
| **States** | Loading spinner/skeleton, empty state (illustration + CTA), error state (retry button), success |
| **Icons** | Icon set usage, semantic mapping |

### 11.3 Màn hình (30 tổng cộng)

| Trạng thái | Số lượng |
| --- | --- |
| Đã thiết kế (Figma) | 23 |
| Cần bổ sung / làm rõ | 7 |
| **Tổng** | **30** |

---

## 12. Thư viện mã nguồn mở đề xuất

Danh sách dưới đây ưu tiên thư viện phổ biến, dễ thay thế và phù hợp MVP đồ án. Khi triển khai thật cần pin version, kiểm tra license và quét lỗ hổng dependency trước khi release.

| Nhóm | Thư viện / dự án | Áp dụng | Vai trò |
| --- | --- | --- | --- |
| Mobile biometric | `expo-local-authentication` | Mobile | Vân tay/Face ID để mở khóa app hoặc xác nhận thao tác nhạy cảm sau login |
| Mobile secure storage | `expo-secure-store` | Mobile | Lưu refresh token/secret nhỏ trong Keychain/Keystore |
| Mobile camera/media | `expo-camera`, `expo-image-picker`, `expo-image-manipulator` | Mobile | Chụp/chọn/nén ảnh trước khi upload recognition |
| Mobile TTS | `expo-speech` | Mobile | Phát âm từ vựng on-device khi `audioUrl = null`; fallback cho AUDIO field trong flashcard/word detail; xem ARC-12 |
| Mobile STT (Voice Search) | `expo-speech-recognition` | Mobile | Capture giọng nói tiếng Việt → text → gửi `/words/search` reverse-lookup; không cần STT cloud API; xem ARC-12 |
| Mobile state/cache | `@tanstack/react-query`, `zustand`, `react-native-mmkv` | Mobile | Server state, client preferences, cache nhẹ |
| Mobile form/schema | `react-hook-form`, `zod` | Mobile/Web | Form validation thống nhất, giảm lỗi request DTO |
| Mobile UI utility | `react-native-reanimated`, `react-native-gesture-handler`, `lucide-react-native` | Mobile | Animation, gesture, icon |
| Web admin UI | `shadcn/ui`, `@radix-ui/*`, `tailwindcss`, `lucide-react` | Web Admin | Component accessible, style nhanh, icon nhất quán |
| Web admin data | `@tanstack/react-query`, `@tanstack/react-table` | Web Admin | API cache, bảng CMS có sort/filter/pagination |
| Web admin chart | `recharts` | Web Admin | Dashboard chart cho thống kê vận hành/sử dụng |
| Web admin passkey | `@simplewebauthn/browser`, `@simplewebauthn/server` | Web Admin optional | WebAuthn/passkey/MFA cho tài khoản Admin nếu cần hardening |
| Backend auth/security | Spring Security, `jjwt` hoặc `nimbus-jose-jwt`, bcrypt/Argon2 | Backend | JWT, password hashing, token validation |
| Backend mapping/docs | MapStruct, Lombok, Springdoc OpenAPI | Backend | DTO mapping, giảm boilerplate, API docs |
| Backend DB/migration | Hibernate, Flyway hoặc Liquibase, Testcontainers | Backend | ORM, schema migration, integration test với DB thật |
| Backend storage | AWS SDK S3-compatible / MinIO client | Backend | Presigned URL, R2/MinIO abstraction |
| AI service | FastAPI, PyTorch, Transformers, OpenCV, Pillow, NumPy | AI | API inference, model runtime, xử lý ảnh |
| AI models | Florence-2, SAM, CLIP | AI | Object detection zero-shot, segmentation, semantic verification |
| Quality/test | Jest, React Native Testing Library, Playwright, JUnit 5, Mockito, Pytest | Multi-layer | Unit/component/E2E/integration test |

### 12.1 Nguyên tắc chọn thư viện

1. Ưu tiên thư viện mã nguồn mở có cộng đồng lớn, release đều, tài liệu rõ.
2. Không thêm thư viện chỉ để giải quyết việc có thể làm đơn giản bằng framework hiện có.
3. Với auth/storage/crypto, ưu tiên thư viện battle-tested thay vì tự viết.
4. Kiểm tra license tương thích đồ án/sản phẩm trước khi dùng dataset/model/library.
5. Pin version theo milestone; nâng cấp Expo/Next/Spring theo branch riêng và test đầy đủ.
6. Với biometric/passkey, phải phân biệt rõ: biometric là xác nhận cục bộ; passkey/WebAuthn là credential chuẩn phía web/backend.

---

## 13. Testing Stack

### 13.1 Công cụ theo lớp

| Lớp | Công cụ | Kiểm thử |
| --- | --- | --- |
| **Mobile unit/component** | Jest, React Native Testing Library | Component render, form validation, state management |
| **Mobile E2E** | Maestro hoặc công cụ E2E tương đương | Auth flow, biometric unlock mock, search, camera mock, quiz, profile |
| **Web admin unit/component** | Jest/Vitest, React Testing Library | Admin table/form/dialog, route guard, validation schema |
| **Web admin E2E** | Playwright | Admin login, user management, dictionary/topic CRUD, mission/shop CMS |
| **Backend unit** | JUnit 5, Mockito | Service logic, validators, FSRS algorithm, reward rules |
| **Backend integration** | Spring Boot Test, Testcontainers (nếu có) | Controller + DB + Security end-to-end |
| **API contract** | Swagger/OpenAPI + API client tests | Request/response schema validation |
| **AI service** | Pytest + sample images | Inference endpoint, no-object, invalid image, timeout |
| **Storage** | Integration smoke test | Presigned upload, upload-complete, private access |
| **System smoke** | Script/API tests | Auth, word search, upload, recognition, save word, quiz/SRS |

### 13.2 Test flows quan trọng

| # | Flow | Modules liên quan | Priority |
| --- | --- | --- | --- |
| 1 | Register → OTP verify → Login → Refresh → Logout | Identity | Critical |
| 2 | Search word → Word detail → Save to Deck | Dictionary, Vocabulary | Critical |
| 3 | Camera/detection → AI detect → Map word → Save Note/Card | Recognition, AI, Vocabulary | Critical |
| 4 | Flashcard session → FSRS rating → ReviewLog | Flashcard, SRS | Critical |
| 5 | Quiz setup → Play → Submit → Result | Quiz | High |
| 6 | SRS due queue → Review → Rating → Card update | SRS, Flashcard | High |
| 7 | Avatar upload (presigned) → Upload complete → Profile update | Storage, Identity | High |
| 8 | Topic browse → Save from topic → Note created | Topic, Vocabulary | Medium |
| 9 | Reward idempotency (retry claim → no duplicate XP/coin) | Gamification | High |
| 10 | No-object / low-reliability / AI timeout → error state | Recognition, AI | High |
| 11 | Admin login → dashboard → dictionary/topic edit | Web Admin, Identity, Dictionary/Topic | High |
| 12 | Biometric unlock enabled → app restart → local auth success/fail → token access policy | Mobile, Identity | High |

### 13.3 Lý do chọn

| Quyết định | Lý do |
| --- | --- |
| Jest + RNTL | Standard cho React Native; component testing nhanh |
| Playwright | Phù hợp web admin E2E trên browser thật: login, table, form, route guard |
| JUnit + Mockito | Standard cho Spring Boot; mock dependencies dễ |
| Testcontainers | Integration test với DB thật trong container; không mock SQL |
| Maestro | Mobile E2E declarative, dễ viết flow test |
| Pytest | Standard cho Python; AI inference test đơn giản |

---

## 14. Build & Deployment Stack

### 14.1 Công nghệ build

| Thành phần | Build tool | Artifact |
| --- | --- | --- |
| Mobile | Expo CLI / EAS Build | APK, IPA, OTA update |
| Web Admin | Next.js build | Static/Node web artifact hoặc Docker image |
| Backend | Maven | JAR (hoặc Docker container) |
| AI Service | pip/requirements.txt + model weights | Docker container (nếu deploy server) |
| Database | SQL migration scripts | Schema + seed data |
| Redis | Managed Redis hoặc container | — |
| Storage | R2 bucket (prod) / MinIO (dev) | — |
| Docs | Markdown + Swagger/OpenAPI | — |

### 14.2 CI/CD pipeline

```text
┌─────────────────────────────────────────────────────────────────────┐
│                          CI/CD PIPELINE                              │
│                                                                     │
│  MOBILE                                                             │
│  commit → typecheck → lint → jest → build preview → test flows     │
│                                      → build release (milestone)    │
│                                                                     │
│  WEB ADMIN                                                          │
│  commit → typecheck → lint → component tests → Playwright smoke    │
│                                         → build/deploy admin site   │
│                                                                     │
│  BACKEND                                                            │
│  commit → mvn test → package JAR → deploy staging                  │
│                                   → smoke test (health/auth/word)   │
│                                   → deploy production (approved)    │
│                                                                     │
│  AI SERVICE                                                         │
│  commit → install deps → pytest (inference sample)                 │
│                        → package container → deploy staging         │
│                        → backend integration test                   │
│                        → deploy production (approved)               │
│                                                                     │
│  DATABASE                                                           │
│  schema change → migration script → apply staging → verify         │
│                                   → apply production                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 15. Environment Configuration

### 15.1 Biến cấu hình theo nhóm

| Nhóm | Biến cấu hình | Ghi chú |
| --- | --- | --- |
| **Mobile** | `API_BASE_URL`, `APP_ENV`, feature flags | Theo môi trường (dev/staging/prod) |
| **Web Admin** | `NEXT_PUBLIC_API_BASE_URL`, `ADMIN_APP_ENV`, `ADMIN_BASE_URL`, feature flags | Theo môi trường; không đưa secret vào biến public |
| **Backend app** | `APP_ENV`, `API_BASE_PATH`, `APP_BASE_URL` | Server config |
| **Database** | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD` | Không commit secret |
| **Redis** | `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` | Password nếu yêu cầu |
| **JWT** | `JWT_SECRET`, `JWT_ACCESS_TTL`, `JWT_REFRESH_TTL` | Secret đủ mạnh, rotate được |
| **Mail** | `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_FROM` | SMTP/API provider |
| **Storage** | `STORAGE_ENDPOINT`, `STORAGE_BUCKET`, `STORAGE_ACCESS_KEY`, `STORAGE_SECRET_KEY`, `STORAGE_REGION` | R2/MinIO/S3-compatible |
| **AI Service** | `AI_SERVICE_BASE_URL`, `AI_SERVICE_TIMEOUT_MS`, `AI_SERVICE_TOKEN` | Timeout mặc định 60s |
| **Upload** | `MAX_AVATAR_SIZE`, `MAX_SCAN_SIZE`, `ALLOWED_IMAGE_TYPES` | Validate media |
| **OpenAPI** | `OPENAPI_ENABLED`, `SWAGGER_UI_ENABLED` | Env-gated |
| **OTP** | `OTP_TTL_MINUTES`, `OTP_MAX_ATTEMPTS`, `OTP_RESEND_COOLDOWN_SECONDS` | Safety rules |

### 15.2 Quy tắc

1. **Không** commit secrets vào VCS.
2. **Không** copy password/token thật vào tài liệu.
3. Tách config local/staging/production.
4. Production secrets quản lý bằng secret manager/env an toàn.
5. Mobile config tách biệt per build variant.
6. Web admin không expose secret qua `NEXT_PUBLIC_*`; mọi secret phải nằm ở backend/env server-side.

---

## 16. Observability Stack

### 16.1 Logging

| Hạng mục | Nội dung cần log |
| --- | --- |
| API request | `requestId`, method, path, status, latency, userId |
| Auth | Login fail/success, OTP fail/expire, refresh fail, registration, biometric unlock enable/disable event (không log biometric data) |
| Admin | Admin login/logout, forbidden access, user lock/unlock, dictionary/topic/gamification changes |
| Recognition | `requestId`, image metadata, modelVersion, processingTimeMs, object count, errors |
| Storage | upload-init, upload-complete, validation fail, orphan cleanup |
| Dictionary | Lookup latency, not-found rate, import job result |
| Learning | Save word, flashcard recall, quiz attempt, SRS review |
| Gamification | XP/coin transaction, mission complete, badge award, idempotent duplicate ignored |
| Notification | Push sent/fail, device token register/expire |
| System | DB/Redis/Storage/AI availability, exception rate, unhandled errors |

### 16.2 Health checks

| Check | Thành phần | Mục đích |
| --- | --- | --- |
| `/health` | Backend | Backend process sống |
| DB check | Backend | Kết nối database |
| Redis check | Backend | Kết nối Redis |
| Storage check | Backend | Bucket/presigned URL hoạt động |
| AI check | Backend/AI | AI model loaded + ready |
| Mail check | Backend | Mail provider config hợp lệ |

### 16.3 Alerts

| Alert | Severity |
| --- | --- |
| Backend error rate > 5% (5xx) | Critical |
| AI service unavailable | Critical |
| AI timeout rate > 20% | Warning |
| Database connection failure | Critical |
| Redis unavailable | Warning |
| Storage upload failure rate > 10% | Warning |
| Mail OTP gửi lỗi liên tục | Warning |

---

## 17. Tech Stack theo Milestone

| Milestone | Stack trọng tâm |
| --- | --- |
| **M1 — Core Auth & Vocabulary** | React Native/Expo, Expo SecureStore, Spring Boot, Spring Security/JWT, MySQL/MariaDB (dictionary import 357K+), MinIO/S3 (avatar), Spring Mail (OTP), Swagger |
| **M2 — Camera/Recognition** | Expo Camera/Image Picker/Image Manipulator, Storage scan image, FastAPI + Florence-2 + SAM + CLIP (GPU T4), Recognition API, ObjectWordMapping |
| **M3 — Learning Engine** | Flashcard/Quiz/SRS backend services, CardTemplate, Progress aggregate, Notification (Expo Push/FCM), TanStack Query learning screens, biometric unlock optional, tests SRS/quiz |
| **M4 — Gamification & Admin Production** | Redis leaderboard sorted set, Mission/Badge/Coin/Shop services, Next.js Admin CMS, shadcn/ui, TanStack Table, Playwright smoke, Cloudflare R2 production, Observability hardening, CI/CD |

---

## 18. Rủi ro tech stack

| # | Rủi ro | Ảnh hưởng | Kiểm soát |
| --- | --- | --- | --- |
| 1 | Florence pipeline nặng (GPU T4, ~15–30s/ảnh) | AI service chậm, UX kém | Hàng đợi AI giới hạn worker/GPU, quota scan/ngày/Learner, timeout 60s, GPU T4+, scale AI riêng; dev có thể mock inference |
| 2 | Florence hallucination (label sai) | Từ sai trên flashcard | CLIP verification (sàn 0.23 + biên độ 0.02), SAM geometry check |
| 3 | Vocabulary collapse nếu fine-tune | Mất khả năng gọi từ mới | MVP giữ zero-shot; fine-tune kèm phép đo word-F1 hai chiều |
| 4 | Dictionary lớn (357K+ từ) | Import/search chậm | Batch import, index, cache từ phổ biến (Redis) |
| 5 | Mobile camera permission phức tạp | Scan flow lỗi trên thiết bị | Test thiết bị thật, fallback chọn ảnh từ thư viện |
| 6 | JWT/refresh flow sai | User bị logout hoặc rủi ro bảo mật | Test auth lifecycle đầy đủ, revoke refresh token đúng |
| 7 | Redis bị dùng như DB chính | Mất dữ liệu khi Redis reset | Chỉ dùng Redis làm cache, rebuild từ DB/events |
| 8 | Reward cộng trùng (retry) | Sai coin/XP/leaderboard | Idempotency key + unique transaction + distributed lock |
| 9 | R2/MinIO khác biệt config | Upload lỗi giữa dev/prod | S3-compatible abstraction, smoke test upload per env |
| 10 | AI label không khớp dictionary Anh-Việt | Không tạo được Note từ scan | ObjectWordMapping + synonym table, dictionary miss state UI |
| 11 | Expo SDK breaking changes | Mobile build fail | Pin Expo SDK version, test upgrade trên branch riêng |
| 12 | Web admin expose secret qua biến public hoặc bundle | Lộ credential/API config nhạy cảm | Chỉ dùng `NEXT_PUBLIC_*` cho cấu hình public; secret nằm backend/server env; review build env trước deploy |
| 13 | Biometric bị hiểu nhầm là xác thực backend | Thiết kế auth sai, khó audit | Biometric chỉ unlock local secret; backend vẫn dùng JWT/refresh; passkey/WebAuthn là optional riêng cho admin |
| 14 | Premium bị hiểu là payment thật | Vượt phạm vi đồ án | MVP không xử lý tiền thật; payment thật cần stack riêng |

---

## 19. Tài liệu liên quan

| Tài liệu | Đường dẫn | Vai trò |
| --- | --- | --- |
| Đặc tả yêu cầu (SRS) | [specs.md](../spec/specs.md) | Source of truth |
| System Architecture | [sa.md](./sa.md) | Kiến trúc tổng thể |
| Server & Deployment | [server.md](./server.md) | Môi trường, ops, smoke test |
| Luồng nghiệp vụ | [buss_mainflow.md](../spec/buss_mainflow.md) | BF — actor, bước, ngoại lệ |
| Phân rã phân hệ | [phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md) | SS — ranh giới module, entities, API |
| Phân rã tính năng | [phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md) | F — sub-feature + AC theo FR |
| Phân rã màn hình | [phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md) | MH — UI map, navigation, states |
---

## 20. Checklist nghiệm thu tech stack

- [x] Mobile stack: React Native/Expo/TypeScript hỗ trợ auth, biometric unlock, camera, detection, dictionary, topic, vocabulary, flashcard, quiz, SRS, progress, gamification, profile, notification.
- [x] Web admin stack: Next.js/React/TypeScript + Tailwind/shadcn/ui + TanStack Query/Table cho CMS quản trị Admin.
- [x] Backend stack: Spring Boot REST API + Spring Security/JWT + JPA/Hibernate + Spring Mail + Springdoc OpenAPI.
- [x] AI service stack: FastAPI + Florence-2 + SAM + CLIP (F2-v13 zero-shot) tách riêng, GPU T4+.
- [x] Database: MySQL/MariaDB source of truth; dictionary import (357K+ từ) chuẩn hóa; index chiến lược cho lookup/SRS/leaderboard.
- [x] Redis: cache dictionary/ranking, leaderboard sorted set, rate limiting — **không** source of truth.
- [x] Object storage: R2/S3-compatible, private bucket, presigned URL, MIME/size validation, orphan cleanup.
- [x] API docs: Swagger/OpenAPI đủ cho mobile tích hợp; error envelope thống nhất; env-gated.
- [x] Security: JWT + refresh, OTP safety, biometric local unlock, optional admin passkey/WebAuthn, upload validation, presigned TTL, idempotent rewards, no secret logging.
- [x] Testing: Jest + RNTL (mobile), Playwright (web admin), JUnit + Mockito (backend), Pytest (AI), integration + smoke test.
- [x] Build/CI: Maven (backend), Expo/EAS (mobile), Next.js build (web admin), Docker (AI), migration scripts (DB).
- [x] Environment config: không chứa secrets trong repo/docs; tách config per env.
- [x] Design System: components cho gamification (XP, coin, badge, streak, progress bar, mission, leaderboard).
- [x] Stack chia theo milestone M1→M4 để triển khai tuần tự, không chặn MVP cốt lõi.
- [x] Thư viện mã nguồn mở đề xuất đã chia theo mobile/web admin/backend/AI/testing và có nguyên tắc chọn.
- [x] Rủi ro tech stack đã nhận diện và có phương án kiểm soát.
- [x] Canonical model: Deck → Note → Card + ReviewLog. Không `SavedWord`/`UserWord`.
- [x] AI pipeline: Florence-2 + SAM + CLIP zero-shot. Không YOLO.
- [x] SRS: FSRS trên Card.
- [x] Actors: Guest, Learner, Admin (CMS web riêng).
