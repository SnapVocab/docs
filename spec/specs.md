# Đặc tả Yêu cầu Hệ thống — SnapVocab

> Tài liệu tổng quan yêu cầu phần mềm (SRS) cho ứng dụng di động hỗ trợ học từ vựng tiếng Anh thông qua nhận diện hình ảnh.

---

## 1. Mục tiêu dự án

SnapVocab hướng đến xây dựng ứng dụng di động giúp người học tiếng Anh ghi nhớ từ vựng từ các vật thể trong đời sống thực tế. Người dùng có thể chụp ảnh hoặc tải ảnh lên, hệ thống sử dụng pipeline nhận diện từ vựng mở (open-vocabulary) dựa trên mô hình Florence-2 kết hợp SAM (cắt nền vật thể) và CLIP (xác thực ngữ nghĩa), chạy ở chế độ zero-shot. Mỗi vật thể phát hiện được trả về kèm một từ tiếng Anh được bảo đảm thuộc từ điển, sau đó ánh xạ sang nghĩa tiếng Việt, phiên âm và phát âm. Khác với các bộ phát hiện tập lớp đóng (như YOLO chỉ nhận diện 80 lớp COCO), pipeline này gọi tên được cả vật thể ngoài mọi danh sách lớp định sẵn — đúng nguồn từ mới mà người học cần.

Mục tiêu chính:

- **Giảm ma sát khi học từ mới**: người học có thể biến ảnh chụp hằng ngày thành nguồn từ vựng học tập.
- **Cá nhân hóa danh sách học**: từ vựng được lưu vào danh sách cá nhân để học lại bằng flashcard, quiz và SRS.
- **Hỗ trợ ghi nhớ dài hạn**: hệ thống đề xuất lịch ôn tập theo thuật toán Spaced Repetition System.
- **Tăng động lực học tập**: tích hợp tiến độ, nhiệm vụ, huy hiệu, điểm kinh nghiệm, coin, bảng xếp hạng và cửa hàng vật phẩm.
- **Tách biệt kiến trúc xử lý**: mobile app, backend API, AI service, database, cache và object storage được thiết kế thành các thành phần rõ ràng để dễ mở rộng.

---

## 2. Phạm vi hệ thống

### Trong phạm vi (In Scope)

| #     | Nhóm chức năng                        | Mô tả                                                                                                        |
| ----- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **A** | **Chức năng Người dùng**              |                                                                                                              |
| 1     | Xác thực tài khoản                    | Đăng ký, đăng nhập (mật khẩu, vân tay/sinh trắc học), xác thực email/OTP, làm mới token, khôi phục mật khẩu  |
| 2     | Hồ sơ người dùng                      | Xem/cập nhật thông tin cá nhân, avatar, thông tin học tập cơ bản                                             |
| 3     | Nhận diện ảnh                         | Chụp ảnh hoặc tải ảnh từ thư viện để gửi xử lý bằng AI                                                       |
| 4     | Bong bóng chụp ảnh (Android)          | Lớp phủ nút nổi (floating widget) để tự động chụp và quét từ vựng khi đang lướt app khác                     |
| 5     | Thông tin từ vựng                     | Hiển thị từ tiếng Anh, nghĩa tiếng Việt, phiên âm, phát âm và thông tin học tập liên quan                    |
| 6     | Tra cứu từ điển & Giọng nói           | Tìm kiếm từ vựng bằng văn bản hoặc đọc giọng nói tiếng Việt (Voice-to-Text) để hệ thống dịch và tra cứu      |
| 7     | Học theo Chủ đề                       | Xem và học từ vựng được phân loại theo các bộ sưu tập (Collections) và chủ đề (Topics)                       |
| 8     | Từ vựng cá nhân                       | Lưu, xem, xóa, lọc và quản lý danh sách từ người học muốn học                                                |
| 9     | Flashcard & Custom Card               | Học từ bằng thẻ, hỗ trợ hệ thống Custom Template (tùy chỉnh hiển thị ảnh, âm thanh, gõ từ) quản lý theo Deck |
| 10    | Quiz                                  | Luyện tập bằng câu hỏi trắc nghiệm, ghép nghĩa, điền từ hoặc các dạng kiểm tra phù hợp                       |
| 11    | SRS Review                            | Tạo hàng đợi ôn tập theo thời điểm đến hạn và cập nhật lịch ôn dựa trên kết quả nhớ                          |
| 12    | Tiến độ học tập                       | Theo dõi số từ đã học, số lượt ôn, streak, độ chính xác và kết quả quiz                                      |
| 13    | Bảng xếp hạng                         | Xếp hạng cá nhân dựa trên XP tuần (Weekly XP)                                                                |
| 14    | Gamification                          | Nhiệm vụ, huy hiệu, điểm kinh nghiệm, coin, vật phẩm và cửa hàng trong ứng dụng                              |
| 15    | Hệ thống thông báo                    | Nhận thông báo đẩy (Push Notification) nhắc nhở ôn tập và thông báo trong ứng dụng (In-app)                  |
| **B** | **Chức năng Admin (CMS / Dashboard)** |                                                                                                              |
| 16    | Quản lý người dùng                    | Xem danh sách, tìm kiếm, xem chi tiết tiến độ học tập, khóa/mở khóa tài khoản và reset mật khẩu Learner      |
| 17    | Quản lý từ điển & chủ đề              | Thêm, sửa, xóa (CRUD) từ vựng, quản lý cấu trúc chủ đề (Collections/Topics), duyệt/import dữ liệu từ điển    |
| 18    | Quản lý Template thẻ học              | Quản lý (thêm/sửa/xóa) các mẫu thẻ học hệ thống (System Templates) cung cấp cho người học                    |
| 19    | Quản lý Gamification                  | Quản lý nhiệm vụ (Missions), vật phẩm trong Shop (Items), huy hiệu (Badges) và cấu hình điểm thưởng          |
| 20    | Quản lý Feedback & Lỗi                | Xem các báo cáo lỗi từ người dùng (nhận diện sai, từ vựng sai) và theo dõi log lỗi cơ bản                    |
| 21    | Thống kê & Báo cáo                    | Biểu đồ lượng người dùng active, số lượt dùng AI service, mức tiêu thụ lưu trữ (Storage), tổng quan hệ thống |
| **C** | **Xử lý Hệ thống / Kỹ thuật**         |                                                                                                              |
| 22    | Ánh xạ từ vựng                        | Chuyển nhãn đối tượng nhận diện được thành từ tiếng Anh tương ứng trong cơ sở dữ liệu từ vựng                |
| 23    | Lưu trữ media                         | Avatar, ảnh quét và tài nguyên vật phẩm qua object storage                                                   |
| 24    | Tài liệu hóa API                      | Swagger/OpenAPI phục vụ kiểm thử và tích hợp giữa mobile, backend và AI service                              |

### Ngoài phạm vi (Out of Scope)

| #   | Hạng mục                                     | Ghi chú                                                                                                                                                                 |
| --- | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Fine-tune mô hình AI trong MVP               | MVP dùng Florence-2 zero-shot (không huấn luyện lại); fine-tune LoRA trên COCO+Open Images là hướng mở rộng đã có thiết kế thí nghiệm (kèm phép đo vocabulary collapse) |
| 2   | Nhận diện AI offline hoàn toàn trên thiết bị | Xử lý chính được thực hiện qua AI service độc lập để giảm yêu cầu phần cứng mobile                                                                                      |
| 3   | Thanh toán tiền thật                         | Cửa hàng chỉ dùng coin/điểm thưởng trong ứng dụng, không xử lý ví điện tử hoặc giao dịch tiền thật                                                                      |
| 4   | Chấm điểm phát âm nâng cao                   | Hệ thống phát âm từ vựng; đánh giá giọng nói/ngữ âm của người học là chức năng mở rộng                                                                                  |
| 5   | Từ điển học thuật hoàn chỉnh                 | Dữ liệu từ điển dùng để tra cứu học tập, không thay thế từ điển chuyên ngành đầy đủ                                                                                     |

### Quyết định canonical (bắt buộc đồng bộ mọi tài liệu)

| Chủ đề          | Quyết định                                      | Ghi chú                                                                                           |
| --------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Source of truth | [specs.md](./specs.md)                          | Mọi BF/SS/MH/SA/DB/API phải truy vết về file này                                                  |
| AI pipeline     | Florence-2 + SAM + CLIP (F2-v13)                | **Không** dùng YOLO làm model chính trong docs                                                    |
| Actor           | Guest, Learner, Admin                           | Admin = CMS                                                                                       |
| Learning domain | `Deck` → `Note` → `Card` + `ReviewLog`          | “Saved vocabulary” = Note trong Deck của Learner; **không** entity song song `SavedWord/UserWord` |
| SRS             | FSRS trên `Card`                                | State/due/stability/difficulty nằm ở Card                                                         |
| Milestone       | 4 mốc (Auth+Dict → Scan → Learning → Game+Prod) | Theo thứ tự ưu tiên triển khai hiện tại                                                           |
| FR IDs          | FR-01 … FR-14 như §5                            | Không dùng map FR cũ                                                                              |

### MVP Cut & Milestones

Tài liệu mô tả phạm vi đầy đủ của SnapVocab. Khi triển khai, hệ thống nên được chia theo milestone để ưu tiên luồng học cốt lõi: đăng nhập → tra cứu/nhận diện → lưu từ → học flashcard/quiz → ôn SRS.

#### Milestone 1 — Core Auth & Vocabulary Lookup

Mục tiêu: hoàn thiện nền tảng tài khoản, hồ sơ và tra cứu/lưu từ vựng.

| Nhóm             | Bao gồm                                                                                                     |
| ---------------- | ----------------------------------------------------------------------------------------------------------- |
| Auth             | Đăng ký, đăng nhập (mật khẩu, vân tay/sinh trắc học), refresh token, xác thực email/OTP, khôi phục mật khẩu |
| Profile          | Xem/cập nhật hồ sơ cá nhân, avatar                                                                          |
| Dictionary       | Tra cứu từ vựng bằng văn bản hoặc giọng nói tiếng Việt, xem nghĩa, phiên âm, phát âm                        |
| Topic Learning   | Duyệt và học từ vựng theo các bộ sưu tập (Collections) và chủ đề (Topics) có sẵn                            |
| Saved Vocabulary | Lưu/xóa từ vào danh sách cá nhân, xem danh sách đã lưu                                                      |
| Flashcard cơ bản | Sinh flashcard từ saved words, flip card, đánh giá FSRS 4 mức (Again/Hard/Good/Easy); template CLASSIC hard-code, chưa có UI cấu hình template |

Done criteria:

1. Guest đăng ký, xác thực và đăng nhập được vào hệ thống.
2. Learner xem và cập nhật được hồ sơ cá nhân.
3. Learner tra cứu được từ vựng và xem thông tin nghĩa tiếng Việt, phiên âm, phát âm nếu có.
4. Learner lưu được từ vào danh sách học cá nhân và mở lại danh sách này.
5. Learner học được saved words bằng flashcard template CLASSIC (flip), đánh giá FSRS 4 mức (Again/Hard/Good/Easy) sau mỗi thẻ, Card được cập nhật dueAt/state theo FSRS.

#### Milestone 2 — Camera/Object Recognition MVP

Mục tiêu: chứng minh luồng scan-to-vocabulary hoạt động end-to-end.

| Nhóm               | Bao gồm                                                                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Image Input        | Chụp ảnh bằng camera hoặc chọn ảnh từ thư viện                                                                                                  |
| Upload/Storage     | Gửi ảnh tới backend hoặc upload qua presigned URL nếu cần lưu trữ                                                                               |
| AI Detection       | Recognition worker gọi FastAPI AI service qua hàng đợi, chạy pipeline Florence-2 + SAM + CLIP (chế độ sản phẩm, phiên bản F2-v13)                |
| Recognition Result | Nhận danh sách vật thể: nhãn từ vựng mở đã qua chuỗi lọc từ điển, điểm tin cậy, bounding box và (tùy chọn) ảnh cắt nền trong suốt cho flashcard |
| Word Mapping       | Ánh xạ label sang từ vựng trong database                                                                                                        |
| Result UI          | Hiển thị danh sách đối tượng/từ vựng kèm nghĩa, phiên âm, phát âm                                                                               |

Done criteria:

1. Learner chụp hoặc chọn ảnh và gửi xử lý thành công.
2. AI service trả về danh sách đối tượng gồm nhãn, độ tin cậy và bounding box.
3. Backend ánh xạ được ít nhất một nhãn phổ biến sang từ vựng trong database.
4. Mobile hiển thị kết quả nhận diện kèm thông tin từ vựng học tập.
5. Learner lưu được từ sinh ra từ ảnh vào danh sách học cá nhân.
6. Trường hợp không nhận diện được hoặc toàn bộ vật thể có độ tin cậy thấp (Medium/Low source) được xử lý bằng thông báo rõ ràng, không làm app crash.

#### Milestone 3 — Learning Engine

Mục tiêu: hoàn thiện cơ chế học, kiểm tra và ôn tập dài hạn.

| Nhóm         | Bao gồm                                                              |
| ------------ | -------------------------------------------------------------------- |
| Quiz         | Trắc nghiệm nghĩa, chọn từ đúng, ghép từ-nghĩa, điền từ nếu phù hợp  |
| Quiz Attempt | Lưu điểm, số câu đúng/sai, thời gian làm và lịch sử attempt          |
| SRS          | Tính state, dueAt, stability, difficulty theo FSRS cho từng Card     |
| Review Queue | Danh sách từ đến hạn ôn tập theo ngày                                |
| Progress     | Streak, số từ đã học, accuracy, số lượt ôn, thống kê tuần/tháng      |
| Notification | Gửi thông báo đẩy (Push) nhắc nhở ôn tập SRS và thông báo in-app     |

Done criteria:

1. Learner làm quiz từ danh sách từ cá nhân và nhận điểm sau khi hoàn thành.
2. Kết quả quiz/review được ghi nhận vào tiến độ học tập.
3. Hệ thống tạo được daily review queue dựa trên lịch SRS.
4. Lịch ôn của một Card thay đổi sau khi Learner đánh giá mức độ nhớ.
5. Màn hình tiến độ phản ánh đúng dữ liệu sau các hoạt động học.

#### Milestone 4 — Gamification & Production Readiness

Mục tiêu: tăng động lực học tập và chuẩn bị vận hành production.

| Nhóm               | Bao gồm                                                                     |
| ------------------ | --------------------------------------------------------------------------- |
| Mission            | Nhiệm vụ ngày/tuần/thành tựu học tập                                        |
| Reward             | XP, coin, badge được trao theo rule rõ ràng                                 |
| Shop               | Cửa hàng vật phẩm dùng coin, áp dụng item trong app                         |
| Leaderboard        | Xếp hạng cá nhân, có cache bằng Redis nếu cần                               |
| Production Storage | Cloudflare R2/S3-compatible private bucket, lifecycle cleanup               |
| Hardening          | Security config, validation upload, logging, monitoring, OpenAPI hoàn chỉnh |

Done criteria:

1. Hoạt động học tập hợp lệ cộng XP theo bảng cơ sở (review = 2 XP, quiz đúng = 1 XP/câu, lưu từ = 1 XP).
2. Learner nhận huy hiệu khi đạt điều kiện cụ thể.
3. Learner mua và sử dụng được vật phẩm bằng coin (nếu Shop được triển khai).
4. Leaderboard cập nhật đúng và có chiến lược cache phù hợp.
5. Media production được lưu qua Cloudflare R2 hoặc S3-compatible storage an toàn.
6. API có tài liệu Swagger/OpenAPI đủ cho mobile và AI service tích hợp.

---

## 3. Đối tượng sử dụng (Actors)

| Actor       | Vai trò                      | Quyền hạn chính                                                                                                        |
| ----------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Guest**   | Người dùng chưa đăng nhập    | Xem onboarding/welcome, đăng ký tài khoản, đăng nhập, khôi phục mật khẩu, xác thực email/OTP                           |
| **Learner** | Người học chính của ứng dụng | Quản lý hồ sơ, scan ảnh, tra cứu từ vựng, lưu từ, học flashcard, làm quiz, ôn SRS, xem tiến độ, nhận reward, dùng shop |
| **Admin**   | Quản trị viên hệ thống       | Quản lý tài khoản người dùng, quản lý dữ liệu từ vựng/từ điển, xem báo cáo thống kê                                    |

Business rules:

- Guest không được truy cập dữ liệu học tập cá nhân hoặc API yêu cầu định danh.
- Learner chỉ xem và chỉnh sửa dữ liệu cá nhân của chính mình.
- Admin có quyền truy cập vào CMS/Dashboard để quản lý dữ liệu hệ thống, nhưng không được can thiệp vào tiến độ học tập cá nhân cụ thể của Learner.
- Các chức năng hạ tầng sâu như cấu hình storage hoặc server monitoring được xem là tác vụ kỹ thuật, không thuộc phạm vi Dashboard của Admin thông thường.

---

## 4. Tổng quan kiến trúc hệ thống

### 4.1 Thành phần hệ thống

| Thành phần        | Công nghệ                                                  | Trách nhiệm                                                                                                                         |
| ----------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Mobile App        | React Native/Expo, TypeScript, Expo Router                 | Giao diện người dùng, camera/gallery, học flashcard/quiz/SRS, gọi backend API                                                       |
| Backend API       | Java Spring Boot REST API                                  | Xử lý nghiệp vụ, xác thực, quản lý user/word/learning/gamification, tích hợp storage và AI service                                  |
| Authentication    | Spring Security + JWT + Refresh Token                      | Bảo vệ API, phân quyền actor, quản lý phiên đăng nhập                                                                               |
| Database          | MySQL/MariaDB + JPA/Hibernate                              | Lưu user, từ vựng, nghĩa, phiên âm, tiến độ, reward và cấu hình nghiệp vụ                                                           |
| Cache             | Redis/Redisson                                             | Cache dữ liệu truy cập thường xuyên, leaderboard, điểm kinh nghiệm (không dùng cho session)                               |
| Object Storage    | Cloudflare R2 hoặc S3-compatible storage/MinIO             | Lưu ảnh scan, avatar và tài nguyên vật phẩm                                                                                         |
| AI Service        | Python FastAPI + Florence-2-large (zero-shot) + SAM + CLIP | Nhận ảnh, chạy pipeline nhận diện từ vựng mở, trả nhãn (bảo đảm thuộc từ điển), điểm tin cậy, bounding box và ảnh cắt nền flashcard |
| Dictionary Source | SQLite/minhqnd dictionary import                           | Cung cấp dữ liệu từ vựng Anh-Việt, định nghĩa, phiên âm, bản dịch                                                                   |
| API Docs          | Swagger/OpenAPI                                            | Tài liệu hóa backend API cho mobile và AI service tích hợp                                                                          |
| UI Design         | Figma                                                      | Thiết kế giao diện, prototype và thống nhất trải nghiệm mobile                                                                      |

### 4.2 Luồng scan-to-learn

1. Learner mở tab camera hoặc chức năng upload ảnh trên mobile app.
2. Learner chụp ảnh mới hoặc chọn ảnh từ thư viện.
3. Mobile gửi ảnh tới backend; backend có thể nhận trực tiếp file hoặc cấp presigned upload URL để client upload lên object storage.
4. Backend lưu metadata ảnh nếu cần và gọi FastAPI AI service.
5. AI service chạy pipeline Florence-2 (đề xuất vật thể qua tác vụ `<OD>` + mô tả vùng, mở rộng độ phủ bằng tiled OD và self-grounding), lọc nhãn qua chuỗi kiểm tra ngôn ngữ (từ điển + danh từ chỉ vật cụ thể), xác thực bằng CLIP, cắt nền bằng SAM, và trả về danh sách đối tượng gồm `label`, `detectionSource`, `clipScore`, `boundingBox`, `cropUrl` (mỗi từ tối đa một thẻ).
6. Backend lọc kết quả theo cặp nguồn phát hiện (detectionSource) và điểm xác thực (clipScore), chuẩn hóa label và ánh xạ sang từ vựng trong database.
7. Backend trả cho mobile danh sách từ vựng gồm từ tiếng Anh, nghĩa tiếng Việt, phiên âm, phát âm và metadata nhận diện.
8. Learner chọn từ muốn lưu vào danh sách học cá nhân.
9. Flashcard, quiz và SRS sử dụng danh sách từ đã lưu để tạo hoạt động học và ôn tập.

### 4.3 Trạng thái hiện tại và định hướng triển khai

| Nhóm                   | Hiện trạng theo mã nguồn                                                                                                                                                   | Định hướng trong SRS                                                                             |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Mobile app             | Đã có cấu trúc Expo/React Native, auth, home, learn, camera/profile tabs và API client                                                                                     | Hoàn thiện màn hình nghiệp vụ, kết nối scan-to-learn, learning engine và gamification            |
| Backend API            | Đã có auth, user, storage, word controller/service/domain                                                                                                                  | Mở rộng recognition, saved vocabulary, flashcard, quiz, SRS, progress, gamification              |
| Word / Learning domain | Đã có Word/Definition/Translation/Pronunciation và Deck/Note/Card/ReviewLog                                                                                                | Mở rộng CardTemplate, quiz attempt, progress aggregate, object→Note mapping từ scan              |
| AI service             | Pipeline Florence-2 ĐÃ được chứng minh trên Colab (notebook F2-v13): COCO128 box-F1 0,646 / word-F1 0,825; Internet-50 word-precision 0,885; ~40 thẻ đúng trên ảnh thực tế | Đóng gói pipeline thành FastAPI service độc lập (giữ nguyên cấu hình chế độ sản phẩm của F2-v13) |
| Storage                | Đã có hướng S3-compatible/MinIO và flow upload                                                                                                                             | Chuẩn hóa R2 production, presigned upload, metadata object và cleanup                            |

---

## 5. Yêu cầu chức năng

### FR-01 — Authentication & Account

| Mã       | Yêu cầu            | Mô tả                                                                  | Ưu tiên |
| -------- | ------------------ | ---------------------------------------------------------------------- | ------- |
| FR-01.01 | Đăng ký            | Guest tạo tài khoản bằng email, mật khẩu và thông tin cơ bản           | Must    |
| FR-01.02 | Xác thực email/OTP | Hệ thống gửi OTP hoặc link xác thực để kích hoạt tài khoản             | Must    |
| FR-01.03 | Đăng nhập          | Guest đăng nhập và nhận access token/refresh token hợp lệ              | Must    |
| FR-01.04 | Refresh token      | Mobile có thể làm mới access token khi token hết hạn                   | Must    |
| FR-01.05 | Khôi phục mật khẩu | Guest yêu cầu OTP/link reset và đặt mật khẩu mới                       | Must    |
| FR-01.06 | Đăng xuất          | Learner đăng xuất và phiên đăng nhập hiện tại bị vô hiệu hóa           | Should  |
| FR-01.07 | Hồ sơ cá nhân      | Learner xem/cập nhật tên, avatar và thông tin hồ sơ                    | Must    |
| FR-01.08 | Đăng nhập vân tay  | Learner có thể bật và sử dụng vân tay/sinh trắc học để đăng nhập nhanh | Should  |

Business rules:

- Mật khẩu phải được hash ở backend, không lưu plaintext.
- Đăng ký thành công tự động đăng nhập. Đặt lại mật khẩu thành công tự động revoke tất cả phiên đăng nhập cũ và bắt buộc đổi mật khẩu mới (khác mật khẩu cũ).
- OTP phải có thời hạn sử dụng và không được dùng lại sau khi thành công.
- API yêu cầu định danh phải kiểm tra JWT hợp lệ.
- Refresh token bị revoke khi đăng xuất hoặc khi có rủi ro bảo mật.

### FR-02 — Image Recognition Vocabulary Flow

| Mã       | Yêu cầu                  | Mô tả                                                                                                                                                        | Ưu tiên |
| -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| FR-02.01 | Chụp ảnh                 | Learner mở camera và chụp ảnh để nhận diện                                                                                                                   | Must    |
| FR-02.02 | Tải ảnh lên              | Learner chọn ảnh từ thư viện thiết bị                                                                                                                        | Must    |
| FR-02.03 | Nút nổi quét màn hình    | (Chỉ Android) Lớp phủ nút nổi để chụp và quét màn hình tự động khi đang dùng app khác                                                                        | Could   |
| FR-02.04 | Gửi ảnh xử lý            | Mobile gửi ảnh tới backend/AI pipeline theo flow đã cấu hình                                                                                                 | Must    |
| FR-02.05 | Nhận diện đối tượng      | AI service trả danh sách object label, detectionSource, clipScore, bounding box                                                                                              | Must    |
| FR-02.06 | Lọc độ tin cậy           | Lọc trong AI service (ngưỡng CLIP + biên độ); backend lọc thêm dựa trên cặp (source allowlist, clipScore floor) làm lớp bảo vệ cuối | Must    |
| FR-02.07 | Xử lý nhiều đối tượng    | Mobile hiển thị nhiều object để Learner chọn/lưu từng từ                                                                                                     | Should  |
| FR-02.08 | No-object/low-reliability| Hệ thống trả thông báo dễ hiểu và gợi ý thử ảnh khác                                                                                                         | Must    |
| FR-02.09 | Lưu kết quả scan         | Learner lưu từ được phát hiện vào danh sách học cá nhân                                                                                                      | Must    |
| FR-02.10 | Quota scan hằng ngày     | Mỗi Learner có giới hạn lượt scan/ngày cấu hình được; API trả số lượt còn lại và lỗi `QUOTA_EXCEEDED` khi hết lượt                                          | Must    |
| FR-02.11 | Hàng đợi xử lý AI        | Recognition request được đưa vào hàng đợi; backend trả trạng thái `QUEUED`/`PROCESSING` kèm vị trí ước tính thay vì giữ client treo                         | Must    |

Business rules:

- Nhãn từ AI service đã được chuẩn hóa và bảo đảm thuộc từ điển ngay trong pipeline (chuỗi lọc ngôn ngữ + cổng từ điển cuối); backend chỉ cần tra cứu trực tiếp, dùng bảng mapping/synonym cho trường hợp từ điển Anh-Việt thiếu mục tương ứng.
- Nếu nhiều bounding box cùng label, backend có thể gom trùng label để tránh trả từ vựng lặp.
- Mỗi Learner có quota scan/ngày mặc định 20 lượt, cấu hình được theo môi trường/gói; lượt chỉ bị trừ khi request ảnh hợp lệ được nhận vào hàng đợi.
- Khi hết lượt, backend trả `QUOTA_EXCEEDED` kèm `remainingScansToday = 0` và thời điểm reset; mobile không gọi AI service.
- Recognition request phải đi qua hàng đợi xử lý tuần tự/giới hạn worker (mặc định 1 worker/GPU, tối đa 2 nếu đo tải cho phép); trạng thái gồm `QUEUED`, `PROCESSING`, `SUCCESS`, `FAILED`, `CANCELED`.
- Hệ thống không tự động lưu toàn bộ kết quả scan nếu Learner chưa xác nhận.
- Ảnh scan chỉ được lưu nếu cần cho lịch sử hoặc debug; nếu lưu phải tuân thủ quyền riêng tư.

### FR-03 — Dictionary & Word Detail

| Mã       | Yêu cầu                   | Mô tả                                                                                                | Ưu tiên |
| -------- | ------------------------- | ---------------------------------------------------------------------------------------------------- | ------- |
| FR-03.01 | Tra cứu văn bản           | Learner gõ từ tiếng Anh để tìm kiếm trực tiếp trong database                                         | Must    |
| FR-03.02 | Tra cứu giọng nói (Voice) | Learner đọc tiếng Việt, hệ thống dùng Voice-to-Text chuyển thành chữ, dịch sang tiếng Anh và tra cứu | Should  |
| FR-03.03 | Nghĩa tiếng Việt          | Hệ thống hiển thị bản dịch/giải nghĩa tiếng Việt                                                     | Must    |
| FR-03.04 | Phiên âm                  | Hệ thống hiển thị IPA hoặc phiên âm có sẵn                                                           | Must    |
| FR-03.05 | Phát âm                   | Hệ thống cung cấp audio URL hoặc cơ chế phát âm từ                                                   | Must    |
| FR-03.06 | Nhiều nghĩa               | Nếu một từ có nhiều nghĩa/loại từ, hệ thống hiển thị theo nhóm rõ ràng                               | Should  |
| FR-03.07 | Quan hệ từ                | Hiển thị synonym/antonym/related words nếu dữ liệu hỗ trợ                                            | Could   |
| FR-03.08 | Ví dụ                     | Hiển thị câu ví dụ nếu dữ liệu có sẵn hoặc được bổ sung sau                                          | Could   |

Business rules:

- Database từ vựng là nguồn chính để trả thông tin học tập sau khi object label được nhận diện.
- Nếu label không khớp chính xác với từ điển, backend có thể dùng bảng mapping/synonym để ánh xạ.
- Không hiển thị dữ liệu thiếu dưới dạng rỗng khó hiểu; UI cần phân biệt “chưa có dữ liệu phát âm/phiên âm”.

### FR-04 — Personal Vocabulary (Deck / Note)

| Mã       | Yêu cầu            | Mô tả                                                                     | Ưu tiên |
| -------- | ------------------ | ------------------------------------------------------------------------- | ------- |
| FR-04.01 | Lưu từ             | Learner lưu từ từ kết quả scan hoặc tra cứu từ điển                       | Must    |
| FR-04.02 | Danh sách cá nhân  | Learner xem danh sách từ đã lưu                                           | Must    |
| FR-04.03 | Xóa khỏi danh sách | Learner xóa hoặc archive từ không muốn học tiếp                           | Must    |
| FR-04.04 | Trạng thái học     | Mỗi từ hiển thị state UI suy từ FSRS: new, learning, reviewing, mastered | Should  |
| FR-04.05 | Lọc/sắp xếp        | Learner lọc theo trạng thái, ngày lưu, độ khó hoặc ngày ôn                | Should  |
| FR-04.06 | Gắn nguồn          | Hệ thống lưu nguồn của từ: scan ảnh, tra cứu dictionary, topic/collection | Could   |

Business rules:

- Canonical model: `Deck` → `Note` → `Card`. UI “My Vocabulary / từ đã lưu” = danh sách Note của Learner.
- Không tạo entity song song `SavedWord`/`UserWord`.
- Một Learner không nên có Note trùng cùng một Word trong cùng một Deck (unique theo rule Deck).
- Xóa/archive Note không xóa Word khỏi dictionary gốc; Card/SRS gắn Note được ẩn hoặc archive theo rule.
- Note/Card là nguồn đầu vào chính cho flashcard, quiz và SRS.

Learning state UI là taxonomy hiển thị/aggregate, không phải enum FSRS lưu trực tiếp trên Card:

| FSRS Card.state | Điều kiện bổ sung | UI/Progress state |
| --------------- | ----------------- | ----------------- |
| `NEW` | — | `new` |
| `LEARNING` | — | `learning` |
| `RELEARNING` | — | `learning` |
| `REVIEW` | `interval < 21 ngày` | `reviewing` |
| `REVIEW` | `interval >= 21 ngày` | `mastered` |

Business rules cho learning state:

- `mastered` là trạng thái suy ra từ Card FSRS đã mature, không phải Learner đánh dấu tay.
- Ngưỡng mature/mastered mặc định là `interval >= 21 ngày` theo quy ước mature card tương tự Anki; nếu cấu hình sản phẩm đổi ngưỡng, mọi query/list/progress/mission phải dùng cùng một config.
- `learnedCount` = số Card không còn `new` (`learning + reviewing + mastered`); `dueCount`/`đang ôn` = số Card có `dueAt <= now`; `masteredCount` = số Card có UI state `mastered`.

### FR-05 — Flashcard & Custom Card

| Mã       | Yêu cầu               | Mô tả                                                                                    | Ưu tiên |
| -------- | --------------------- | ---------------------------------------------------------------------------------------- | ------- |
| FR-05.01 | Tạo flashcard         | Hệ thống sinh 1 flashcard duy nhất cho mỗi từ vựng được lưu (Note)                       | Must    |
| FR-05.02 | System Template       | Cung cấp sẵn các mẫu thẻ hệ thống (Classic, Listening, Spelling, Image Vocab...)         | Must    |
| FR-05.03 | Custom Template       | Learner tự tạo template cấu hình layout, field mapping và kiểu tương tác (Flip, Type-in) | Should  |
| FR-05.04 | Gán Template cho Deck | Learner chọn 1 template áp dụng cho toàn bộ thẻ trong một bộ bài (Deck)                  | Must    |
| FR-05.05 | Render theo Config    | Mobile app linh hoạt render giao diện thẻ dựa trên cấu hình template trả về từ API       | Must    |
| FR-05.06 | Đánh giá recall       | Learner chọn mức độ nhớ (FSRS) sau mỗi thẻ để tính lịch ôn tiếp theo                     | Must    |

Business rules:

- System template không thể sửa/xóa; Custom template xóa mềm (soft-delete).
- Khi đổi Template của Deck, Card cũ không bị mất mà chỉ thay đổi cách render (vỏ bọc), SRS giữ nguyên.
- Nếu từ thiếu audio/IPA, flashcard vẫn hoạt động, tự động ẩn field tương ứng mà không lỗi layout.

### FR-06 — Quiz

| Mã       | Yêu cầu          | Mô tả                                                  | Ưu tiên |
| -------- | ---------------- | ------------------------------------------------------ | ------- |
| FR-06.01 | Tạo quiz         | Hệ thống sinh quiz từ Note/Card trong Deck của Learner | Must    |
| FR-06.02 | Multiple choice  | Learner chọn nghĩa/từ đúng từ nhiều đáp án             | Must    |
| FR-06.03 | Matching         | Learner ghép từ tiếng Anh với nghĩa tiếng Việt         | Should  |
| FR-06.04 | Fill blank       | Learner điền từ còn thiếu trong câu/gợi ý              | Could   |
| FR-06.05 | Chấm điểm        | Hệ thống tính điểm, số câu đúng/sai và tỷ lệ chính xác | Must    |
| FR-06.06 | Lịch sử attempt  | Hệ thống lưu kết quả mỗi lượt làm quiz                 | Should  |
| FR-06.07 | Cập nhật tiến độ | Quiz ảnh hưởng đến progress, XP và nhiệm vụ nếu có     | Should  |

Business rules:

- Đáp án nhiễu lấy từ Note cùng Deck/POS, không trùng nghĩa.
- Nếu số lượng Note/Card chưa đủ (tối thiểu 4 thẻ), hệ thống cần thông báo Learner lưu thêm từ trước khi tạo quiz.
- Thoát quiz giữa chừng hệ thống sẽ hủy bỏ, không lưu draft.
- Kết quả quiz không cập nhật thông số FSRS (chỉ ghi nhận QuizAttempt, progress, XP).

### FR-07 — Spaced Repetition System (SRS)

| Mã       | Yêu cầu          | Mô tả                                                    | Ưu tiên |
| -------- | ---------------- | -------------------------------------------------------- | ------- |
| FR-07.01 | Review queue     | Hệ thống hiển thị danh sách Card đến hạn ôn tập          | Must    |
| FR-07.02 | Recall quality   | Learner đánh giá mức nhớ sau khi xem flashcard/review    | Must    |
| FR-07.03 | Tính lịch ôn     | Hệ thống cập nhật due date, interval và trạng thái học   | Must    |
| FR-07.04 | Daily review     | Learner xem số lượng từ cần ôn trong ngày                | Should  |
| FR-07.05 | Overdue handling | Từ quá hạn ôn được ưu tiên trong review queue            | Should  |
| FR-07.06 | Reset/Archive    | Learner có thể bỏ qua hoặc reset lịch học của từ nếu cần | Could   |

Business rules:

- Từ mới được đưa vào trạng thái học ban đầu và có lịch ôn đầu tiên.
- Recall tốt làm tăng khoảng cách ôn; recall kém làm giảm khoảng cách hoặc đưa từ về trạng thái learning.
- Review queue chỉ gồm Card thuộc Deck/Note của Learner hiện tại.

### FR-08 — Progress Tracking

| Mã       | Yêu cầu           | Mô tả                                                    | Ưu tiên |
| -------- | ----------------- | -------------------------------------------------------- | ------- |
| FR-08.01 | Tổng quan tiến độ | Learner xem số từ đã lưu, đã học, đang ôn và mastered theo bảng map learning state | Must    |
| FR-08.02 | Streak            | Hệ thống tính chuỗi ngày học liên tiếp                   | Should  |
| FR-08.03 | Accuracy          | Hệ thống tính độ chính xác quiz/review                   | Should  |
| FR-08.04 | Lịch sử học       | Learner xem hoạt động học theo ngày/tuần/tháng           | Should  |
| FR-08.05 | Goal tracking     | Learner theo dõi mục tiêu học hằng ngày nếu có           | Could   |
| FR-08.06 | Progress summary  | Home screen hiển thị summary ngắn gọn về tiến độ         | Must    |

Business rules:

- Progress phải cập nhật sau các hoạt động học quan trọng: lưu từ, flashcard, review, quiz.
- Các count `learned`/`đang ôn`/`mastered` dùng cùng bảng map learning state ở FR-04.
- Streak chỉ tăng khi Learner hoàn thành điều kiện học tối thiểu trong ngày (thực hiện tối thiểu 1 review hoặc 1 quiz).
- Dữ liệu progress cá nhân chỉ hiển thị cho chính Learner và Admin có quyền phù hợp, ngoại trừ `displayName`, `avatar` và `Weekly XP` được công khai trên bảng xếp hạng.

### FR-09 — Leaderboard & Gamification

| Mã       | Yêu cầu             | Mô tả                                                                | Ưu tiên |
| -------- | ------------------- | -------------------------------------------------------------------- | ------- |
| FR-09.01 | XP                  | Learner nhận điểm kinh nghiệm khi hoàn thành hoạt động học           | Should  |
| FR-09.02 | Coin                | Learner nhận coin theo nhiệm vụ hoặc milestone học tập               | Should  |
| FR-09.03 | Mission             | Hệ thống cung cấp nhiệm vụ ngày/tuần/thành tựu                       | Should  |
| FR-09.04 | Badge               | Learner nhận huy hiệu khi đạt điều kiện cụ thể                       | Should  |
| FR-09.05 | Leaderboard cá nhân | Learner xem xếp hạng theo XP tuần (Weekly XP). Lượng XP tổng kết từ 00:00 Thứ Hai đến 23:59 Chủ Nhật (theo múi giờ cấu hình của server). | Should  |
| FR-09.06 | Shop item           | Learner dùng coin mua vật phẩm trong cửa hàng                        | Could   |
| FR-09.07 | Apply item          | Learner áp dụng vật phẩm như theme, avatar frame hoặc booster nếu có | Could   |

Business rules:

- Reward phải có rule rõ ràng để tránh cộng trùng khi retry API.
- Coin chỉ là đơn vị trong ứng dụng, không quy đổi thành tiền thật trong phạm vi đồ án.
- Leaderboard nên dùng Redis/cache khi dữ liệu tăng hoặc cần cập nhật thường xuyên.
- Daily missions reset lúc 00:00 múi giờ Asia/Ho_Chi_Minh.

### FR-10 — Notification System

| Mã       | Yêu cầu             | Mô tả                                                                 | Ưu tiên |
| -------- | ------------------- | --------------------------------------------------------------------- | ------- |
| FR-10.01 | Push Notification   | Gửi thông báo đẩy nhắc nhở Learner vào ôn tập từ vựng (SRS)           | Must    |
| FR-10.02 | In-app Notification | Hiển thị thông báo trong ứng dụng (tin báo hệ thống, nhận Badge/Coin) | Should  |
| FR-10.03 | Cấu hình thông báo  | Learner có thể bật/tắt nhận thông báo đẩy trong phần Cài đặt          | Must    |

Business rules:

- Push Notification sử dụng dịch vụ trung gian (Expo Push hoặc Firebase FCM).
- In-app Notification được lưu trong database để Learner xem lại khi mở app.
- Tối đa 1 push nhắc SRS/ngày khung 19–21h, tuân thủ cấu hình giờ nhận của Learner (nếu có).

### FR-11 — Storage & Media

| Mã       | Yêu cầu            | Mô tả                                                          | Ưu tiên |
| -------- | ------------------ | -------------------------------------------------------------- | ------- |
| FR-11.01 | Avatar upload      | Learner upload avatar qua storage flow                         | Must    |
| FR-11.02 | Scan image storage | Hệ thống lưu ảnh scan nếu cần cho lịch sử hoặc debug           | Should  |
| FR-11.03 | Presigned upload   | Backend cấp URL upload trực tiếp tới object storage            | Must    |
| FR-11.04 | Upload complete    | Client báo upload hoàn tất để backend validate và lưu metadata | Must    |
| FR-11.05 | Private bucket     | Media không công khai trực tiếp nếu chứa dữ liệu cá nhân       | Should  |

Business rules:

- Không dùng tên file người dùng nhập làm object key trực tiếp.
- Backend phải validate loại file/kích thước ở biên hệ thống.
- Object storage chỉ lưu binary; quyền truy cập và metadata do backend/database kiểm soát.

### FR-12 — API Documentation

| Mã       | Yêu cầu        | Mô tả                                                               | Ưu tiên |
| -------- | -------------- | ------------------------------------------------------------------- | ------- |
| FR-12.01 | Swagger UI     | Backend cung cấp Swagger/OpenAPI để kiểm thử API                    | Must    |
| FR-12.02 | API grouping   | API được nhóm theo auth, user, word, storage, recognition, learning | Should  |
| FR-12.03 | DTO schema     | Request/response chính có schema rõ trong OpenAPI                   | Should  |
| FR-12.04 | Error response | Lỗi API có format thống nhất và được tài liệu hóa                   | Should  |

Business rules:

- Không hardcode secrets hoặc thông tin môi trường thật vào tài liệu API.
- Endpoint nội bộ giữa backend và AI service cần tách rõ với endpoint public/mobile.

### FR-13 — Admin Dashboard & Management

| Mã       | Yêu cầu                | Mô tả                                                                                        | Ưu tiên |
| -------- | ---------------------- | -------------------------------------------------------------------------------------------- | ------- |
| FR-13.01 | Quản lý người dùng     | Admin xem danh sách user, chi tiết tiến độ, có thể ban/unban hoặc reset mật khẩu             | Must    |
| FR-13.02 | Quản lý từ điển        | Admin thêm, sửa, xóa các từ vựng, định nghĩa, phiên âm và chủ đề học tập                     | Must    |
| FR-13.03 | Import dữ liệu         | Admin có công cụ import từ vựng hàng loạt qua CSV/Excel hoặc API                             | Should  |
| FR-13.04 | Quản lý Gamification   | Admin cấu hình điểm XP, thiết lập Missions, Badges và vật phẩm Shop                          | Should  |
| FR-13.05 | Quản lý lỗi / Feedback | Admin xem danh sách các từ vựng bị người dùng báo lỗi để điều chỉnh                          | Could   |
| FR-13.06 | Thống kê hệ thống      | Dashboard hiển thị biểu đồ người dùng, mức sử dụng AI, độ dài hàng đợi AI, quota scan và dung lượng R2/S3 | Should  |
| FR-13.07 | Quản lý Card Template  | Admin có thể quản lý, tạo mới và cấu hình các mẫu System Template (Classic, Listening, v.v.) | Must    |

Business rules:

- CMS/Dashboard chạy độc lập với Mobile App (có thể là Web App nội bộ).
- Các API quản trị phải được bảo vệ bởi role ROLE_ADMIN.
- Admin xem được AI queue depth, p95 queue wait, p95 processing time, số job timeout/failed và phân bổ lượt scan theo ngày.
- Quota scan/ngày và số worker AI/GPU là cấu hình vận hành, không hardcode trong mobile.
- Hành động xóa từ vựng phải là soft-delete (xóa mềm) để không làm hỏng dữ liệu flashcard của Learner hiện tại.

---

### FR-14 — Topic Learning (Collections & Topics)

| Mã       | Yêu cầu                       | Mô tả                                                                                     | Ưu tiên |
| -------- | ----------------------------- | ----------------------------------------------------------------------------------------- | ------- |
| FR-14.01 | Duyệt Collections             | Learner xem danh sách bộ sưu tập từ vựng có sẵn (TOEIC, Animals…) với pagination          | Must    |
| FR-14.02 | Duyệt Topics trong Collection | Learner duyệt danh sách Topic thuộc Collection; hỗ trợ phân cấp parent/child              | Must    |
| FR-14.03 | Xem TopicItems + thuộc tính  | Learner xem danh sách từ vựng trong Topic kèm thuộc tính EAV (nghĩa, IPA, ví dụ, audio)  | Must    |
| FR-14.04 | Lưu TopicItem thành Note      | Learner lưu từ vựng từ Topic vào Deck được chọn/gần nhất/mặc định với source = TOPIC      | Must    |
| FR-14.05 | Admin CRUD Collection/Topic   | Admin quản lý cấu trúc Collection và Topic (thêm, sửa, xóa mềm), hỗ trợ phân cấp         | Must    |
| FR-14.06 | Admin CRUD TopicItem + EAV    | Admin quản lý nội dung từ vựng theo mô hình EAV (TopicAttributeGroup/Attribute/Value)    | Must    |

Business rules:

- Mô hình dữ liệu EAV: TopicAttributeGroup → TopicAttribute → TopicItemAttributeValue.
- Soft-delete Collection/Topic không xóa Note/Card đã lưu của Learner.
- Learner không có quyền tạo/sửa/xóa Collection hoặc Topic (read-only + lưu).
- Unique per Deck: Learner không lưu trùng cùng TopicItem vào cùng Deck (áp dụng rule unique per Deck của FR-04).

---

## 6. Mô hình dữ liệu tổng quan

### 6.1 Nhóm dữ liệu đã có hoặc có bằng chứng mạnh trong mã nguồn

| Entity                         | Mục đích                                       | Ghi chú                                                |
| ------------------------------ | ---------------------------------------------- | ------------------------------------------------------ |
| User                           | Tài khoản và hồ sơ người dùng                  | Phục vụ auth, profile và quyền truy cập                |
| Authority                      | Quyền/vai trò bảo vệ API                       | Dùng cho Spring Security/JWT                           |
| Word                           | Từ vựng gốc trong dictionary                   | Trung tâm của chức năng tra cứu/học                    |
| Definition                     | Định nghĩa hoặc giải thích của từ              | Có thể có nhiều định nghĩa cho một từ                  |
| Translation                    | Bản dịch/nghĩa tiếng Việt hoặc ngôn ngữ khác   | Dùng để hiển thị nghĩa học tập                         |
| Pronunciation                  | Phiên âm, audio hoặc dữ liệu phát âm           | Dùng trong word detail/flashcard                       |
| WordDefinition                 | Liên kết word-definition nếu domain tách riêng | Hỗ trợ nhiều nghĩa/loại từ                             |
| WordRelation                   | Quan hệ giữa các từ                            | Synonym, antonym hoặc related words nếu dữ liệu hỗ trợ |
| Deck / Note / Card / ReviewLog | Học tập cá nhân + SRS                          | Canonical learning model — xem §6.2                    |

### 6.2 Nhóm dữ liệu học tập cá nhân (canonical — khớp code)

> Thuật ngữ UI “từ đã lưu / My Vocabulary” = các `Note` thuộc `Deck` của Learner.  
> Không duy trì entity song song `SavedWord`/`UserWord`.

| Entity                            | Mục đích                                                     | Ghi chú                                      |
| --------------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| Deck                              | Bộ thẻ của Learner; gán 1 CardTemplate                       | Đã có trong code                             |
| Note                              | Đơn vị từ vựng cá nhân muốn học (word + nguồn)               | Đã có; thay cho SavedWord                    |
| NoteMeaning                       | Nghĩa/POS/example/ghi chú gắn Note                           | Đã có                                        |
| NotePronunciation                 | IPA/audio gắn Note                                           | Đã có                                        |
| Card                              | Thẻ học + tham số SRS (state, dueAt, stability, difficulty…) | Đã có; 1 Note → 1 Card theo template Deck    |
| ReviewLog                         | Lịch sử từng lượt ôn (rating, time)                          | Đã có                                      |
| CardTemplate                      | Layout system/custom, interaction type                       | **Entity đầy đủ M3**; M1 seed 1 bản ghi CLASSIC hard-code, không có CRUD/UI — xem custom_card |
| CardTemplateField                 | Field mapping front/back                                     | **Entity đầy đủ M3** (đi kèm CardTemplate CRUD)              |
| Quiz / QuizQuestion / QuizAttempt | Kiểm tra từ trong Deck/Note                                  | Dự kiến M3                                   |
| Notification / DeviceToken        | In-app + push                                                | Dự kiến M3                                   |
| LearningProgress / LearningEvent  | Aggregate streak, accuracy, summary                          | Dự kiến M3; rebuild từ ReviewLog/QuizAttempt |

### 6.3 Nhóm dữ liệu cần cho nhận diện ảnh

| `ScanRequest`           | Lưu request xử lý ảnh, user, object key, trạng thái, và thời gian xử lý (gộp ImageRecognitionRequest, RecognitionResult, ScanHistory) |
| `DetectedObject`        | Label, detectionSource, clipScore, bounding box, cropUrl cho từng đối tượng                                                           |
| `ObjectWordMapping`     | Mapping từ nhãn AI pipeline sang Word trong dictionary (nhãn đã thuộc từ điển tiếng Anh; bảng này xử lý ánh xạ sang mục từ Anh-Việt)  |

### 6.4 Nhóm dữ liệu cần cho gamification

| Entity đề xuất   | Mục đích                                |
| ---------------- | --------------------------------------- |
| Mission          | Nhiệm vụ ngày/tuần/thành tựu            |
| MissionProgress  | Tiến độ thực hiện nhiệm vụ của Learner  |
| Badge            | Định nghĩa huy hiệu                     |
| UserBadge        | Huy hiệu Learner đã đạt được            |
| CoinTransaction  | Lịch sử cộng/trừ coin                   |
| ExperienceLog    | Lịch sử cộng XP                         |
| ShopItem         | Vật phẩm trong cửa hàng                 |
| UserItem         | Vật phẩm Learner đã sở hữu/đang sử dụng |
| LeaderboardEntry | Bản ghi xếp hạng cá nhân                |

### 6.5 Nhóm dữ liệu Chủ đề học tập

Sử dụng mô hình EAV (Entity-Attribute-Value) để lưu trữ các bộ từ vựng, chủ đề theo cấu trúc linh hoạt.

| Entity                  | Mục đích                                                                        |
| ----------------------- | ------------------------------------------------------------------------------- |
| Collection              | Tập hợp các chủ đề lớn (VD: TOEIC Words, Animals)                               |
| Topic                   | Các chủ đề cụ thể, có hỗ trợ phân cấp (parent_id) thuộc một Collection          |
| TopicItem               | Phần tử nội dung chi tiết (một từ vựng, cụm từ) thuộc Topic                     |
| TopicAttributeGroup     | Khai báo nhóm thuộc tính cho một Topic                                          |
| TopicAttribute          | Định nghĩa các thuộc tính cần có (VD: Nghĩa tiếng Việt, Phiên âm, Ví dụ, Audio) |
| TopicItemAttributeValue | Lưu trữ giá trị thực tế của các thuộc tính tương ứng cho từng TopicItem         |

---

## 7. Yêu cầu API và tích hợp

### 7.1 Nhóm API backend

| Nhóm API            | Trạng thái                      | Mục đích                                                                 |
| ------------------- | ------------------------------- | ------------------------------------------------------------------------ |
| Authentication API  | Đã có bằng chứng trong mã nguồn | Register, login, refresh token, OTP/email, forgot/reset password         |
| User/Profile API    | Đã có bằng chứng trong mã nguồn | Xem/cập nhật hồ sơ người dùng                                            |
| Word/Dictionary API | Đã có bằng chứng trong mã nguồn | Tra cứu từ, xem word detail, definition, translation, pronunciation      |
| Storage API         | Đã có bằng chứng trong mã nguồn | Presigned upload, upload complete, media metadata                        |
| Recognition API     | Dự kiến/MVP AI                  | Gửi ảnh nhận diện, kiểm tra quota, tạo job hàng đợi, theo dõi trạng thái, nhận detected objects, map sang vocabulary |
| Deck / Note API     | Đã có bằng chứng một phần       | CRUD Deck/Note — tương đương “saved vocabulary” UI                       |
| Card Template API   | Dự kiến learning module         | CRUD custom template, lấy system templates                               |
| Card / Review API   | Đã có domain Card/ReviewLog     | Queue ôn, submit rating FSRS, render theo template                       |
| Quiz API            | Dự kiến learning module         | Tạo quiz, submit answer, lưu attempt                                     |
| SRS/Review API      | Dự kiến learning module         | Lấy review queue, cập nhật lịch ôn                                       |
| Progress API        | Dự kiến learning module         | Tổng hợp tiến độ, streak, accuracy, activity                             |
| Gamification API    | Dự kiến reward module           | Mission, badge, coin, shop, leaderboard                                  |
| Notification API    | Dự kiến notification module     | Đăng ký device token, lấy danh sách in-app notification, đánh dấu đã đọc |
| Admin API           | Dự kiến admin module            | Quản lý user, dictionary, gamification, dashboard stats                  |

### 7.2 Tích hợp AI service

| Thuộc tính     | Yêu cầu                                                                                                                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Giao thức      | Worker recognition gọi FastAPI AI service qua HTTP nội bộ; mobile/backend public giao tiếp qua `requestId` và trạng thái job                                                             |
| Input          | Ảnh hoặc object key/URL tạm thời truy cập ảnh                                                                                                                                            |
| Output         | Danh sách object: label (bảo đảm thuộc từ điển), confidence, bounding box, crop cắt nền (tùy chọn)                                                                                       |
| Model          | Florence-2-large zero-shot + SAM (ViT-H) + CLIP (ViT-B/32), cấu hình chế độ sản phẩm F2-v13 (tiled OD, self-grounding, từ vựng nền + cửa CLIP, 1 thẻ/từ)                                 |
| Confidence     | Florence-2 không trả xác suất từng box; điểm tin cậy là pseudo-score theo nguồn phát hiện, có thể thay bằng điểm CLIP khi bật xác thực toàn phần — tài liệu API cần ghi rõ ngữ nghĩa này |
| Phần cứng      | Cần GPU (T4 trở lên); độ trễ ~15–30 giây/ảnh ở chế độ đầy đủ — xem NFR Recognition Performance                                                                                           |
| Error handling | Timeout, model error, invalid image, no-object, quota exceeded và queue full phải được trả về có cấu trúc                                                                                |
| Logging        | Ghi nhận requestId, trạng thái, queue wait, thời gian xử lý, số object nhận diện, lỗi nếu có                                                                                             |

### 7.3 Tích hợp dictionary

| Thuộc tính         | Yêu cầu                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| Nguồn              | Dữ liệu Từ điển SQLite từ minhqnd/dictionary                                                        |
| Quy mô             | Khoảng 357,729+ từ vựng và 443,116+ định nghĩa theo mô tả đề tài                                    |
| Xử lý              | Chuẩn hóa, import vào MySQL/MariaDB và ánh xạ sang domain Word/Definition/Translation/Pronunciation |
| Tra cứu            | Backend cung cấp API lookup theo từ, label nhận diện hoặc mapping tương đương                       |
| Chất lượng dữ liệu | Cần xử lý trùng lặp, thiếu nghĩa, thiếu phiên âm hoặc bản dịch không phù hợp                        |

### 7.4 Tích hợp object storage

| Thuộc tính | Yêu cầu                                                                        |
| ---------- | ------------------------------------------------------------------------------ |
| Dev/local  | Có thể dùng MinIO hoặc S3-compatible storage                                   |
| Production | Cloudflare R2 qua S3-compatible API                                            |
| Upload     | Ưu tiên presigned URL để mobile upload trực tiếp khi phù hợp                   |
| Access     | Bucket private; backend cấp URL tạm thời hoặc proxy khi cần                    |
| Metadata   | Database lưu object key, owner, loại media, kích thước, MIME, thời điểm upload |
| Cleanup    | Có job hoặc quy trình xóa object không còn được tham chiếu                     |

---

## 8. Yêu cầu phi chức năng

| #   | Nhóm                    | Yêu cầu (đo được khi có thể)                                                                                     |
| --- | ----------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | Security                | API cá nhân yêu cầu JWT; password hash an toàn; refresh token revoke được; không log secret/OTP                  |
| 2   | Authorization           | Learner chỉ truy cập dữ liệu cá nhân; Admin API yêu cầu `ROLE_ADMIN`                                             |
| 3   | OTP Safety              | OTP TTL ≤ 10 phút; tối đa 5 lần thử; resend cooldown ≥ 60s; không tái sử dụng sau success                        |
| 4   | Idempotency             | Reward/XP/coin/claim mission và submit quiz dùng event key — retry không cộng trùng                              |
| 5   | File Safety             | Validate MIME allowlist ảnh; avatar ≤ 5MB, scan ≤ 10MB (trừ khi cấu hình khác); object key do backend sinh       |
| 6   | Privacy                 | Bucket private; presigned URL TTL ngắn (≤ 15 phút); ảnh scan/avatar không public mặc định                        |
| 7   | Recognition Performance | Recognition xử lý qua hàng đợi; mặc định 1 worker/GPU; timeout worker→AI 60s cấu hình được; GPU T4 ~15–30s/ảnh full mode |
| 8   | Recognition Quota       | Quota scan/ngày mặc định 20/Learner, cấu hình được; hết lượt trả `QUOTA_EXCEEDED`; job bị từ chối do queue full không trừ lượt |
| 9   | Dictionary Performance  | Lookup word phổ biến p95 < 500ms (server, exclude mobile network); cache Redis top words khi cần                 |
| 10  | Leaderboard Performance | Không full-scan aggregate mỗi request; Redis sorted set hoặc snapshot cache TTL rõ                               |
| 11  | Reliability             | AI lỗi / no-object / low-reliability / upload fail / dictionary miss → error có `code` + message, app không crash |
| 12  | Scalability             | Tách mobile, backend, AI, DB, Redis, object storage scale độc lập                                                |
| 13  | API Standard            | REST + JSON envelope thống nhất (`success/data/error/requestId`); OpenAPI cho endpoint public                    |
| 14  | Mobile UX               | Scan-to-save ≤ 3 bước chính sau khi có ảnh; empty/error/loading có CTA rõ                                        |
| 15  | Accessibility           | Contrast đọc được trên mobile; touch target ≥ 44pt cho CTA chính                                                 |
| 16  | Admin isolation         | CMS/Admin không nằm trong mobile app; tách deploy/route                                                          |
| 17  | Maintainability         | FE theo feature; BE controller-service-repository + DTO/mapper                                                   |
| 18  | Observability           | Log requestId, auth fail, recognition status/queue wait/latency/object count, upload fail, learning events; health checks bắt buộc |

### Security decisions

- Access token dùng để gọi API; refresh token dùng để cấp lại access token theo flow bảo mật của backend.
- Password và OTP không bao giờ được log hoặc trả về response.
- Upload media phải giới hạn loại file và kích thước, không tin hoàn toàn vào metadata từ client.
- Presigned URL cần có thời hạn ngắn và gắn với object key do backend sinh.
- API giữa backend và AI service nên nằm trong mạng nội bộ hoặc có cơ chế xác thực riêng nếu public.

### Reliability decisions

- Recognition request cần có timeout rõ ràng khi worker gọi AI service.
- Backend không gọi AI service trực tiếp từ request mobile; request hợp lệ được đưa vào hàng đợi và client theo dõi bằng `requestId`.
- Queue phải có giới hạn nhận job; khi đầy, backend trả `AI_QUEUE_FULL` và không trừ quota.
- Nếu AI service unavailable, backend trả lỗi nghiệp vụ có message thân thiện cho mobile.
- Nếu label không map được sang dictionary, hệ thống trả kết quả nhận diện nhưng đánh dấu chưa có từ vựng tương ứng.
- Hoạt động cộng XP/coin phải idempotent để tránh cộng trùng khi client retry.
- Các aggregate như progress/leaderboard có thể rebuild từ log học tập hoặc dữ liệu nguồn khi cần.

---

## 9. Acceptance Criteria tổng thể

| #   | Tiêu chí nghiệm thu                                                                                       |
| --- | --------------------------------------------------------------------------------------------------------- |
| 1   | Guest đăng ký, xác thực, đăng nhập (bao gồm vân tay/sinh trắc học nếu bật) và khôi phục mật khẩu hợp lệ.  |
| 2   | Learner xem/cập nhật được hồ sơ cá nhân và avatar.                                                        |
| 3   | Learner tra cứu từ tiếng Anh và nhận nghĩa tiếng Việt, phiên âm, phát âm nếu dữ liệu có.                  |
| 4   | Learner duyệt được Collection/Topic và đưa từ chủ đề vào Deck/Note cá nhân (nếu Topic Learning bật).      |
| 5   | Learner tạo/xóa Note trong Deck; danh sách My Vocabulary phản ánh đúng Note còn hiệu lực.                 |
| 6   | Learner học Card bằng flashcard; hệ thống ghi ReviewLog và cập nhật SRS trên Card.                        |
| 7   | Learner chụp/chọn ảnh, gửi xử lý, thấy `QUEUED`/`PROCESSING` khi phải chờ và nhận object từ AI (Florence-2 pipeline). |
| 8   | Backend kiểm tra quota scan/ngày, trả lượt còn lại/resetAt và không gọi AI khi `QUOTA_EXCEEDED`.          |
| 9   | Backend ánh xạ label sang Word dictionary và trả word detail cho mobile.                                  |
| 10  | No-object / low-reliability / AI lỗi / queue full trả `error.code` rõ, app không crash.                    |
| 11  | Learner lưu được từ từ ảnh thành Note/Card trong Deck.                                                    |
| 12  | Learner làm quiz từ Note/Deck và xem điểm/đúng-sai.                                                       |
| 13  | Daily review queue lấy Card `dueAt` đến hạn; rating cập nhật lịch ôn.                                     |
| 14  | Home/progress hiển thị summary học tập khớp ReviewLog/QuizAttempt.                                        |
| 15  | Leaderboard cá nhân phản ánh XP/activity theo rule (M4).                                                  |
| 16  | Admin (nếu milestone bật) ban/unban hoặc CRUD dictionary qua role `ROLE_ADMIN`, không qua mobile Learner. |
| 17  | Mission/badge/XP/coin idempotent — retry không cộng trùng.                                                |
| 18  | Learner mua item bằng coin; UserItem được ghi nhận (nếu Shop được triển khai).                            |
| 19  | Upload media private bucket + presigned; không lộ object ngoài quyền.                                     |
| 20  | OpenAPI/Swagger đủ cho mobile tích hợp; error envelope thống nhất.                                        |
| 21  | Docs/code/UI không chứa secrets môi trường thật.                                                          |

---

## 10. Tài liệu liên quan

| Tài liệu              | Đường dẫn                                                    | Vai trò                      |
| --------------------- | ------------------------------------------------------------ | ---------------------------- |
| Luồng nghiệp vụ chính | [buss_mainflow.md](./buss_mainflow.md)                       | BF — actor, bước, ngoại lệ   |
| Phân rã phân hệ       | [phan_ra_phan_he_he_thong.md](./phan_ra_phan_he_he_thong.md) | SS — ranh giới module        |
| Phân rã tính năng     | [phan_ra_tinh_nang.md](./phan_ra_tinh_nang.md)               | F — sub-feature + AC theo FR |
| Phân rã màn hình      | [phan_ra_man_hinh.md](./phan_ra_man_hinh.md)                 | MH — UI map                  |
| API Spec              | [../API_SPEC.md](../API_SPEC.md)                             | Contract endpoint (cần viết) |
| Database Spec         | [../db/database.md](../db/database.md)                       | Entity/index canonical       |
| Thiết kế UI/UX        | [../design.md](../design.md)                                 | Figma/tokens (cần bổ sung)   |
| System Architecture   | [../sa/sa.md](../sa/sa.md)                                   | Kiến trúc tổng thể           |
| Tech Stack            | [../sa/techstack.md](../sa/techstack.md)                     | Công nghệ theo lớp           |
| Server & Deployment   | [../sa/server.md](../sa/server.md)                           | Môi trường, ops, smoke test  |
| Docs index            | [../README.md](../README.md)                                 | Mục lục + quy tắc đồng bộ    |

---

## 11. Checklist kiểm tra tài liệu

- Tiêu đề và nội dung mô tả SnapVocab.
- Actor chính đúng theo đề tài: Guest, Learner, Admin.
- Luồng scan-to-learn được mô tả rõ từ mobile → backend → AI service → dictionary → saved vocabulary.
- Stack khớp với dự án: React Native/Expo, Spring Boot, Spring Security/JWT, MySQL/MariaDB, Redis, Cloudflare R2/S3-compatible storage, FastAPI, Florence-2 + SAM + CLIP (pipeline open-vocabulary), Swagger/OpenAPI, Figma.
- Tài liệu phân biệt phần đã có bằng chứng trong mã nguồn và phần dự kiến triển khai.
- Functional requirements bao phủ auth, profile, recognition, dictionary, topic/collection learning, Deck/Note vocabulary, flashcard/custom card, quiz, SRS, progress, leaderboard, gamification, storage, notification và Admin.
- Non-functional requirements bao phủ security, performance, reliability, scalability, usability, maintainability và observability.
- Milestone có done criteria đo được để phục vụ triển khai và nghiệm thu đồ án.
- Không chứa secrets, mật khẩu, token, endpoint nhạy cảm hoặc giá trị cấu hình local/dev.

---

## 12. Phụ lục A — Thuật ngữ AI cần biết

Các định nghĩa dưới đây giải thích thuật ngữ theo đúng cách chúng được dùng trong pipeline của dự án (notebook F2-v13), để người đọc tài liệu không cần nền tảng thị giác máy tính vẫn theo dõi được.

| Thuật ngữ                           | Định nghĩa trong ngữ cảnh dự án                                                                                                                                                                                                                                                                                                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Open-vocabulary / Closed-set**    | Closed-set (tập lớp đóng, ví dụ YOLO): chỉ nhận diện được danh sách lớp cố định có trong dữ liệu huấn luyện. Open-vocabulary (từ vựng mở): không gian nhãn là toàn bộ ngôn ngữ — nhận diện được cả vật ngoài mọi danh sách định sẵn. Dự án chọn open-vocabulary vì từ mới chính là mục tiêu học tập.                                                                          |
| **Zero-shot**                       | Dùng mô hình pretrained trực tiếp, không huấn luyện lại trên dữ liệu của bài toán. Toàn bộ MVP chạy zero-shot.                                                                                                                                                                                                                                                                |
| **Florence-2**                      | Mô hình thị giác–ngôn ngữ của Microsoft, sinh nhãn dưới dạng chuỗi văn bản; một mô hình thực hiện nhiều tác vụ qua prompt: `<OD>` (phát hiện vật thể), `<DENSE_REGION_CAPTION>` (mô tả từng vùng), `<CAPTION_TO_PHRASE_GROUNDING>` (định vị cụm từ trong ảnh).                                                                                                                |
| **Bounding box**                    | Khung chữ nhật [x1, y1, x2, y2] bao quanh vật thể trong ảnh — đầu ra định vị của bộ phát hiện.                                                                                                                                                                                                                                                                                |
| **Detection Source / Clip Score**   | Điểm tin cậy được bóc tách thành `detectionSource` (nguồn phát hiện từ AI pipeline như `<OD>`, `GROUNDING`) và `clipScore` (điểm xác thực CLIP nếu bật). UI phân loại High/Medium/Low theo nguồn.                                                               |
| **Ngưỡng (threshold)**              | Mức cắt trên một điểm số: kết quả trên ngưỡng được giữ, dưới thì loại. Hệ dùng nhiều ngưỡng ở các tầng khác nhau: ngưỡng IoU 0,5 khi chấm điểm đánh giá; sàn CLIP 0,23 cho box từ vựng nền; ngưỡng diện tích mask tối thiểu của SAM (400 px). Bài học của dự án: mỗi ngưỡng là một cán cân precision ↔ recall, phải chỉnh bằng đo đạc chứ không đoán.                         |
| **SAM (Segment Anything Model)**    | Mô hình phân đoạn của Meta: nhận bounding box làm gợi ý, trả về mặt nạ (mask) tách vật khỏi nền. Trong hệ, SAM đảm nhiệm hai việc: xác thực hình học (mask quá nhỏ so với box → phát hiện sai → loại) và cắt nền trong suốt (RGBA) cho ảnh flashcard.                                                                                                                         |
| **CLIP**                            | Mô hình của OpenAI nhúng ảnh và văn bản vào cùng không gian vector, cho phép đo độ khớp giữa một ảnh cắt và câu "a photo of a {từ}". Trong hệ, CLIP là cửa xác thực: box từ vựng nền phải khớp với chính từ của nó (điểm ≥ sàn 0,23 VÀ không thua từ khớp nhất quá biên độ 0,02) mới được giữ.                                                                                |
| **Biên độ CLIP (margin)**           | Tiêu chí xác thực tương đối: thay vì chỉ dùng ngưỡng tuyệt đối, từ của một box phải khớp gần bằng từ khớp nhất trong bộ từ vựng ứng viên. Ví dụ thực tế của dự án: crop cái bát mang nhãn "frisbee" thua "bowl" 0,07 → loại; crop cái nĩa mang nhãn "fork" thua "spoon" 0,01 → giữ. Điều kiện tiên quyết: bộ ứng viên phải chứa đáp án đúng (bài học từ lỗi turkey→sandwich). |
| **IoU / IoA**                       | IoU (Intersection over Union): diện tích giao / diện tích hợp của hai box — thước đo độ trùng khớp, dùng để chấm điểm (khớp khi ≥ 0,5) và khử trùng lặp. IoA (Intersection over Area): diện tích giao / diện tích box phát hiện — dùng riêng cho group-box của Open Images.                                                                                                   |
| **NMS / WBF**                       | Các kỹ thuật khử box trùng lặp: NMS (Non-Maximum Suppression) giữ box điểm cao nhất trong nhóm box chồng nhau; WBF (Weighted Boxes Fusion) hợp nhất các box chồng nhau thành một box trung bình có trọng số.                                                                                                                                                                  |
| **Tiled OD**                        | Chạy phát hiện thêm trên 4 ô chồng lấn của ảnh (mỗi ô 60%) để bắt vật nhỏ mà một lần nhìn toàn ảnh bỏ sót (thìa, điện thoại, cốc ở góc ảnh).                                                                                                                                                                                                                                  |
| **Self-grounding**                  | Cơ chế phủ từ vựng của dự án: bảo Florence-2 tự mô tả ảnh chi tiết rồi định vị chính các cụm từ trong mô tả đó — từ vựng đến từ những gì mô hình thực sự thấy nên gần như không ảo giác.                                                                                                                                                                                      |
| **Ảo giác (hallucination)**         | Hiện tượng mô hình trả box cho từ không có trong ảnh (hỏi "elephant" trong ảnh bếp vẫn nhận được một box). Là rủi ro chính của grounding từ-áp-đặt; được kiểm soát bằng cửa CLIP.                                                                                                                                                                                             |
| **WordNet & bộ lọc danh từ cụ thể** | WordNet là cơ sở tri thức từ vựng tiếng Anh (đồng nghĩa, quan hệ thượng danh/hạ danh). Hệ dùng WordNet để: bảo đảm nhãn thuộc từ điển; loại từ trừu tượng/động từ (nghĩa phổ biến phải là vật chụp được); và so khớp nhãn ngữ nghĩa khi đánh giá (man khớp person, sedan khớp car).                                                                                           |
| **Precision / Recall / F1**         | Precision: tỷ lệ phát hiện đúng trong tổng phát hiện (đo độ sạch). Recall: tỷ lệ vật thật được tìm thấy (đo độ phủ). F1: trung bình điều hòa của hai chỉ số. Với ứng dụng học từ, precision quan trọng hơn — một thẻ sai từ tệ hơn một vật bị sót.                                                                                                                            |
| **Box-level / Word-level**          | Hai thước đo của dự án: box-level chấm từng khung theo IoU + nhãn (chuẩn học thuật); word-level chỉ hỏi "ảnh này hệ gọi tên đúng những vật gì" bỏ qua độ khít khung (sát mục tiêu sản phẩm). Số liệu hiện tại: box-F1 0,646 / word-F1 0,825 trên COCO128.                                                                                                                     |
| **Vocabulary collapse**             | Rủi ro khi fine-tune trên tập lớp hẹp: mô hình học ngầm "đáp án luôn thuộc N từ này" và teo dần khả năng gọi từ mới. Là lý do MVP giữ zero-shot và mọi thí nghiệm fine-tune phải kèm phép đo hai chiều (điểm benchmark tăng VÀ word-F1 trên bộ từ-mới không giảm quá 2 điểm).                                                                                                 |
| **LoRA**                            | Kỹ thuật fine-tune nhẹ: chỉ huấn luyện các ma trận hạng thấp gắn thêm vào mô hình, giữ nguyên trọng số gốc; adapter nhỏ (vài chục MB), gỡ ra được. Là phương án fine-tune trong hướng mở rộng.                                                                                                                                                                                |
