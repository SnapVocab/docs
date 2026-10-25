# Đặc tả UI: Thành tựu (Badges & Achievements) — Mobile App User

> **Trạng thái:** ACTIVE / AUTHORITATIVE cho visual và interaction của màn Thành tựu (MH-GAME-03), modal huy hiệu mới và phần Thành tựu trên Profile. **Freeze 2026-10-07**, sau khi Product/Design duyệt OD-1, OD-2, OD-3 và gói B1, B2, B5, B6, B8, B9.
>
> **Authority:** rule nghiệp vụ, dữ liệu và API thuộc [badges.md](../decisions/badges.md). File này chỉ quyết định cách hiển thị và tương tác, không đổi rule. Token gốc (màu, chữ, chiều sâu, motion) thuộc [design.md](../../snapVocab-frontend/docs/design.md); file này là ngoại lệ được duyệt cho §10.5 (OD-2). Bản trình bày có mockup: artifact "Đặc tả UI Thành tựu" (rev 2).

---

## 1. Concept

Màn Thành tựu là một phòng trưng bày nhỏ. Mỗi chuỗi thành tích (một `metric`) là một **tấm kệ**; các huy chương đứng trên kệ theo ngưỡng tăng dần. Huy chương đã đạt được dập nổi bằng vàng; huy chương chưa đạt là phôi phẳng màu xám, vẫn giữ hình dáng và con số. Bên trong mỗi kệ có **vạch vàng** chạy từ trái sang, lấp tới đúng `currentValue`. Đây là chỉ báo tiến độ duy nhất trên màn.

Thứ tự ưu tiên trải nghiệm: Recognition → Collection → Progress → Aspiration. Trạng thái thường gặp nhất (đã có badge, không có badge mới) **không có card và không có nút nào**.

## 2. Màn Thành tựu (`/profile/achievements`)

| Vùng | Spec |
| :--- | :--- |
| Header | `ScreenHeader` "Thành tựu" |
| Vùng cuộn | Lề ngang 16, trên 20, dưới 48 + safe area. Kéo xuống để làm mới thì gọi `POST /api/me/badges/sync` |
| Dải trạng thái (STALE hoặc RECORDING) | Cao tối thiểu 40, bo 12, chữ 14. STALE: nền `surfaceMuted`, "Đang hiển thị dữ liệu đã lưu." RECORDING: nền vàng nhạt, spinner + "Đang ghi nhận huy hiệu mới…" |
| Header bộ sưu tập | Trái "Bộ sưu tập" (Nunito 800 22). Phải `earnedCount` (Nunito 800 28, tabular) + "/`totalCount`" (Nunito 800 18, màu phụ). Từ fontScale 1,5 con số xuống dòng. Label screen reader: "Đã đạt N trên M huy hiệu" |
| Khối nổi bật | Chỉ ở trạng thái EMPTY, NEW, COMPLETE (§5). Bo 24, viền 1,5 + viền đáy 4, padding 16, cách kệ 26 |
| Danh sách kệ | Mỗi chuỗi một kệ (§4), giữa các kệ 30. Không footer, không CTA |

**Mọi chữ thường ≥ 14px.** Hai ngoại lệ có điều kiện: nhãn "Mới" 11,5px (nhãn trang trí, luôn kèm quầng sáng và label screen reader) và số trên tấm số (một phần của hình, chỉ hợp lệ vì cùng con số luôn có trong nhãn chữ ≥ 14px dưới kệ).

## 3. Huy chương (`AchievementArtifact`)

| `metric` | Hình dáng | Emblem (`iconKey`) | Kích thước quang học trong khung 64 |
| :--- | :--- | :--- | :--- |
| `LONGEST_STREAK_DAYS` | Huân chương tròn | `flame` | Đường kính 54 |
| `QUIZ_COMPLETED` | Khiên | `clipboard-check` | 50 × 57 |
| `PERFECT_QUIZ` | Dấu ấn 8 cánh | `target` | Bán kính 29 / 25,5 |
| `DISTINCT_WORDS_ADDED` | Lục giác đỉnh nhọn | `book-open` | Bán kính 28,5 |
| Metric lạ | Tròn | `award` | Đường kính 54 |

`iconKey` không nhận ra thì giữ hình dáng theo metric và dùng emblem `award`.

| Lớp | Đã đạt (dập nổi) | Chưa đạt (phôi) | Kế tiếp | Đang ghi nhận | Mới |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Lớp đáy | `#C98A00`, lệch xuống 3,5 | Không có (phẳng) | Như chưa đạt | Như chưa đạt | Như đã đạt |
| Thân | Gradient `#FFE48A → #FFC42E → #F0A800`, viền 1,5 `#E0A000`, vòng trong 1px `#FFE48A` | `#E8ECF2`, viền 1,5 `#7C889C` | Như chưa đạt | Viền 1,5 nét đứt `#A87400` | Như đã đạt + quầng sáng 2s |
| Mặt | `#FFF3C4` | `#F1F5F9` | Như chưa đạt | Như chưa đạt | Như đã đạt |
| Emblem | `#8A5A00` (5,33:1), nét 2,6; 3,2 khi < 48px | `#64748B` (4,34:1) | Như chưa đạt | Như chưa đạt | Như đã đạt |
| Tấm số (≥ 56px) | Nền `#1E2A44`, chữ trắng | Nền trắng, viền `#7C889C`, chữ `#64748B` | Viền 2 `#A87400`, chữ `#8A5A00` | Như kế tiếp | Như đã đạt + nhãn "Mới" |

**Dark mode:** vàng giữ nguyên; phôi: thân `#24263D`, viền `#5A5F86` (3,22:1), mặt `#2A2C47`, emblem `#8A90AD` (4,7:1), tấm số nền `#171A2F` chữ `#9497A8`.

**Kích thước:** 40 (dải Profile, kệ thu nhỏ; ẩn tấm số) · 64 (kệ) · 88 (khối nổi bật) · 112 (sheet) · 128 / 80 (modal một / nhiều badge).

**Render:** SVG (`react-native-svg`) trong `GameVisual`, id gradient riêng mỗi instance. Không thay bằng `AchievementBadge3D` (dựng theo rarity, trái PD-BDG-09).

## 4. Kệ (`AchievementShelf`)

- **Dữ liệu:** `AchievementSeries { metric, title, unit, currentValue, items[] }`. Không có logic cố định 3 huy chương.
- **Tấm kệ:** cao 13 (mặt 9 + viền đáy 4), bo tròn, không phối cảnh. Light: mặt `#E3E8EF`, viền đáy `#CDD5E0`. Dark: `#23263F` / `#1A1C30`. Đáy tấm số chồng lên mặt kệ khoảng 6px.
- **Vạch vàng:** cao 5 trong mặt kệ, `#FFC42E`, viền 1,5 `#A87400` (3,3:1 so với kệ). Tính theo `currentValue`, không theo `earned`. Với `c[i]` là tâm item i: nếu `currentValue ≥ threshold` cuối thì vạch kéo hết kệ; ngược lại lấy `j` là item đầu tiên có `threshold > currentValue`, `prev = j == 0 ? 0 : threshold[j-1]`, `start = j == 0 ? mép trái : c[j-1]`, vị trí = `start + (c[j] − start) × (currentValue − prev) / (threshold[j] − prev)`.
- **Nhãn dưới kệ:** tên badge (Nunito 700 14/18, ≤ 2 dòng ở 100%, không cắt ở cỡ chữ lớn) + dòng phụ (Inter 600 14/18): đã đạt "✓ {ngưỡng} {đơn vị}", kế tiếp "{min(cur,thr)}/{ngưỡng} {đơn vị}" màu `#8A5A00`, xa hơn "{ngưỡng} {đơn vị}" màu phụ, đang ghi nhận "Đang ghi nhận…" (offline: "Đủ điều kiện").
- **Tên kệ:** trái tên chuỗi (Nunito 800 16); phải "{đã đạt}/{tổng} đã đạt" (Inter 600 14) hoặc "✓ Hoàn thành".
- **Bố cục theo số item:** 1 → một ô rộng 50% căn giữa, kệ 50%; 2 hoặc 3 → các cột bằng nhau; ≥ 4 → ray cuộn ngang, ô 104, bắt dính theo ô, mép phải mờ, tự cuộn để mốc kế tiếp vào giữa.
- **Vùng chạm:** cả ô (huy chương + nhãn). **Screen reader:** mỗi ô một câu "{tên}. {Đã đạt ngày … | Chưa đạt, X trên Y {đơn vị} | Đang ghi nhận}. Mốc i trên n của {chuỗi}". Kệ và vạch ẩn.

**Bảng ánh xạ ở client theo `metric`:**

| `metric` | Tên chuỗi | Đơn vị | Ghi chú trong sheet | CTA (cấp ba) |
| :--- | :--- | :--- | :--- | :--- |
| `LONGEST_STREAK_DAYS` | Chuỗi ngày học | ngày | Tính theo chuỗi dài nhất; mất chuỗi không làm mất tiến độ | "Làm một bài quiz ›" → `/study/quiz-setup` |
| `QUIZ_COMPLETED` | Quiz hoàn thành | quiz | Không | "Làm một bài quiz ›" → `/study/quiz-setup` |
| `PERFECT_QUIZ` | Điểm tuyệt đối | lần | Quiz cùng chủ đề vẫn được tính | "Làm một bài quiz ›" → `/study/quiz-setup` |
| `DISTINCT_WORDS_ADDED` | Vốn từ tự xây | từ | Chỉ tính các từ thêm sau ngày ra mắt huy hiệu; mỗi từ tính một lần (bắt buộc, §2.2 badges.md) | "Thêm từ mới ›" → `/collections` |
| Khác | Thành tựu khác | (rỗng) | Không | Không |

### 4.1. Contract thứ tự (B8)

| Phạm vi | Khoá sắp xếp |
| :--- | :--- |
| Item trong series | `threshold` ASC, `sortOrder` ASC, `id` ASC |
| Các series | `min(sortOrder)` ASC, rồi thứ tự xuất hiện đầu tiên của `metric` trong response |
| 3 badge trên Profile | `earnedAt` DESC, `sortOrder` DESC, `id` DESC |
| Modal, khối NEW | Thứ tự series, rồi thứ tự item |

Thứ tự của response không được dùng trực tiếp để hiển thị.

## 5. Trạng thái màn (B9)

Ưu tiên: `NEW → COMPLETE → RECORDING → EMPTY → NORMAL`. `STALE` là lớp phủ (dải trạng thái) trên mọi trạng thái; khi `STALE` thì `RECORDING` tắt.

| Trạng thái | Điều kiện | Đầu màn |
| :--- | :--- | :--- |
| NEW | Có id trong `introducedIds` (§7.2) hoặc `highlight` | Card ghi nhận: huy chương 88 (quầng sáng), "Vừa đạt", tên, "Mốc i/n · {chuỗi}". Nhiều badge: ≤ 3 huy chương xếp chồng + "N huy hiệu mới". Chạm → sheet. Không CTA, không Snapy |
| COMPLETE | `earnedCount == totalCount > 0` | Card: Snapy `tu_hao` tĩnh, "Bạn đã hoàn thành cả bộ sưu tập.", hàng huy chương cuối mỗi chuỗi (40). Không CTA |
| RECORDING | Có item `!earned && currentValue >= threshold`, không STALE | Không card; dải "Đang ghi nhận huy hiệu mới…". Không CTA |
| EMPTY | `earnedCount == 0` | Card: Snapy `to_mo` tĩnh, "Huy hiệu đầu tiên đang chờ bạn", "{tên} · còn N {đơn vị}" theo `highestProgressRatio`, link cấp ba theo metric |
| NORMAL | Còn lại | Không card, không nút |

`highestProgressRatio`: trong các mốc kế tiếp chưa đủ ngưỡng, chọn `min(cur,thr)/thr` cao nhất; hoà thì `threshold − cur` nhỏ hơn, rồi thứ tự series. Thuật toán không đo công sức; câu chữ chỉ nói "còn N {đơn vị}".

Trạng thái khác: đang tải lần đầu → skeleton (header + 3 kệ xám); lỗi không cache → card Snapy `suy_nghi` "Không tải được huy hiệu. Kiểm tra mạng rồi thử lại nhé." + "Thử lại"; danh sách rỗng → "Huy hiệu đang được chuẩn bị. Quay lại sau nhé." + "Thử lại"; sync lỗi → im lặng, giữ dữ liệu hiện có.

## 6. Sheet chi tiết

- Khung theo `PreviewSheet` của Shop: bo 24 hai góc trên, cao theo nội dung (tối đa 90%), nút đóng (vùng chạm 48), đóng bằng vuốt, nền mờ, back.
- Huy chương 112; tên (Nunito 800 22, ≤ 3 dòng); `description` (Inter 15–16, màu phụ).
- Khối trạng thái: đã đạt "✓ Đạt ngày {d} tháng {m}, {y}" (GMT+7); đang có tiến độ "{cur} / {thr} {đơn vị}" (Nunito 800 26) + "Còn {thr−cur} {đơn vị}"; tiến độ 0 → "Chưa bắt đầu"; đang ghi nhận "Đã đủ điều kiện" + spinner "Đang ghi nhận…".
- Kệ thu nhỏ của chính chuỗi đó (huy chương 40, nhãn là ngưỡng, item đang xem có vòng focus navy 2px); chỉ để xem, ẩn với screen reader.
- Ghi chú theo bảng §4 (tối đa 2 câu, nền `surfaceMuted`).
- CTA cấp ba theo bảng §4, **chỉ** khi chưa đạt và chưa đủ ngưỡng. Sheet không có nút chính.

## 7. Huy hiệu mới

### 7.1. Modal (`BadgeUnlockModal`)

| Biến thể | Khi nào | Nội dung, cường độ |
| :--- | :--- | :--- |
| Thường | 1 badge, không phải mốc cuối | Pill "Huy hiệu mới", huy chương 128, tên, mô tả, "Mốc i/n · {chuỗi}", "Xem bộ sưu tập" (primary) + "Để sau" (ghost). Scale + quầng sáng + 3 hạt lấp lánh |
| Mốc cuối chuỗi | `threshold` lớn nhất của series **và** series ≥ 2 item | Pill "Hoàn thành chuỗi", "Bạn đã hoàn thành {chuỗi}.", Snapy `tu_hao`. Thêm `jump_celebrate` + `confetti.json` một lần (P2) |
| Nhiều badge | ≥ 2 id trong lần hiện | "Bạn có N huy hiệu mới", ≤ 3 huy chương 80 lệch 80ms, danh sách tên + chuỗi, "+N huy hiệu khác". Cường độ theo item cao nhất |
| Giảm | Mở từ dòng nối tiếp trên Quiz Result | Như trên nhưng không Snapy, không confetti |

Khung theo `PurchaseSuccessModal`: bo 32, viền 2 + viền đáy 6, rộng ≤ 350, spring 0.85→1. Đóng bằng nút nào hoặc back đều gửi `POST /seen` với đúng các id đã hiện; lỗi → `pendingSeenIds`, gửi lại ở lần sync thành công kế tiếp, không hiện lại trong phiên. "Xem bộ sưu tập" → `/profile/achievements?highlight=id1,id2`.

**Host (`BadgeUnlockHost`, gắn ở `(app)/_layout.tsx`):** `pendingIds` = item `seen === false` chưa thuộc `shownIds`. Modal mở khi: route hiện tại không thuộc denylist, `pendingIds` khác rỗng, không có modal khác đang mở. Kiểm tra lại mỗi khi route đổi và mỗi khi cache `['badges']` đổi. **Denylist:** `/study/quiz-session`, route phiên flashcard (khi có), `/study/quiz-result`, `/profile/achievements`. Hệ quả: sau quiz, modal hiện ở route hợp lệ đầu tiên sau khi rời Quiz Result (OD-1).

**Quiz Result:** khi có badge chưa xem, hiện một dòng nối tiếp dưới phần XP (huy chương 40, "Huy hiệu mới: {tên}" hoặc "N huy hiệu mới", "Xem ›"). Chạm → modal cường độ giảm ngay trên Quiz Result.

### 7.2. Giới thiệu ngay trên màn Thành tựu (OD-3)

- Trên `/profile/achievements` không bật modal; badge chưa xem được giới thiệu bằng trạng thái NEW.
- `introducedIds` = tập id chưa xem **đã được trạng thái NEW render**. Snapshot đầu tiên lấy khi lượt sync lúc vào màn kết thúc (thành công, lỗi, hoặc sau 3 giây thì dùng cache). Mỗi lượt sync sau trong cùng lần vào màn (kéo làm mới, quay lại foreground) mang về id chưa xem mới thì trạng thái NEW render lại để gồm chúng, và chúng được thêm vào tập.
- Khi rời màn (blur hoặc app vào background): `POST /seen(introducedIds chưa gửi)`. Không bao giờ gửi "mọi id chưa xem lúc rời màn".
- Id từ `highlight` đã được modal đánh dấu nên không gửi lại.

## 8. Profile

- Dòng menu "Thành tựu": hoạt động, dòng phụ "{earnedCount}/{totalCount} huy hiệu", chấm đỏ 11px ở góc ô icon khi `unseenCount > 0`. Label: "Thành tựu, N trên M huy hiệu[, có huy hiệu mới]".
- Dải 3 badge gần nhất trong thẻ hồ sơ (dưới email và ngày tham gia, ngăn bằng đường kẻ): huy chương 40 không tấm số, cách nhau 10, + "Xem tất cả ›". Cả dải là một vùng chạm → `/profile/achievements`. Ẩn khi chưa có badge.

## 9. Sync (B1)

Gọi `POST /api/me/badges/sync` khi: mở app (sau đăng nhập), app quay lại foreground, `useCompleteQuiz` thành công, `useAddTopicItem` thành công, **vào màn Thành tựu**, kéo làm mới trên màn Thành tựu. Gọi tường minh trong các mutation, không qua `invalidateProgress` (hàm này còn chạy khi xoá collection, làm mới Home/Stats). Thành công → `setQueryData(['badges'])`; lỗi → im lặng.

## 10. Motion

| Lớp | Tương tác | Chuyển động | Thời lượng | Reduced motion |
| :--- | :--- | :--- | :--- | :--- |
| Chức năng | Chạm ô | Scale 0.96 | 120ms ease-out | Độ mờ 0.7 |
| Chức năng | Sheet | Trượt + nền mờ | 280 / 240ms | Fade 120ms |
| Chức năng | Skeleton → nội dung | Cross-fade | 200ms | Thay ngay |
| Tiến độ | Vạch vàng | Chiều rộng 0 → giá trị, một lần mỗi lần vào màn (không dùng `scaleX` vì làm méo viền bo tròn) | 480ms ease-out | Vẽ ngay |
| Ghi nhận | Badge mới (kệ, khối NEW) | Scale 0.85→1, quầng sáng mờ dần, nhãn "Mới" hiện | 480ms spring, quầng 2000ms | Quầng tĩnh 600ms |
| Ghi nhận | Modal | Scale 0.6→1 + 3 hạt | 480ms spring | Hiện ngay |
| Ăn mừng | Mốc cuối chuỗi | Snapy `jump_celebrate` + confetti | 800 / 1000ms, một lần | Star burst tĩnh 600ms |

Ngoài vạch vàng, chỉ animate `transform` và `opacity`; không chuyển động lặp vô hạn. Không có hiệu ứng lần lượt hiện kệ khi vào màn.

## 11. Ngoài phạm vi

Rarity, tier, thưởng, nút nhận, chia sẻ, ghim, bộ lọc, tìm kiếm, badge bí mật, leaderboard, push, âm thanh, engine 3D. Haptics (`expo-haptics`) và chạm huy chương trên Profile để mở thẳng sheet: giai đoạn 2.
