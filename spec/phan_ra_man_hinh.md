# Phân rã màn hình — SnapVocab

> Truy vết: [specs.md](./specs.md) · [buss_mainflow.md](./buss_mainflow.md) · [phan_ra_tinh_nang.md](./phan_ra_tinh_nang.md) · [phan_ra_phan_he_he_thong.md](./phan_ra_phan_he_he_thong.md).  
> **Canonical:** Florence-2 pipeline · Deck/Note/Card · Actor Guest/Learner/Admin.

---

## 1. Phạm vi màn hình theo milestone

| Nhóm MH                                  | MVP M1–M4    | Ghi chú                         |
| ---------------------------------------- | ------------ | ------------------------------- |
| ONBOARD, AUTH, MAIN, CAMERA, DICT, TOPIC | **In scope** | M1–M2                           |
| LEARN (Flashcard, Quiz, SRS)             | **In scope** | M1 (basic), M3 (full)           |
| STATS, PROGRESS                          | **In scope** | M3                              |
| GAME, ECONOMY                            | **M4**       | Coin only, no real payment      |
| PROFILE, NOTIFICATION, SETTINGS          | **In scope** | M1 (profile), M3 (notification) |
| Admin CMS screens                        | **Tách web** | Không nằm mobile; FR-13, SS-17  |
| SYSTEM (Design System, UI States)        | **Ongoing**  | Cross-cutting, cần từ M1        |

**Thuật ngữ UI:** "Vocabulary / từ đã lưu" = danh sách `Note` trong `Deck`. Word Detail save → Note + Card.

---

## 2. Quy ước mã màn hình

```text
MH-{GROUP}-{nn}
```

| Nhóm    | Ý nghĩa                                             | Milestone |
| ------- | --------------------------------------------------- | --------- |
| ONBOARD | Onboarding/welcome cho Guest                        | M1        |
| AUTH    | Login, signup, forgot password, OTP, reset password | M1        |
| MAIN    | Home Dashboard, Learn Hub                           | M1        |
| CAMERA  | Camera Scan và Detection Result                     | M2        |
| DICT    | Search, Word Detail, Voice Search                   | M1        |
| TOPIC   | Collections, Topic List, Topic Items                | M1        |
| VOCAB   | My Vocabulary (Deck/Note list), Deck Detail         | M1        |
| LEARN   | Flashcards, Quiz, SRS Review, Template Management   | M1, M3    |
| STATS   | Stats/Progress, Level                               | M3        |
| GAME    | Missions, Achievements, Leaderboard, Rewards        | M4        |
| ECONOMY | Wallet, Shop, Inventory                             | M4        |
| PROFILE | Profile, Edit Profile, Settings, Notifications      | M1, M3    |
| SYSTEM  | Design System, Loading/Empty/Error States           | Ongoing   |

---

## 3. Sơ đồ điều hướng tổng quan

```mermaid
flowchart TD
    subgraph Guest
        OB["MH-ONBOARD-01<br/>Onboarding"]
        LG["MH-AUTH-01<br/>Login"]
        SU["MH-AUTH-02<br/>Signup"]
        FP["MH-AUTH-03<br/>Forgot Password"]
        OTP["MH-AUTH-04<br/>OTP Verify"]
        RP["MH-AUTH-05<br/>Reset Password"]
    end

    subgraph Main["Main Navigation (Bottom Tabs)"]
        HOME["MH-MAIN-01<br/>Home Dashboard"]
        LEARN_HUB["MH-MAIN-02<br/>Learn Hub"]
        CAM["MH-CAMERA-01<br/>Camera Scan"]
        SEARCH["MH-DICT-01<br/>Search"]
        PROF["MH-PROFILE-01<br/>Profile"]
    end

    subgraph Dict["Dictionary & Topic"]
        WD["MH-DICT-02<br/>Word Detail"]
        VS["MH-DICT-03<br/>Voice Search"]
        COL["MH-TOPIC-01<br/>Collections"]
        TOP["MH-TOPIC-02<br/>Topic Items"]
    end

    subgraph Vocab["Vocabulary"]
        DECK["MH-VOCAB-01<br/>My Vocabulary"]
        DD["MH-VOCAB-02<br/>Deck Detail"]
    end

    subgraph Camera["Camera & Detection"]
        DET["MH-CAMERA-02<br/>Detection Result"]
    end

    subgraph Learning["Learning Engine"]
        FC["MH-LEARN-01<br/>Flashcards"]
        QS["MH-LEARN-02<br/>Quiz Setup"]
        QP["MH-LEARN-03<br/>Quiz Play"]
        QR["MH-LEARN-04<br/>Quiz Result"]
        SRS["MH-LEARN-05<br/>SRS Review"]
        TM["MH-LEARN-06<br/>Template Mgmt"]
    end

    subgraph Stats["Progress & Gamification"]
        ST["MH-STATS-01<br/>Stats"]
        LV["MH-STATS-02<br/>Level"]
        MI["MH-GAME-02<br/>Missions"]
        AC["MH-GAME-03<br/>Achievements"]
        LB["MH-GAME-01<br/>Leaderboard"]
    end

    subgraph Economy["Economy"]
        WA["MH-ECONOMY-01<br/>Wallet"]
        SH["MH-ECONOMY-02<br/>Shop"]
        IN["MH-ECONOMY-03<br/>Inventory"]
    end

    subgraph Profile["Profile & Settings"]
        EP["MH-PROFILE-02<br/>Edit Profile"]
        SE["MH-PROFILE-03<br/>Settings"]
        NO["MH-PROFILE-04<br/>Notifications"]
    end

    OB --> LG & SU
    SU --> OTP
    FP --> OTP --> RP
    LG --> HOME
    RP --> LG

    HOME --> LEARN_HUB & CAM & SEARCH & PROF
    HOME --> SRS & MI & ST

    CAM --> DET --> WD
    SEARCH --> WD
    WD --> DECK

    HOME --> COL --> TOP --> WD
    LEARN_HUB --> DECK & FC & QS & SRS
    DECK --> DD --> FC & QS
    FC --> SRS
    QS --> QP --> QR

    PROF --> EP & SE & NO & AC & WA
    WA --> SH --> IN

    HOME --> LB
    MI --> HOME
```

---

## 4. Danh sách màn hình tổng quan

| Mã            | Tên màn hình               | Actor   | Feature Area            | Milestone | Trạng thái  |
| ------------- | -------------------------- | ------- | ----------------------- | --------- | ----------- |
| MH-ONBOARD-01 | Onboarding                 | Guest   | AUTH                    | M1        | Đã thiết kế |
| MH-AUTH-01    | Login                      | Guest   | AUTH                    | M1        | Đã thiết kế |
| MH-AUTH-02    | Signup/Register            | Guest   | AUTH                    | M1        | Đã thiết kế |
| MH-AUTH-03    | Forgot Password            | Guest   | AUTH                    | M1        | Đã thiết kế |
| MH-AUTH-04    | OTP/Email Verification     | Guest   | AUTH                    | M1        | Cần bổ sung |
| MH-AUTH-05    | Reset Password             | Guest   | AUTH                    | M1        | Cần bổ sung |
| MH-MAIN-01    | Home Dashboard             | Learner | PROGRESS, SRS, GAME     | M1        | Đã thiết kế |
| MH-MAIN-02    | Learn Hub                  | Learner | VOCAB, FLASH, QUIZ, SRS | M1        | Đã thiết kế |
| MH-CAMERA-01  | Camera Scan                | Learner | RECOG, STORAGE          | M2        | Đã thiết kế |
| MH-CAMERA-02  | Detection Result           | Learner | RECOG, DICT, VOCAB      | M2        | Đã thiết kế |
| MH-DICT-01    | Search/Dictionary          | Learner | DICT                    | M1        | Đã thiết kế |
| MH-DICT-02    | Word Detail                | Learner | DICT, VOCAB             | M1        | Cần bổ sung |
| MH-DICT-03    | Voice Search               | Learner | DICT                    | M1        | Cần bổ sung |
| MH-TOPIC-01   | Collections & Topics       | Learner | TOPIC                   | M1        | Cần bổ sung |
| MH-TOPIC-02   | Topic Items                | Learner | TOPIC, VOCAB            | M1        | Cần bổ sung |
| MH-VOCAB-01   | My Vocabulary (Deck List)  | Learner | VOCAB                   | M1        | Đã thiết kế |
| MH-VOCAB-02   | Deck Detail                | Learner | VOCAB, FLASH, QUIZ      | M1        | Cần bổ sung |
| MH-LEARN-01   | Flashcards Study Session   | Learner | FLASH, SRS              | M1        | Đã thiết kế |
| MH-LEARN-02   | Quiz Setup                 | Learner | QUIZ                    | M3        | Đã thiết kế |
| MH-LEARN-03   | Quiz Play                  | Learner | QUIZ                    | M3        | Đã thiết kế |
| MH-LEARN-04   | Quiz Result                | Learner | QUIZ, PROGRESS, GAME    | M3        | Đã thiết kế |
| MH-LEARN-05   | SRS Review Session         | Learner | SRS, FLASH              | M3        | Đã thiết kế |
| MH-LEARN-06   | Template Management        | Learner | FLASH                   | M3        | Cần bổ sung |
| MH-STATS-01   | Stats/Progress             | Learner | PROGRESS                | M3        | Đã thiết kế |
| MH-STATS-02   | Level Progress             | Learner | PROGRESS, GAME          | M4        | Đã thiết kế |
| MH-GAME-01    | Leaderboard                | Learner | GAME                    | M4        | Đã thiết kế |
| MH-GAME-02    | Missions                   | Learner | GAME                    | M4        | Đã thiết kế |
| MH-GAME-03    | Achievements/Badges        | Learner | GAME                    | M4        | Đã thiết kế |
| MH-ECONOMY-01 | Wallet                     | Learner | GAME                    | M4        | Đã thiết kế |
| MH-ECONOMY-02 | Shop                       | Learner | GAME                    | M4        | Đã thiết kế |
| MH-ECONOMY-03 | Inventory/My Items         | Learner | GAME                    | M4        | Đã thiết kế |
| MH-PROFILE-01 | Profile                    | Learner | PROFILE, PROGRESS, GAME | M1        | Đã thiết kế |
| MH-PROFILE-02 | Edit Profile               | Learner | PROFILE, STORAGE        | M1        | Đã thiết kế |
| MH-PROFILE-03 | Settings                   | Learner | PROFILE, AUTH, NOTIF    | M1        | Đã thiết kế |
| MH-PROFILE-04 | Notifications              | Learner | NOTIF                   | M3        | Đã thiết kế |
| MH-SYSTEM-01  | Design System              | Dev     | SYSTEM                  | Ongoing   | Đã thiết kế |
| MH-SYSTEM-02  | Empty/Error/Loading States | All     | SYSTEM                  | Ongoing   | Cần đặc tả  |

---

## 5. ONBOARD & AUTH Screens

### MH-ONBOARD-01 — Onboarding

| Thuộc tính | Mô tả                                             |
| ---------- | ------------------------------------------------- |
| Actor      | Guest                                             |
| Feature    | F-AUTH-01, F-AUTH-03                              |
| BF         | BF-01, BF-02                                      |
| FR         | FR-01                                             |
| Mục tiêu   | Giới thiệu SnapVocab và dẫn tới đăng nhập/đăng ký |

**Dữ liệu hiển thị:**

- Logo/tên SnapVocab
- Value proposition slides: AI scan objects → learn vocabulary → review with SRS → gamification
- CTA: **Login** → MH-AUTH-01, **Signup** → MH-AUTH-02
- Nếu có phiên hợp lệ → auto redirect MH-MAIN-01

---

### MH-AUTH-01 — Login

| Thuộc tính | Mô tả                                            |
| ---------- | ------------------------------------------------ |
| Actor      | Guest                                            |
| Feature    | F-AUTH-03, F-AUTH-04, F-AUTH-09                  |
| BF         | BF-02                                            |
| FR         | FR-01.03, FR-01.04, FR-01.08                     |
| Mục tiêu   | Đăng nhập bằng email/password hoặc sinh trắc học |

**Dữ liệu/form:**

- Email input
- Password input (masked)
- Biometric login button (nếu đã bật — Should)
- "Quên mật khẩu?" link → MH-AUTH-03
- "Chưa có tài khoản? Đăng ký" → MH-AUTH-02
- Login button

**Trạng thái UI cần có:**

- Field validation (email format, password required)
- Sai thông tin → generic message "Thông tin đăng nhập không đúng"
- Tài khoản chưa xác thực → redirect MH-AUTH-04
- Tài khoản bị khóa → message "Tài khoản đã bị khóa"
- Loading khi submit

---

### MH-AUTH-02 — Signup/Register

| Thuộc tính | Mô tả                     |
| ---------- | ------------------------- |
| Actor      | Guest                     |
| Feature    | F-AUTH-01, F-AUTH-02      |
| BF         | BF-01                     |
| FR         | FR-01.01, FR-01.02        |
| Mục tiêu   | Tạo tài khoản Learner mới |

**Dữ liệu/form:**

- Email input
- Display name / full name
- Password (+ policy hint: min length, ký tự đặc biệt)
- Confirm password
- Terms agreement (nếu có)
- Signup button

**Kết quả:** Signup thành công → MH-AUTH-04 (OTP Verify)

**Trạng thái UI:**

- Email đã tồn tại → "Email đã được sử dụng"
- Password không đạt policy → hiển thị yêu cầu cụ thể
- Loading khi submit

---

### MH-AUTH-03 — Forgot Password

| Thuộc tính | Mô tả                             |
| ---------- | --------------------------------- |
| Actor      | Guest                             |
| Feature    | F-AUTH-05                         |
| BF         | BF-03                             |
| FR         | FR-01.05                          |
| Mục tiêu   | Yêu cầu OTP để khôi phục mật khẩu |

**Dữ liệu/form:**

- Email input
- Submit button → "Gửi mã OTP"
- Back to login link

**Kết quả:** Submit → MH-AUTH-04 (OTP) → MH-AUTH-05 (Reset Password)

---

### MH-AUTH-04 — OTP/Email Verification

| Thuộc tính | Mô tả                                              |
| ---------- | -------------------------------------------------- |
| Actor      | Guest                                              |
| Feature    | F-AUTH-02, F-AUTH-05                               |
| BF         | BF-01, BF-03                                       |
| FR         | FR-01.02, FR-01.05                                 |
| Mục tiêu   | Nhập OTP để xác thực tài khoản hoặc reset password |

**Dữ liệu/form:**

- Email đang xác thực (read-only)
- OTP input (4–6 digit)
- Countdown timer (resend cooldown ≥ 60s)
- Resend OTP button (disabled khi countdown)
- Verify button

**Trạng thái UI:**

- OTP sai → "Mã không đúng, còn X lần thử" (max 5)
- OTP hết hạn → "Mã đã hết hạn, vui lòng gửi lại"
- Quá 5 lần → khóa OTP hiện tại, yêu cầu tạo mới

**Kết quả:**

- Xác thực signup → account ACTIVE → MH-AUTH-01 hoặc auto login → MH-MAIN-01
- Xác thực reset → MH-AUTH-05

---

### MH-AUTH-05 — Reset Password

| Thuộc tính | Mô tả                               |
| ---------- | ----------------------------------- |
| Actor      | Guest                               |
| Feature    | F-AUTH-05                           |
| BF         | BF-03                               |
| FR         | FR-01.05                            |
| Mục tiêu   | Đặt mật khẩu mới sau khi OTP hợp lệ |

**Dữ liệu/form:**

- New password (+ policy hint)
- Confirm new password
- Submit button

**Kết quả:** Reset thành công → MH-AUTH-01 (Login)

---

## 6. MAIN Screens

### MH-MAIN-01 — Home Dashboard

| Thuộc tính | Mô tả                                     |
| ---------- | ----------------------------------------- |
| Actor      | Learner                                   |
| Feature    | F-PROG-01, F-PROG-06, F-SRS-03, F-GAME-03 |
| BF         | BF-10, BF-11, BF-12                       |
| FR         | FR-07.04, FR-08.06, FR-09.03              |
| Mục tiêu   | Tổng quan học tập và điều hướng nhanh     |

**Dữ liệu hiển thị:**

- Greeting theo tên Learner
- **Progress widget:** số từ đã lưu / đã học / mastered theo learning-state map
- **SRS due count:** số Card cần ôn hôm nay + overdue badge
- **Streak:** chuỗi ngày học liên tiếp (flame icon)
- **XP / Coin / Level** (M4)
- **Mission nổi bật:** daily mission progress bar (M4)
- **Quick actions:** Camera Scan, Search, Flashcard, Quiz, SRS Review
- **Recently learned:** 3–5 từ gần nhất

**Hành động điều hướng:**

| Hành động     | Đích         |
| ------------- | ------------ |
| Camera        | MH-CAMERA-01 |
| Search        | MH-DICT-01   |
| SRS Review    | MH-LEARN-05  |
| Flashcard     | MH-LEARN-01  |
| Quiz          | MH-LEARN-02  |
| Missions      | MH-GAME-02   |
| View Progress | MH-STATS-01  |
| Collections   | MH-TOPIC-01  |

---

### MH-MAIN-02 — Learn Hub

| Thuộc tính | Mô tả                                      |
| ---------- | ------------------------------------------ |
| Actor      | Learner                                    |
| Feature    | VOCAB, FLASH, QUIZ, SRS                    |
| BF         | BF-07, BF-08, BF-09, BF-10                 |
| Mục tiêu   | Trung tâm điều hướng các hoạt động học tập |

**Dữ liệu hiển thị:**

- Card **My Vocabulary** (Note count, CTA → MH-VOCAB-01)
- Card **Flashcards** (Card to review, CTA → MH-LEARN-01)
- Card **Quiz** (Available quiz, CTA → MH-LEARN-02)
- Card **SRS Review** (Due count, CTA → MH-LEARN-05)
- Card **Collections** (Topic count, CTA → MH-TOPIC-01)
- Gợi ý "Tiếp tục học" dựa trên hoạt động gần nhất

---

## 7. CAMERA & DETECTION Screens

### MH-CAMERA-01 — Camera Scan

| Thuộc tính | Mô tả                                         |
| ---------- | --------------------------------------------- |
| Actor      | Learner                                       |
| Feature    | F-RECOG-01, F-RECOG-02                        |
| BF         | BF-06                                         |
| FR         | FR-02.01, FR-02.02                            |
| Mục tiêu   | Chụp ảnh hoặc chọn ảnh để nhận diện đối tượng |

**Dữ liệu/UX:**

- Camera preview (full screen)
- Capture button (center)
- Gallery picker button
- Flash toggle
- Permission request UX (camera/gallery)
- Hướng dẫn: "Chụp rõ vật thể để nhận diện từ vựng"
- Loading overlay khi gửi ảnh xử lý
- Lượt scan còn lại trong ngày (`remainingScansToday`) và thời điểm reset khi gần/hết lượt

**Trạng thái UI:**

- Camera permission denied → settings redirect
- Loading scan → spinner + "Đang nhận diện..."; nếu job còn chờ thì hiển thị "Đang xếp hàng" + vị trí ước tính
- Hết lượt scan → "Bạn đã dùng hết lượt scan hôm nay" + thời điểm reset + CTA học từ đã lưu
- Upload failed → retry button

---

### MH-CAMERA-02 — Detection Result

| Thuộc tính | Mô tả                                          |
| ---------- | ---------------------------------------------- |
| Actor      | Learner                                        |
| Feature    | F-RECOG-04 → F-RECOG-12, F-DICT-06, F-VOCAB-02 |
| BF         | BF-06                                          |
| FR         | FR-02.04 → FR-02.11                            |
| Mục tiêu   | Hiển thị kết quả nhận diện và cho phép lưu từ  |

**Dữ liệu hiển thị:**

- Ảnh đã scan (có bounding box overlay nếu hỗ trợ)
- Danh sách detected objects:
    - Object label (từ tiếng Anh)
    - Source/Reliability badge (High/Medium/Low)
    - Nghĩa tiếng Việt
    - IPA / nút phát âm
    - Ảnh crop (nếu có cropUrl)
    - **Save** button per object → lưu vào Deck gần nhất/mặc định, có option **Đổi Deck** trước khi xác nhận
    - Trạng thái đã lưu (nếu đã có trong Deck đang chọn)
- Deck đích hiện tại: tên Deck + action **Đổi Deck**
- "Lưu tất cả" button → lưu các object chưa trùng vào Deck đang chọn; nếu có trùng, báo số từ bị bỏ qua
- "Chụp lại" button → MH-CAMERA-01

**Trạng thái UI bắt buộc:**

| Tình huống         | UI cần thể hiện                                         |
| ------------------ | ------------------------------------------------------- |
| No object detected | "Không nhận diện được vật thể" + CTA "Thử ảnh khác"     |
| All low reliability| "Không tìm thấy vật thể có độ tin cậy cao" + CTA "Thử ảnh rõ hơn"        |
| Dictionary miss    | Có label nhưng đánh dấu "Chưa có từ vựng tương ứng"     |
| Queued / processing | "Đang xếp hàng"/"Đang nhận diện" + vị trí/thời gian chờ ước tính + nút hủy |
| Quota exceeded     | "Bạn đã dùng hết lượt scan hôm nay" + `resetAt` + CTA quay lại học |
| AI error / timeout | "Xử lý thất bại" + CTA "Thử lại" hoặc "Quay lại camera" |
| Queue full         | "Hệ thống đang quá tải, thử lại sau"; không tự retry liên tục |
| Partial results    | Hiển thị object có data, ẩn/ghi chú object thiếu data   |
| Already saved      | Badge "Đã có trong Deck được chọn"; không tạo trùng     |

**Tap object →** MH-DICT-02 (Word Detail)

---

## 8. DICTIONARY & SEARCH Screens

### MH-DICT-01 — Search/Dictionary

| Thuộc tính | Mô tả                                        |
| ---------- | -------------------------------------------- |
| Actor      | Learner                                      |
| Feature    | F-DICT-01, F-DICT-02                         |
| BF         | BF-05                                        |
| FR         | FR-03.01, FR-03.02                           |
| Mục tiêu   | Tìm kiếm từ tiếng Anh hoặc tra cứu giọng nói |

**Dữ liệu hiển thị:**

- Search input (text) + microphone button (voice — Should)
- Recent searches (nếu có)
- Search suggestions/autocomplete
- Result list: word, nghĩa ngắn, IPA, saved state badge
- Empty state: "Không tìm thấy từ, kiểm tra lại chính tả"

**Tap result →** MH-DICT-02 (Word Detail)

---

### MH-DICT-02 — Word Detail

| Thuộc tính | Mô tả                                           |
| ---------- | ----------------------------------------------- |
| Actor      | Learner                                         |
| Feature    | F-DICT-03, F-DICT-04, F-DICT-05, F-DICT-07      |
| BF         | BF-05, BF-07                                    |
| FR         | FR-03.03 → FR-03.08, FR-04.01                   |
| Mục tiêu   | Xem chi tiết từ vựng và lưu/xóa khỏi vocabulary |

**Dữ liệu hiển thị:**

- **Từ tiếng Anh** (heading lớn)
- **Phiên âm IPA** (hoặc label "Chưa có phiên âm")
- **Nút phát âm** 🔊: nếu `audioUrl` có → phát URL; nếu `audioUrl = null` → TTS on-device (`expo-speech`). Nếu khả năng TTS không khả dụng → label “Chưa có” (ARC-12)
- **Nghĩa tiếng Việt** — nhóm theo POS nếu nhiều nghĩa:
    - _noun_ — nghĩa 1, nghĩa 2
    - _verb_ — nghĩa 3
- **Câu ví dụ** (nếu có)
- **Synonym / Antonym / Related words** (Could)
- **Ảnh crop** từ scan (nếu vào từ Detection Result)
- **Save / Remove button:**
    - Chưa lưu → "Lưu vào Deck" → chọn Deck hoặc dùng Deck gần nhất/mặc định → tạo Note + Card
    - Đã lưu trong Deck đang chọn → "Đã lưu ✓" + option "Xóa khỏi Deck"

**Trạng thái UI:**

- Field thiếu dữ liệu → label rõ "Chưa có dữ liệu phát âm", không để trống
- Từ đã có trong Deck đang chọn → "Từ đã có trong Deck được chọn"

---

### MH-DICT-03 — Voice Search (Should)

| Thuộc tính | Mô tả                             |
| ---------- | --------------------------------- |
| Actor      | Learner                           |
| Feature    | F-DICT-02                         |
| BF         | BF-05                             |
| FR         | FR-03.02                          |
| Mục tiêu   | Đọc tiếng Việt để tra cứu từ vựng |

**UX:** Có thể là overlay/modal trên MH-DICT-01 hoặc màn riêng.

- Nhấn microphone → `expo-speech-recognition` (STT on-device) bắt đầu lắng nghe tiếng Việt
- Chuyển giọng nói → text tiếng Việt (hiển thị ngay trên UI)
- Gửi text lên `/words/search` → backend reverse-lookup bảng Translation (tiếng Việt → Word). **Không** gọi STT API hay translate API cloud. (ARC-12)
- Hiển thị kết quả hoặc redirect MH-DICT-02

**Trạng thái UI:**

- Không nhận diện → "Không nhận diện được, vui lòng thử lại"
- Microphone permission denied → settings redirect
- STT không khả dụng trên thiết bị → bướt nút mic và hiển thị tooltip

---

## 9. TOPIC Screens

### MH-TOPIC-01 — Collections & Topics

| Thuộc tính | Mô tả                                     |
| ---------- | ----------------------------------------- |
| Actor      | Learner                                   |
| Feature    | F-TOPIC-01, F-TOPIC-02                    |
| BF         | BF-05                                     |
| FR         | FR-03 (topic browse)                      |
| Mục tiêu   | Duyệt bộ sưu tập và chủ đề học tập có sẵn |

**Dữ liệu hiển thị:**

- Danh sách Collections (icon, tên, số topics)
- Tap Collection → danh sách Topics (hỗ trợ phân cấp parent/child)
- Topic card: tên, số từ, progress badge (nếu đã học)

---

### MH-TOPIC-02 — Topic Items

| Thuộc tính | Mô tả                                              |
| ---------- | -------------------------------------------------- |
| Actor      | Learner                                            |
| Feature    | F-TOPIC-03, F-TOPIC-04                             |
| BF         | BF-05                                              |
| FR         | FR-04.01 (save from topic)                         |
| Mục tiêu   | Xem danh sách từ vựng trong chủ đề và lưu vào Deck |

**Dữ liệu hiển thị:**

- Topic title, description
- Danh sách TopicItems:
    - Từ vựng
    - Nghĩa tiếng Việt (từ EAV attributes)
    - Phiên âm, audio (nếu có)
    - Save button per item → lưu vào Deck gần nhất/mặc định, có option **Đổi Deck** trước khi xác nhận; tạo Note (source=TOPIC)
    - Trạng thái đã lưu trong Deck đang chọn
- Deck đích hiện tại: tên Deck + action **Đổi Deck**
- "Lưu tất cả" button → lưu các item chưa trùng vào Deck đang chọn; nếu có trùng, báo số từ bị bỏ qua

**Tap item →** MH-DICT-02 (Word Detail)

---

## 10. VOCABULARY Screens

### MH-VOCAB-01 — My Vocabulary (Deck List)

| Thuộc tính | Mô tả                                           |
| ---------- | ----------------------------------------------- |
| Actor      | Learner                                         |
| Feature    | F-VOCAB-01, F-VOCAB-08                          |
| BF         | BF-07                                           |
| FR         | FR-04.02                                        |
| Mục tiêu   | Xem danh sách Deck và tổng quan từ vựng cá nhân |

**Dữ liệu hiển thị:**

- Danh sách Decks:
    - Tên Deck
    - Note count
    - Template hiện tại (CLASSIC, LISTENING...)
    - Due count (Cards đến hạn)
- "Tạo Deck mới" button
- Empty state: "Chưa có Deck nào. Tạo Deck và bắt đầu lưu từ!"

**Tap Deck →** MH-VOCAB-02 (Deck Detail)

---

### MH-VOCAB-02 — Deck Detail

| Thuộc tính | Mô tả                                       |
| ---------- | ------------------------------------------- |
| Actor      | Learner                                     |
| Feature    | F-VOCAB-02 → F-VOCAB-07                     |
| BF         | BF-07                                       |
| FR         | FR-04.02 → FR-04.06                         |
| Mục tiêu   | Xem/quản lý Notes trong Deck và bắt đầu học |

**Dữ liệu hiển thị:**

- Deck name + template badge
- Filter/sort: UI state (new/learning/reviewing/mastered), ngày lưu, độ khó, due date
- Danh sách Notes:
    - Từ tiếng Anh
    - Nghĩa ngắn
    - Learning state badge (new/learning/reviewing/mastered) suy từ FSRS + interval
    - Source tag (SCAN/DICT/TOPIC) — Could
    - Swipe delete/archive
- **Action buttons:**
    - "Học Flashcard" → MH-LEARN-01
    - "Làm Quiz" → MH-LEARN-02
    - "Ôn SRS" → MH-LEARN-05
    - "Đổi Template" → template picker
- Empty state: "Chưa có từ nào. Tra cứu hoặc Scan để thêm từ mới!"

**Tap Note →** MH-DICT-02 (Word Detail)

---

## 11. LEARNING Screens

### MH-LEARN-01 — Flashcards Study Session

| Thuộc tính | Mô tả                                           |
| ---------- | ----------------------------------------------- |
| Actor      | Learner                                         |
| Feature    | F-FLASH-01 → F-FLASH-09                         |
| BF         | BF-08                                           |
| FR         | FR-05                                           |
| Mục tiêu   | Học từ bằng flashcard và đánh giá recall (FSRS) |

**Dữ liệu hiển thị:**

- Card render theo **CardTemplate config** của Deck:
    - **Front side:** fields theo template (VD: WORD + IPA cho CLASSIC)
    - **Back side:** fields theo template (VD: MEANING + POS + EXAMPLE + AUDIO)
    - Field thiếu dữ liệu → ẩn, layout tự điều chỉnh
- **Interaction:**
    - FLIP: tap để lật thẻ
    - TYPE_IN: input field → so khớp answer
    - TAP_TO_REVEAL: chạm từng phần lộ dần
- **FSRS rating buttons:** Again / Hard / Good / Easy (sau khi xem back)
- Progress bar (X/Y cards)
- Session summary khi hết Card: số thẻ, accuracy

**Empty state:** "Chưa có Card nào. Lưu thêm từ để bắt đầu học."

---

### MH-LEARN-02 — Quiz Setup

| Thuộc tính | Mô tả                                       |
| ---------- | ------------------------------------------- |
| Actor      | Learner                                     |
| Feature    | F-QUIZ-01                                   |
| BF         | BF-09                                       |
| FR         | FR-06.01                                    |
| Mục tiêu   | Cấu hình quiz: chọn Deck, loại quiz, số câu |

**Dữ liệu hiển thị:**

- Chọn Deck (source Notes)
- Quiz modes: Multiple choice / Matching / Fill blank
- Số câu hỏi (slider hoặc preset)
- Số từ khả dụng (disable nếu < min)
- Start Quiz CTA → MH-LEARN-03

**Empty state:** "Cần ít nhất X từ để tạo quiz. Lưu thêm từ!"

---

### MH-LEARN-03 — Quiz Play

| Thuộc tính | Mô tả                           |
| ---------- | ------------------------------- |
| Actor      | Learner                         |
| Feature    | F-QUIZ-02, F-QUIZ-03, F-QUIZ-04 |
| BF         | BF-09                           |
| FR         | FR-06.02 → FR-06.04             |
| Mục tiêu   | Trả lời câu hỏi quiz            |

**Dữ liệu hiển thị:**

- Câu hỏi (từ/nghĩa/audio tùy mode)
- Đáp án: MCQ options / Matching pairs / Fill blank input
- Progress: câu X / Y
- Timer (nếu có)
- Feedback đúng/sai per câu (tùy mode)
- Next / Submit button

---

### MH-LEARN-04 — Quiz Result

| Thuộc tính | Mô tả                                |
| ---------- | ------------------------------------ |
| Actor      | Learner                              |
| Feature    | F-QUIZ-05, F-QUIZ-07, F-QUIZ-08      |
| BF         | BF-09                                |
| FR         | FR-06.05 → FR-06.07                  |
| Mục tiêu   | Hiển thị kết quả quiz và phần thưởng |

**Dữ liệu hiển thị:**

- Score (điểm)
- Correct / Wrong count
- Accuracy %
- Duration
- Danh sách câu sai (từ + đáp án đúng)
- XP / Coin reward (M4)
- CTA: "Thử lại" / "Ôn từ sai" / "Về Home"

---

### MH-LEARN-05 — SRS Review Session

| Thuộc tính | Mô tả                       |
| ---------- | --------------------------- |
| Actor      | Learner                     |
| Feature    | F-SRS-01 → F-SRS-07         |
| BF         | BF-10                       |
| FR         | FR-07                       |
| Mục tiêu   | Ôn tập từ đến hạn theo FSRS |

**Dữ liệu hiển thị:**

- Due count + Overdue count (header)
- Card render tương tự MH-LEARN-01 (theo template Deck)
- FSRS rating: Again / Hard / Good / Easy
- Interval preview (hiển thị "Xem lại sau X ngày" cho mỗi rating)
- Progress bar
- Review summary khi hoàn thành: số Card ôn, accuracy, time

**Empty state:** "Bạn đã ôn xong hôm nay! 🎉" hoặc "Không có từ đến hạn."

---

### MH-LEARN-06 — Template Management (Should)

| Thuộc tính | Mô tả                               |
| ---------- | ----------------------------------- |
| Actor      | Learner                             |
| Feature    | F-FLASH-03, F-FLASH-09, F-FLASH-11  |
| BF         | BF-08                               |
| FR         | FR-05.03                            |
| Mục tiêu   | Quản lý và tạo Custom Card Template |

**Dữ liệu hiển thị:**

- System templates (read-only): CLASSIC, REVERSE, LISTENING, IMAGE_VOCAB, SPELLING, CONTEXT
- Custom templates (CRUD): tên, interaction type, field count
- "Tạo Template mới" → template builder:
    - Chọn base layout (1-col, 2-col, image-top, audio-center)
    - Cấu hình Front side: chọn/bật tắt fields, sắp xếp, đánh dấu primary
    - Cấu hình Back side: tương tự
    - Chọn Interaction type: Flip / Type-in / Tap-to-reveal
    - Preview (dùng Note mẫu)
    - Save

---

## 12. STATS & PROGRESS Screens

### MH-STATS-01 — Stats/Progress

| Thuộc tính | Mô tả                             |
| ---------- | --------------------------------- |
| Actor      | Learner                           |
| Feature    | F-PROG-01 → F-PROG-05             |
| BF         | BF-11                             |
| FR         | FR-08                             |
| Mục tiêu   | Thống kê học tập chi tiết cá nhân |

**Dữ liệu hiển thị:**

- Words: saved / learned / reviewing / mastered theo learning-state map (progress bars)
- Streak: current + longest
- Accuracy: quiz + review tổng hợp
- Activity heatmap / chart: daily / weekly / monthly
- Review count, quiz attempts
- Daily goal progress (Could)

---

### MH-STATS-02 — Level Progress

| Thuộc tính | Mô tả                                      |
| ---------- | ------------------------------------------ |
| Actor      | Learner                                    |
| Feature    | F-GAME-01, F-PROG-01                       |
| BF         | BF-11, BF-12                               |
| FR         | FR-08, FR-09.01                            |
| Mục tiêu   | Hiển thị cấp độ, XP và mốc level tiếp theo |

**Dữ liệu hiển thị:**

- Current level
- XP hiện tại / XP cần cho level tiếp
- XP progress bar
- Recent XP events (source + amount)
- Level benefits (nếu có)

---

## 13. GAME Screens

### MH-GAME-01 — Leaderboard

| Thuộc tính | Mô tả                                    |
| ---------- | ---------------------------------------- |
| Actor      | Learner                                  |
| Feature    | F-GAME-06                                |
| BF         | BF-11                                    |
| FR         | FR-09.05                                 |
| Mục tiêu   | Xếp hạng theo XP / streak / điểm học tập |

**Dữ liệu hiển thị:**

- My rank + score
- Top N users (avatar, name, score)
- Filter: period (chỉ dùng weekly cho MVP), type (chỉ dùng XP)
- Scroll to "Your position"

---

### MH-GAME-02 — Missions

| Thuộc tính | Mô tả                                 |
| ---------- | ------------------------------------- |
| Actor      | Learner                               |
| Feature    | F-GAME-03, F-GAME-04, F-GAME-10       |
| BF         | BF-12                                 |
| FR         | FR-09.03                              |
| Mục tiêu   | Xem nhiệm vụ ngày/tuần và nhận thưởng |

**Dữ liệu hiển thị:**

- **Daily missions** (3–5 missions): tên, progress bar (VD: 7/10), reward (coin + XP)
- **Weekly stamps** (hoàn thành all daily → stamp → rương tuần)
- Claim button (COMPLETED → CLAIMED)
- Go-to-task CTA (deep link tới hoạt động liên quan)
- Reset countdown (thời gian đến 00:00)

**Detail:** Xem [daily_mission.md](./daily_mission.md)

---

### MH-GAME-03 — Achievements/Badges

| Thuộc tính | Mô tả                                    |
| ---------- | ---------------------------------------- |
| Actor      | Learner                                  |
| Feature    | F-GAME-05                                |
| BF         | BF-12                                    |
| FR         | FR-09.04                                 |
| Mục tiêu   | Xem huy hiệu đã đạt và điều kiện mở khóa |

**Dữ liệu hiển thị:**

- Badge grid: icon, name, locked/unlocked state
- Tap badge → detail: điều kiện, date earned, rarity
- Progress towards locked badges

---

## 14. ECONOMY Screens

### MH-ECONOMY-01 — Wallet

| Thuộc tính | Mô tả                                    |
| ---------- | ---------------------------------------- |
| Actor      | Learner                                  |
| Feature    | F-GAME-02                                |
| BF         | BF-12                                    |
| FR         | FR-09.02                                 |
| Mục tiêu   | Hiển thị số dư Coin và lịch sử giao dịch |

**Dữ liệu:** Coin balance, XP summary, CoinTransaction history (earn/spend), source labels.

---

### MH-ECONOMY-02 — Shop

| Thuộc tính | Mô tả                  |
| ---------- | ---------------------- |
| Actor      | Learner                |
| Feature    | F-GAME-07              |
| BF         | BF-12                  |
| FR         | FR-09.06               |
| Mục tiêu   | Mua vật phẩm bằng Coin |

**Dữ liệu hiển thị:**

- ShopItem list: icon, name, price, category (theme/avatar frame/booster)
- Coin balance (header)
- Owned badge (nếu đã sở hữu)
- Buy button (disabled nếu balance < price)
- Confirm modal: "Mua X với Y Coin?"

---

### MH-ECONOMY-03 — Inventory/My Items

| Thuộc tính | Mô tả                             |
| ---------- | --------------------------------- |
| Actor      | Learner                           |
| Feature    | F-GAME-08                         |
| BF         | BF-12                             |
| FR         | FR-09.07                          |
| Mục tiêu   | Xem và áp dụng vật phẩm đã sở hữu |

**Dữ liệu:** UserItem list, active/equipped state, expiry (nếu có), Apply/Remove CTA.

> **Ghi chú:** MVP không xử lý thanh toán tiền thật. Premium screen (nếu có) chỉ mô tả quyền lợi, không tích hợp payment gateway.

---

## 15. PROFILE Screens

### MH-PROFILE-01 — Profile

| Thuộc tính | Mô tả                                      |
| ---------- | ------------------------------------------ |
| Actor      | Learner                                    |
| Feature    | F-AUTH-07, F-PROG-01, F-GAME-05            |
| BF         | BF-04, BF-11                               |
| FR         | FR-01.07, FR-08.01                         |
| Mục tiêu   | Hồ sơ cá nhân, summary học tập và shortcut |

**Dữ liệu hiển thị:**

- Avatar (+ avatar frame nếu equip)
- Display name, email
- Streak, learned words count, level/XP
- Badges nổi bật (3–5 badges)
- Shortcut: Edit Profile, Settings, Achievements, Inventory, Wallet

---

### MH-PROFILE-02 — Edit Profile

| Thuộc tính | Mô tả                                |
| ---------- | ------------------------------------ |
| Actor      | Learner                              |
| Feature    | F-AUTH-07, F-AUTH-08, F-STOR-04      |
| BF         | BF-04                                |
| FR         | FR-01.07, FR-11.01                   |
| Mục tiêu   | Cập nhật thông tin cá nhân và avatar |

**Dữ liệu/form:**

- Avatar picker (camera/gallery → presigned upload ≤ 5MB)
- Display name input
- Save / Cancel buttons

---

### MH-PROFILE-03 — Settings

| Thuộc tính | Mô tả                                     |
| ---------- | ----------------------------------------- |
| Actor      | Learner                                   |
| Feature    | F-AUTH-06, F-AUTH-09, F-NOTIF-05          |
| BF         | BF-02, BF-13                              |
| FR         | FR-01.06, FR-01.08, FR-10.03              |
| Mục tiêu   | Cài đặt tài khoản, thông báo và đăng xuất |

**Dữ liệu hiển thị:**

- **Account:** Change password, Biometric login toggle
- **Notifications:** Push on/off, quiet hours (nếu hỗ trợ)
- **Learning:** Daily goal (Could), SRS reminder time
- **App:** Language, Theme (nếu có shop theme)
- **Logout** button
- App version

---

### MH-PROFILE-04 — Notifications

| Thuộc tính | Mô tả                                      |
| ---------- | ------------------------------------------ |
| Actor      | Learner                                    |
| Feature    | F-NOTIF-01, F-NOTIF-02, F-NOTIF-03         |
| BF         | BF-13                                      |
| FR         | FR-10                                      |
| Mục tiêu   | Xem danh sách thông báo và đánh dấu đã đọc |

**Dữ liệu hiển thị:**

- Notification list:
    - Icon theo type (SRS reminder, Badge, Mission, System)
    - Title + body
    - Read/unread state (bold/dimmed)
    - Created time (relative: "2 giờ trước")
    - Tap → deep link tới target screen
- Mark all as read button
- Empty state: "Chưa có thông báo nào"

---

## 16. SYSTEM Screens & UI States

### MH-SYSTEM-01 — Design System

| Thuộc tính | Mô tả                                               |
| ---------- | --------------------------------------------------- |
| Actor      | Designer / Developer                                |
| Mục tiêu   | Chuẩn hóa component và visual language cho toàn app |

**Nội dung cần có:**

- Colors (primary, secondary, accent, semantic: success/warning/error)
- Typography (font family, sizes, weights)
- Buttons (primary, secondary, ghost, disabled, loading)
- Inputs (text, password, OTP, search, error state)
- Cards (vocabulary card, quiz card, mission card)
- Badges/chips (state badges, source tags, reliability badge)
- Navigation / Tab bar
- Gamification components: XP bar, Coin badge, Streak flame, Level badge, Progress bar
- Leaderboard row
- Flashcard component (front/back, flip animation)
- Icon usage

---

### MH-SYSTEM-02 — Empty/Error/Loading States

| State                | Áp dụng cho                  | Nội dung cần có                                         |
| -------------------- | ---------------------------- | ------------------------------------------------------- |
| **Loading**          | API call, upload, scan, quiz | Spinner/skeleton + text mô tả (VD: "Đang nhận diện...") |
| **Empty vocabulary** | Vocab / Deck Detail          | Illustration + "Tra cứu / Scan để lưu từ đầu tiên"      |
| **Empty search**     | Dictionary Search            | "Không tìm thấy từ, kiểm tra lại chính tả"              |
| **Empty review**     | SRS / Home                   | "Bạn đã ôn xong hôm nay! 🎉"                            |
| **No object**        | Detection Result             | "Không nhận diện được" + CTA "Thử ảnh khác"             |
| **Low reliability**   | Detection Result             | "Không tìm thấy vật thể có độ tin cậy cao" + CTA retry                   |
| **Network error**    | Toàn app                     | Illustration + "Không có kết nối" + Retry button        |
| **Auth expired**     | Toàn app                     | Auto refresh hoặc redirect Login                        |
| **Permission deny**  | Camera / Gallery / Mic       | Hướng dẫn bật permission trong Settings                 |
| **Quiz not enough**  | Quiz Setup                   | "Cần thêm X từ để tạo quiz" + CTA lưu từ                |
| **AI timeout**       | Detection                    | "Xử lý quá lâu" + CTA "Thử lại"                         |

---

## 19. Ma trận MH ↔ BF (Luồng nghiệp vụ)

| Màn hình                      | Luồng nghiệp vụ            |
| ----------------------------- | -------------------------- |
| MH-ONBOARD-01                 | BF-01, BF-02               |
| MH-AUTH-01 → MH-AUTH-05       | BF-01, BF-02, BF-03        |
| MH-MAIN-01                    | BF-10, BF-11, BF-12        |
| MH-MAIN-02                    | BF-07, BF-08, BF-09, BF-10 |
| MH-CAMERA-01, MH-CAMERA-02    | BF-06                      |
| MH-DICT-01 → MH-DICT-03       | BF-05                      |
| MH-TOPIC-01, MH-TOPIC-02      | BF-05                      |
| MH-VOCAB-01, MH-VOCAB-02      | BF-07                      |
| MH-LEARN-01                   | BF-08                      |
| MH-LEARN-02 → MH-LEARN-04     | BF-09                      |
| MH-LEARN-05                   | BF-10                      |
| MH-LEARN-06                   | BF-08                      |
| MH-STATS-01, MH-STATS-02      | BF-11                      |
| MH-GAME-01 → MH-GAME-03       | BF-11, BF-12               |
| MH-ECONOMY-01 → MH-ECONOMY-03 | BF-12                      |
| MH-PROFILE-01, MH-PROFILE-02  | BF-04                      |
| MH-PROFILE-03                 | BF-02, BF-13               |
| MH-PROFILE-04                 | BF-13                      |

---

## 20. Ma trận MH ↔ Feature Area

| Màn hình                   | Feature Areas                           |
| -------------------------- | --------------------------------------- |
| MH-ONBOARD-01, AUTH-01→05  | AUTH                                    |
| MH-MAIN-01                 | PROGRESS, SRS, FLASH, QUIZ, GAME        |
| MH-MAIN-02                 | VOCAB, FLASH, QUIZ, SRS, TOPIC          |
| MH-CAMERA-01, CAMERA-02    | RECOG, DICT, VOCAB, STORAGE             |
| MH-DICT-01 → DICT-03       | DICT                                    |
| MH-TOPIC-01, TOPIC-02      | TOPIC, VOCAB                            |
| MH-VOCAB-01, VOCAB-02      | VOCAB, FLASH, QUIZ                      |
| MH-LEARN-01                | FLASH, SRS, PROGRESS                    |
| MH-LEARN-02 → LEARN-04     | QUIZ, PROGRESS, GAME                    |
| MH-LEARN-05                | SRS, FLASH, PROGRESS                    |
| MH-LEARN-06                | FLASH                                   |
| MH-STATS-01, STATS-02      | PROGRESS, GAME                          |
| MH-GAME-01 → GAME-03       | GAME                                    |
| MH-ECONOMY-01 → ECONOMY-03 | GAME, SHOP                              |
| MH-PROFILE-01 → PROFILE-04 | AUTH, PROFILE, STORAGE, NOTIF, PROGRESS |
| MH-SYSTEM-01, SYSTEM-02    | SYSTEM (cross-cutting)                  |

---

## 21. Ưu tiên màn hình theo Milestone

| Milestone                          | Màn hình ưu tiên                                                                                                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **M1** — Core Auth & Vocabulary    | ONBOARD-01, AUTH-01→05, MAIN-01, MAIN-02, DICT-01, DICT-02, TOPIC-01, TOPIC-02, VOCAB-01, VOCAB-02, LEARN-01 (basic), PROFILE-01, PROFILE-02, PROFILE-03, SYSTEM-01, SYSTEM-02 |
| **M2** — Camera Recognition        | CAMERA-01, CAMERA-02, DICT-02 (từ scan), detection states                                                                                                                      |
| **M3** — Learning Engine           | LEARN-01 (template), LEARN-02→05, LEARN-06, STATS-01, STATS-02, PROFILE-04                                                                                                     |
| **M4** — Gamification & Production | GAME-01→03, ECONOMY-01→03, MAIN-01 (mission widget), Level Progress                                                                                                            |
| **Admin (web riêng)**              | User mgmt, Dict mgmt, Templates, Game config, Stats — không mã MH mobile                                                                                                       |

---

## 22. Tổng quan số lượng

| Trạng thái           | Số màn hình |
| -------------------- | ----------- |
| Đã thiết kế (Figma)  | 23          |
| Cần bổ sung / làm rõ | 7           |
| **Tổng**             | **30**      |

---

## 23. Checklist

- [x] Bộ MH bao phủ M1–M4 theo phạm vi specs.
- [x] MH-AUTH-04 (OTP) và MH-AUTH-05 (Reset Password) đã bổ sung.
- [x] MH-DICT-02 (Word Detail) đã bổ sung — dùng chung cho search/detection/topic/vocabulary.
- [x] MH-TOPIC-01, MH-TOPIC-02 đã bổ sung — duyệt Collection/Topic/TopicItem.
- [x] MH-VOCAB-02 (Deck Detail) đã bổ sung — quản lý Notes trong Deck.
- [x] MH-LEARN-06 (Template Management)
- [x] Detection Result thể hiện đủ states: success, no-object, low-reliability, dictionary miss, AI error.
- [x] Vocabulary screens = Deck/Note/Card, không `SavedWord`/`UserWord`.
- [x] AI pipeline = Florence-2 + SAM + CLIP, không YOLO.
- [x] SRS = FSRS trên Card.
- [x] Actor: Guest, Learner, Admin (CMS web riêng).
- [x] Ma trận MH ↔ BF và MH ↔ Feature Area đầy đủ.
- [x] Milestone mapping rõ ràng M1–M4.
- [x] FR IDs khớp specs (Game=FR-09, Noti=FR-10, Storage=FR-11, OpenAPI=FR-12, Admin=FR-13).
- [x] Premium không xử lý thanh toán tiền thật trong MVP.
- [x] Design System cần component cho gamification (XP, coin, badge, streak, progress bar).
