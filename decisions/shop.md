# Architecture Decision: Shop — ShopItem, UserItem, Equip, XP Booster, Coin Purchase

> **Trạng thái:** APPROVED (Product decisions 2026-09-24; audit Shop/Inventory 2026-09-24).
> **Implementation:** Backend learner API (§10.1) IMPLEMENTED 2026-09-24 — `shop/` (ShopController, ShopService, InventoryService, WalletService, ShopRewardService) + `xp/XpService`. Backend Admin API (§10.2) IMPLEMENTED 2026-09-25 — `AdminShopController`, `AdminShopService`. Chưa có: Admin frontend nối API thật, `GET /api/me` bổ sung `equipped`/`activeBooster`, caller thật cho XpService/ShopRewardService (Review, Quiz M4, Mission, Chest). Admin CMS và mobile Shop/Inventory hiện vẫn là **mock** (xem §13).
> **Traceability:** FR-09.06, FR-09.07 ([specs.md](../spec/specs.md)) → BF-12 ([buss_mainflow.md](../spec/buss_mainflow.md)) → SS-13, SS-14 ([phan_ra_phan_he_he_thong.md](../spec/phan_ra_phan_he_he_thong.md)) → F-GAME-02, F-GAME-07, F-GAME-08a, F-GAME-08b, F-GAME-09 ([phan_ra_tinh_nang.md](../spec/phan_ra_tinh_nang.md)) → DB ([database.md](../db/database.md) §Gamification & Shop) → Screen MH-ECONOMY-01/02/03, MH-PROFILE-01/03 ([phan_ra_man_hinh.md](../spec/phan_ra_man_hinh.md)).
> **Source of truth:** Nếu tài liệu khác mâu thuẫn với file này về Shop/Item/Inventory/Coin purchase/XP Booster, file này thắng; [specs.md](../spec/specs.md) chỉ giữ requirement tóm tắt và trỏ về đây.

---

## 1. Quyết định

| # | Quyết định | Nội dung canonical |
| :--- | :--- | :--- |
| S1 | Tiền tệ | Chỉ **Coin** nội bộ. Không Gems, không thanh toán tiền thật. |
| S2 | Item types | `ShopItemType` = `THEME` \| `AVATAR_FRAME` \| `XP_BOOSTER`. Enum do developer kiểm soát. Không có `BOOSTER` chung, `CONSUMABLE`, `STREAK_FREEZE`, `COIN_BOOSTER`, `RETRY_TOKEN`. |
| S3 | Type vs Effect | **Không** tách `ItemEffectType`. Mỗi type là một runtime behavior do developer viết code; Admin chỉ cấu hình tham số trong giới hạn của type. |
| S4 | Ownership | `UserItem` = bản ghi **sở hữu**: 1 row / (user, shopItem), có `quantity`. Không phải instance. "Inventory" chỉ là tên API resource/màn hình, **không** phải entity. |
| S5 | Cosmetic | `THEME`, `AVATAR_FRAME`: `quantity` luôn = 1; mua lại → `ITEM_ALREADY_OWNED`. |
| S6 | XP Booster | ×2 XP (hằng số developer), `durationMinutes` cấu hình theo item (mặc định 30). Áp dụng cho XP từ **Flashcard/SRS Review** và **Quiz**. Không áp dụng cho Mission, Daily/Weekly Chest, Admin adjustment, Save word hay reward thụ động. Bonus XP **có** tính vào Weekly XP/Leaderboard. |
| S7 | Booster concurrency | Tối đa **1** XP Booster active/user. Không stack, không queue, không extend. Kích hoạt khi đang active → `BOOSTER_ALREADY_ACTIVE`, không trừ item. |
| S8 | Booster time | Wall-clock theo server time; vẫn trôi khi app đóng. Hết hạn **lazy** (`now >= expiresAt`), không cron, không cột status. |
| S9 | Inventory limit | `XP_BOOSTER` có `maxQuantity` cấu hình theo item (mặc định 5, giới hạn developer 1–10). Mua khi đã đạt → `MAX_QUANTITY_REACHED`. |
| S10 | Equip | Trạng thái equip nằm trên `UserItem.equipped`. Mỗi user tối đa 1 `THEME` và 1 `AVATAR_FRAME` equipped; equip item mới cùng type tự unequip item cũ **trong cùng transaction**. Enforce ở DB bằng `UNIQUE(user_id, equipped_slot)` (§9). Không thêm cột equip vào `users`. |
| S11 | Theme | Theme definition nằm trong **mobile code** (design tokens + theme registry). Theme chỉ override token bề mặt (§6). Admin chỉ chọn `themeKey` có sẵn. |
| S12 | Avatar Frame | Asset ảnh do Admin upload (PNG/WebP nền trong suốt, vuông 512×512, ≤ 300 KB). Hành vi runtime chung cho mọi frame: overlay lên avatar. |
| S13 | Purchase | 1 đơn vị / request. Header `Idempotency-Key` bắt buộc; body `expectedPrice`. Trừ Coin + ghi `CoinTransaction` + cấp `UserItem` trong **một** DB transaction (§7.1). |
| S14 | Purchase history | Không có bảng Purchase/Order riêng. `CoinTransaction` (source `SHOP_PURCHASE`) là biên lai và anchor idempotency; Wallet hiển thị lịch sử. |
| S15 | Server authority | Giá, quyền sở hữu, số dư, booster active và XP bonus do Backend quyết định. Client không gửi owner, price thực tế, multiplier hay thời gian. |
| S16 | ShopItem lifecycle | `DRAFT → PUBLISHED → ARCHIVED`. `type`, `code` và field hiệu ứng bị khóa sau publish. Bật/tắt bán = `purchasable`, không đổi status. |
| S17 | Reward item | Reward (mission/chest) tham chiếu **một** `shopItem.code` cụ thể. Không cấp được (cosmetic đã sở hữu hoặc booster đã đạt `maxQuantity`) → cộng `fallbackCoin` lấy từ **reward configuration**, không hard-code trong Shop service. |
| S18 | Quyền Admin | `ROLE_ADMIN`. Không có permission riêng kiểu `SHOP_MANAGE`. |
| S19 | API base path | `/api` (như [quiz.md](./quiz.md) D1). Catalog: `/api/shop/...`; dữ liệu cá nhân: `/api/me/...`; quản trị: `/api/admin/shop/...`. |

**Ngoài scope hiện tại** (không implement, không đưa vào schema/API/mock): Streak Freeze (chỉ thêm lại khi streak settlement/lifecycle có đặc tả riêng), Gems, rarity, stock toàn cục, flash sale/discount, coin booster, retry token, hint/heart/timer/magnet/refresh item, cosmetic cho Snapy, mua nhiều đơn vị một lần, refund, hiển thị frame của người khác trên Leaderboard.

---

## 2. Type system

### 2.1. `ShopItemType` (developer-controlled)

| Type | Behavior | Stackable | Equip slot | Field cấu hình bắt buộc | Runtime |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `THEME` | EQUIP | Không | `THEME` | `themeKey` | Mobile áp theme từ registry |
| `AVATAR_FRAME` | EQUIP | Không | `AVATAR_FRAME` | `frameAssetKey` | Mobile overlay frame lên avatar |
| `XP_BOOSTER` | ACTIVATE | Có | — | `boostDurationMinutes`, `maxQuantity` | Backend nhân XP ×2 trong thời gian hiệu lực |

```java
enum ShopItemType {
    THEME(Behavior.EQUIP, false),
    AVATAR_FRAME(Behavior.EQUIP, false),
    XP_BOOSTER(Behavior.ACTIVATE, true);

    final Behavior behavior;   // không lưu DB
    final boolean stackable;   // không lưu DB
}
enum Behavior { EQUIP, ACTIVATE }
```

- "Cosmetic" = `behavior == EQUIP`; "Consumable" = `behavior == ACTIVATE`. Đây là nhãn UI suy ra từ type, **không** là cột hay type riêng.
- Thêm type mới = thay đổi code (backend + mobile) + cập nhật tài liệu này. Admin không thể tạo type.

### 2.2. Enum khác

```java
enum ShopItemStatus { DRAFT, PUBLISHED, ARCHIVED }                 // developer-controlled
enum ThemeKey { /* registry, đồng bộ với mobile theme registry */ } // developer-controlled
enum CoinSourceType { MISSION_REWARD, DAILY_CHEST, WEEKLY_CHEST, SHOP_PURCHASE, ADMIN_ADJUSTMENT }
enum XpSourceType   { FLASHCARD_REVIEW, QUIZ, SAVE_WORD, MISSION_REWARD, DAILY_CHEST, WEEKLY_CHEST, ADMIN_ADJUSTMENT }
```

- Booster chỉ áp dụng khi `XpSourceType ∈ {FLASHCARD_REVIEW, QUIZ}`.
- `ThemeKey` khởi đầu 2–3 giá trị (tên và palette do design chốt trong `design.md`); giá trị `DEFAULT` là theme mặc định của app, **không** phải ShopItem.
- Không có enum `EquipSlot`: slot = `ShopItemType` có `behavior == EQUIP`.

### 2.3. Giới hạn tham số (developer-defined)

| Tham số | Giới hạn | Mặc định |
| :--- | :--- | :--- |
| `price` | 1 – 100 000 Coin | — |
| `boostDurationMinutes` | 5 – 180 | 30 |
| `maxQuantity` (XP_BOOSTER) | 1 – 10 | 5 |
| XP multiplier | cố định ×2 (hằng số code) | — |
| Frame asset | PNG/WebP, alpha, 512×512, ≤ 300 KB | — |
| Icon | PNG/WebP, vuông ≥ 256×256, ≤ 200 KB | — |

---

## 3. Domain model

```text
ShopItem 1 ─── N UserItem N ─── 1 User
ShopItem 1 ─── N BoosterActivation N ─── 1 User
User     1 ─── N CoinTransaction
```

### 3.1. ShopItem (aggregate root của catalog)

**Purpose:** Vật phẩm do Admin tạo trong phạm vi một `ShopItemType`.

```text
id, code, type, name, description, iconKey,
status, purchasable, price, availableFrom, availableUntil, sortOrder,
themeKey            (chỉ THEME)
frameAssetKey       (chỉ AVATAR_FRAME)
boostDurationMinutes, maxQuantity   (chỉ XP_BOOSTER)
publishedAt, version, createdAt, updatedAt
```

Business rules:

- `code` unique, dạng `UPPER_SNAKE_CASE` (VD `XP_BOOSTER_X2_30M`, `FRAME_GOLDEN_WORDSMITH`, `THEME_OCEAN`); sửa được khi `DRAFT`, bất biến sau publish. Reward tham chiếu item bằng `code`.
- Field cấu hình phải khớp type: field của type khác phải `NULL`.
- `purchasable = true` ⇒ `price` NOT NULL.
- Hiển thị trong Shop ⇔ `status = PUBLISHED` ∧ `purchasable` ∧ (`availableFrom` null hoặc ≤ now) ∧ (`availableUntil` null hoặc > now).
- `PUBLISHED` + `purchasable = false` = item chỉ nhận qua reward.
- `themeKey` unique giữa các ShopItem (một theme chỉ bán bằng một item).

### 3.2. UserItem

**Purpose:** Quyền sở hữu một ShopItem của một user, số lượng và trạng thái equip.

```text
id, userId, shopItemId, itemType (copy bất biến từ ShopItem.type),
quantity, equipped, acquiredAt, updatedAt
```

Business rules:

- UNIQUE(`userId`, `shopItemId`).
- Cosmetic: `quantity = 1`. XP_BOOSTER: `0 ≤ quantity ≤ maxQuantity` tại thời điểm cộng thêm.
- `equipped` chỉ có thể `true` với cosmetic; tối đa 1 row `equipped = true` cho mỗi (`userId`, `itemType`).
- `quantity = 0` → giữ row (không xóa); Inventory ẩn row nếu không có booster active của item đó.
- `acquiredAt` = lần nhận đầu tiên (mua hoặc reward).
- Item `ARCHIVED` vẫn thuộc sở hữu, vẫn equip/activate được.

### 3.3. BoosterActivation

**Purpose:** Một lần kích hoạt XP Booster. Là nguồn xác định booster active, anchor idempotency và lịch sử sử dụng.

```text
id, userId, shopItemId, startedAt, expiresAt, idempotencyKey
```

Business rules:

- Active ⇔ `startedAt ≤ t < expiresAt` (server time). Không có cột status.
- Tại mọi thời điểm, mỗi user có tối đa 1 activation active (enforce bằng lock user row, §7).
- `expiresAt = startedAt + ShopItem.boostDurationMinutes` (snapshot khi kích hoạt; sửa item sau đó không ảnh hưởng).

### 3.4. CoinTransaction (ledger — thuộc SS-13, Shop dùng)

```text
id, userId, amount (có dấu), balanceAfter, sourceType, referenceId, description, eventKey, createdAt
```

Business rules:

- Append-only. `amount > 0` nhận, `< 0` tiêu. Không có cột `type`.
- `users.coin` là số dư hiện hành; invariant `users.coin = Σ amount` và `balanceAfter ≥ 0`.
- `eventKey` unique toàn bảng. Mua hàng: `SHOP_PURCHASE:{userId}:{Idempotency-Key}`, `referenceId = shopItemId`, `description` = tên item tại thời điểm mua.

### 3.5. Không phải entity

| Khái niệm | Thực tế |
| :--- | :--- |
| Inventory / UserInventory | API resource `/api/me/items` + màn hình MH-ECONOMY-03 |
| Purchase / Order | `CoinTransaction` source `SHOP_PURCHASE` |
| ItemEffect | Không tồn tại; behavior gắn với `ShopItemType` |
| Theme definition | Mobile theme registry |
| Item ledger (nhập/xuất item) | Không cần: mua → CoinTransaction; dùng → BoosterActivation; reward → claim log của mission/chest |

---

## 4. Lifecycle

### 4.1. ShopItem

```text
DRAFT ──publish──► PUBLISHED ──archive──► ARCHIVED (terminal)
  │
  └──delete (hard delete, chỉ khi DRAFT)
```

| Hành động | Điều kiện | Kết quả |
| :--- | :--- | :--- |
| Create | Admin | `DRAFT`, `purchasable = false`, `version = 0` |
| Update | `DRAFT`: mọi field. `PUBLISHED`: chỉ field mở (§8.2) | `version + 1` |
| Publish | `DRAFT`; validate đầy đủ (§8.3) | `PUBLISHED`, `publishedAt = now` |
| Archive | `PUBLISHED` | `ARCHIVED`, `purchasable = false`; không bán, không chọn được cho reward mới; ownership, equip, activation, reward đã cấu hình trước đó giữ nguyên |
| Delete | `DRAFT` | Xóa cứng (chưa ai sở hữu vì DRAFT không thể mua/nhận) |

Không có unarchive. Muốn bán lại → tạo item mới.

### 4.2. UserItem (trạng thái suy ra, không lưu status)

| Trạng thái hiển thị | Điều kiện |
| :--- | :--- |
| Owned | row tồn tại, `quantity > 0` |
| Equipped | `equipped = true` |
| Active (booster) | tồn tại `BoosterActivation` của item với `expiresAt > now` |
| Hết | XP_BOOSTER `quantity = 0` và không active → ẩn khỏi Inventory |

### 4.3. XP Booster

```text
Purchase / Reward ──► UserItem.quantity +1
        │
Activate ──► quantity −1, BoosterActivation(startedAt, expiresAt)
        │
Effect active (server time < expiresAt) ──► XP REVIEW/QUIZ ×2
        │
Expire (lazy, now ≥ expiresAt)
```

---

## 5. XP Booster — quy tắc tính XP

1. Tại thời điểm XpService **ghi** XP (server time `awardedAt`), tìm activation active của user.
2. Nếu có activation và `sourceType ∈ {FLASHCARD_REVIEW, QUIZ}`: `bonusAmount = baseAmount` (×2 ⇒ bonus bằng base); ngược lại `bonusAmount = 0`.
3. Ghi `ExperienceLog(baseAmount, bonusAmount, amount = base + bonus, boosterActivationId)`; cộng `users.exp` và Weekly XP bằng `amount` (bonus **có** tính Leaderboard).
4. Thời điểm xét là thời điểm ghi XP, không phải thời điểm bắt đầu hoạt động. Quiz ghi XP khi `complete` ([quiz.md](./quiz.md) D7): bắt đầu khi booster còn hạn nhưng complete sau `expiresAt` → không bonus.
5. Idempotency của XP theo `eventKey` hiện có (FR-09); replay không tính bonus lần nữa.
6. Mobile chỉ hiển thị `bonusAmount` do Backend trả; không tự nhân.

---

## 6. Theme

### 6.1. Kiến trúc

```text
Design tokens (snapVocab-frontend/docs/design.md §03)
  → ThemeDefinition(themeKey, light, dark) trong mobile theme registry
  → ThemeProvider (đọc themeKey equipped; fallback DEFAULT)
  → Component chỉ đọc semantic token
```

### 6.2. Token

| Được theme override | Cố định (không theme nào được đổi) |
| :--- | :--- |
| `background`, `card`, `muted`, `border`, `input` | `foreground`, `card-foreground`, `muted-foreground` |
| `header-background`, `header-border`, hero/header gradient | `primary-*` (CTA, đúng, progress) |
| `tab-bar-background`, `tab-bar-border`, `tab-bar-inactive` | `reward-*` (XP, Coin, Chest, Badge) |
| | `mascot-*` (Snapy, streak) |
| | `error-soft`/sai, `danger-*`, `warning-*`, `info-*`, `destructive`, `ring` |

- Mỗi ThemeDefinition phải khai báo cả `light` và `dark`.
- Accessibility do developer bảo đảm lúc build: test tự động contrast ≥ 4.5:1 của `foreground`/`muted-foreground` trên `background`, `card`, `muted`, `header-background`, `tab-bar-background` cho mọi theme × mode. Admin không chạm được token nên không thể phá contrast.
- Theme **không** thay thế chế độ Sáng/Tối/Hệ thống; hai trục độc lập.

### 6.3. Tương thích phiên bản app

- `GET /api/shop/items` trả `themeKey`; mobile ẩn item THEME có `themeKey` không có trong registry của bản app đang chạy.
- Inventory: THEME đã sở hữu nhưng app không biết `themeKey` → hiện "Cần cập nhật ứng dụng", CTA disabled; nếu nó đang equipped → áp `DEFAULT`.
- Quy trình phát hành theme mới: thêm ThemeDefinition vào mobile → phát hành app → thêm giá trị vào `ThemeKey` backend → Admin tạo item.

---

## 7. Transaction, idempotency, concurrency

### 7.1. Quy tắc chung

- Mọi thao tác thay đổi economy của một user (purchase, activate, equip/unequip, reward grant, cộng/trừ coin) mở transaction và **lock row `users` trước tiên** (`SELECT … FOR UPDATE`). Thứ tự lock duy nhất: `users` → `user_items` → `coin_transactions`/`booster_activations`. Tránh deadlock và tuần tự hóa thao tác cùng user.
- `Idempotency-Key`: header bắt buộc cho purchase và activate, 1–128 ký tự (cùng quy ước [quiz.md](./quiz.md)); thiếu → `IDEMPOTENCY_KEY_REQUIRED` (400).
- Chỉ request **thành công** mới lưu key. Request bị từ chối nghiệp vụ (VD `INSUFFICIENT_COINS`) không lưu key; gửi lại cùng key sẽ được đánh giá lại.
- Replay trả HTTP 200, cùng shape response với `replayed = true`, phản ánh trạng thái **hiện tại** (balance, quantity).
- Domain event (`SHOP_ITEM_PURCHASED`, `BOOSTER_USED`) phát **sau commit** và không phát khi replay.

### 7.2. Purchase

`POST /api/shop/items/{itemId}/purchase`, header `Idempotency-Key`, body `{ "expectedPrice": 250 }`.

```text
BEGIN
1. SELECT user FOR UPDATE
2. tx = coin_transactions WHERE event_key = SHOP_PURCHASE:{userId}:{key}
     tx tồn tại ∧ tx.reference_id = itemId → COMMIT, trả replay
     tx tồn tại ∧ khác itemId              → 409 IDEMPOTENCY_KEY_REUSED
3. item không tồn tại / DRAFT              → 404 SHOP_ITEM_NOT_FOUND
   ¬PUBLISHED ∨ ¬purchasable ∨ ngoài sale window → 409 ITEM_NOT_AVAILABLE
4. item.price ≠ expectedPrice              → 409 PRICE_CHANGED {currentPrice}
5. cosmetic ∧ đã sở hữu                    → 409 ITEM_ALREADY_OWNED
   XP_BOOSTER ∧ quantity ≥ maxQuantity     → 409 MAX_QUANTITY_REACHED
6. UPDATE users SET coin = coin - price WHERE id = ? AND coin >= price
     0 row                                 → 409 INSUFFICIENT_COINS
7. INSERT coin_transactions(amount = -price, balance_after, SHOP_PURCHASE, reference_id = itemId, description = item.name, event_key)
8. UPSERT user_items (insert quantity = 1 | quantity + 1)
COMMIT → publish SHOP_ITEM_PURCHASED
```

| Tình huống | Hành vi |
| :--- | :--- |
| Double tap / retry / timeout / server thành công nhưng mobile mất response | Mobile gửi lại **cùng key** → replay, không trừ coin lần 2 |
| Hai request cùng key chạy song song | Lock user tuần tự hóa; request sau thấy tx ở bước 2 → replay. UNIQUE `event_key` là lớp phòng thủ cuối (vi phạm → rollback, đọc lại, trả replay) |
| Mua song song hai item khác nhau, tổng giá > balance | Tuần tự hóa; request sau nhận `INSUFFICIENT_COINS` |
| Lỗi bất kỳ giữa bước 6–8 | Rollback toàn bộ; không thể có "trừ coin mà không cấp item" |
| Item bị archive / tắt bán / hết sale window khi đang xác nhận | `ITEM_NOT_AVAILABLE` |
| Admin đổi giá khi learner đang xác nhận | `PRICE_CHANGED`; mobile hiện giá mới, yêu cầu xác nhận lại với **key mới** |

### 7.3. Activate XP Booster

`POST /api/me/items/{itemId}/activate`, header `Idempotency-Key`.

```text
BEGIN
1. SELECT user FOR UPDATE
2. act = booster_activations WHERE user_id ∧ idempotency_key
     tồn tại ∧ act.shop_item_id = itemId → replay; khác → 409 IDEMPOTENCY_KEY_REUSED
3. user_item không tồn tại               → 404 ITEM_NOT_OWNED
   type ≠ XP_BOOSTER                      → 400 ITEM_NOT_ACTIVATABLE
   quantity = 0                           → 409 ITEM_QUANTITY_EMPTY
4. có activation active (bất kỳ XP booster nào) → 409 BOOSTER_ALREADY_ACTIVE {expiresAt}
5. quantity − 1; INSERT booster_activations(startedAt = now, expiresAt = now + duration, key)
COMMIT → publish BOOSTER_USED
```

### 7.4. Equip / Unequip

`PUT /api/me/equipment/{type}` body `{ "itemId": 12 }` · `DELETE /api/me/equipment/{type}`; `type ∈ {THEME, AVATAR_FRAME}`.

```text
BEGIN
1. SELECT user FOR UPDATE
2. PUT: user_item không tồn tại → 404 ITEM_NOT_OWNED; item.type ≠ {type} → 400 ITEM_NOT_EQUIPPABLE
3. PUT: đã equipped → trả thành công, không đổi
        ngược lại: UPDATE user_items SET equipped = false WHERE user_id ∧ item_type = {type} ∧ equipped
                   UPDATE user_items SET equipped = true  WHERE id = target
   DELETE: UPDATE user_items SET equipped = false WHERE user_id ∧ item_type = {type} ∧ equipped (0 row vẫn thành công)
COMMIT
```

Idempotent tự nhiên, không cần `Idempotency-Key`. Thứ tự "unequip rồi equip" bắt buộc để không vi phạm `UNIQUE(user_id, equipped_slot)`.

### 7.5. Reward grant (Mission / Daily Chest / Weekly Chest)

Chạy **bên trong** transaction claim của reward (đã lock user):

```text
grant(userId, itemCode, quantity, fallbackCoin, eventKey):
  item = ShopItem by code (phải PUBLISHED hoặc ARCHIVED)
  cosmetic ∧ đã sở hữu                         → cộng fallbackCoin (CoinTransaction, source = nguồn reward, event_key = {eventKey}:FALLBACK)
  XP_BOOSTER: cấp tối đa tới maxQuantity; phần không cấp được → fallbackCoin × số đơn vị không cấp được
  ngược lại                                    → upsert user_items
```

- `fallbackCoin` là field bắt buộc của reward configuration có item (xem [daily_mission.md](./daily_mission.md)); Shop service không hard-code giá trị.
- Reward configuration chỉ được chọn item `PUBLISHED` (kể cả `purchasable = false`).

---

## 8. Admin CMS

### 8.1. Admin được phép

Tạo item (chọn type có sẵn), sửa field mở, upload icon/frame qua flow presign, chọn `themeKey` từ danh sách backend trả, cấu hình `price`, `purchasable`, sale window, `boostDurationMinutes`, `maxQuantity`, `sortOrder`; publish; archive; xóa DRAFT.

### 8.2. Field khóa

| Field | DRAFT | PUBLISHED | ARCHIVED |
| :--- | :--- | :--- | :--- |
| `code`, `type` | sửa | khóa | khóa |
| `themeKey`, `frameAssetKey`, `boostDurationMinutes` | sửa | khóa | khóa |
| `name`, `description`, `iconKey`, `sortOrder` | sửa | sửa | sửa |
| `price`, `purchasable`, `availableFrom`, `availableUntil`, `maxQuantity` | sửa | sửa | khóa (`purchasable = false`) |

Sửa field khóa → `409 FIELD_LOCKED`. Giảm `maxQuantity` không thu hồi item đã sở hữu; chỉ chặn mua/nhận thêm.

### 8.3. Validation (create/update/publish)

- `name` 1–100 ký tự; `description` ≤ 500; `code` `^[A-Z][A-Z0-9_]{2,63}$` unique.
- Tham số trong giới hạn §2.3; field của type khác phải null.
- Publish yêu cầu: `iconKey` đã upload; THEME có `themeKey` ∈ `ThemeKey` và chưa được item khác dùng; AVATAR_FRAME có `frameAssetKey` đạt spec asset; XP_BOOSTER có `boostDurationMinutes`, `maxQuantity`; nếu `purchasable` thì có `price`; `availableFrom < availableUntil` nếu cả hai có.
- Optimistic lock: mọi PATCH gửi `version`; lệch → `409 VERSION_CONFLICT`.

### 8.4. Dynamic form

`GET /api/admin/shop/item-types` trả, cho từng type: field bắt buộc, giới hạn, giá trị mặc định và (THEME) danh sách `themeKey` còn trống. Form hiện field theo type:

| Type | Field riêng | Preview |
| :--- | :--- | :--- |
| THEME | `themeKey` (dropdown) | ảnh preview tĩnh theo `themeKey` |
| AVATAR_FRAME | upload frame | frame trên avatar mẫu |
| XP_BOOSTER | `boostDurationMinutes`, `maxQuantity` | "×2 XP · 30 phút" |

### 8.5. Admin KHÔNG được

Nhập chuỗi type/effect/themeKey tùy ý; nhập màu/CSS/token; đổi multiplier; tạo tiền tệ khác; sửa field khóa; cộng/trừ Coin hoặc cấp item cho user (ngoài scope; khi làm phải qua `ADMIN_ADJUSTMENT` có ledger).

---

## 9. Database

```text
shop_items
  id                     BIGINT PK AUTO_INCREMENT
  code                   VARCHAR(64)  NOT NULL UNIQUE
  type                   VARCHAR(30)  NOT NULL            -- ShopItemType
  name                   VARCHAR(100) NOT NULL
  description            VARCHAR(500) NULL
  icon_key               VARCHAR(255) NULL                -- bắt buộc khi publish
  status                 VARCHAR(20)  NOT NULL DEFAULT 'DRAFT'
  purchasable            BOOLEAN      NOT NULL DEFAULT FALSE
  price                  BIGINT       NULL  CHECK (price IS NULL OR price > 0)
  available_from         DATETIME(6)  NULL
  available_until        DATETIME(6)  NULL
  sort_order             INT          NOT NULL DEFAULT 0
  theme_key              VARCHAR(50)  NULL UNIQUE         -- chỉ THEME
  frame_asset_key        VARCHAR(255) NULL                -- chỉ AVATAR_FRAME
  boost_duration_minutes INT          NULL                -- chỉ XP_BOOSTER
  max_quantity           INT          NULL                -- chỉ XP_BOOSTER
  published_at           DATETIME(6)  NULL
  version                BIGINT       NOT NULL DEFAULT 0
  created_at, updated_at DATETIME(6)  NOT NULL
  CHECK (NOT purchasable OR price IS NOT NULL)
  UNIQUE (id, type)                                       -- hỗ trợ composite FK từ user_items
  INDEX idx_shop_items_catalog (status, purchasable, type, sort_order)

user_items
  id             BIGINT PK AUTO_INCREMENT
  user_id        BIGINT NOT NULL FK → users(id)
  shop_item_id   BIGINT NOT NULL
  item_type      VARCHAR(30) NOT NULL                     -- copy từ shop_items.type; composite FK bảo đảm bằng DB
  quantity       INT NOT NULL CHECK (quantity >= 0)
  equipped       BOOLEAN NOT NULL DEFAULT FALSE
  equipped_slot  VARCHAR(30) GENERATED ALWAYS AS (IF(equipped, item_type, NULL)) STORED
  acquired_at    DATETIME(6) NOT NULL
  updated_at     DATETIME(6) NOT NULL
  FK (shop_item_id, item_type) → shop_items(id, type)     -- DB enforce: user_items.item_type == shop_items.type
  UNIQUE uq_user_items_owner (user_id, shop_item_id)
  UNIQUE uq_user_items_equipped (user_id, equipped_slot)  -- NULL không xung đột ⇒ tối đa 1 equipped / type

booster_activations
  id               BIGINT PK AUTO_INCREMENT
  user_id          BIGINT NOT NULL FK → users(id)
  shop_item_id     BIGINT NOT NULL FK → shop_items(id)
  started_at       DATETIME(6) NOT NULL
  expires_at       DATETIME(6) NOT NULL
  idempotency_key  VARCHAR(128) NOT NULL
  UNIQUE uq_booster_activation_key (user_id, idempotency_key)
  INDEX idx_booster_activation_active (user_id, expires_at)

coin_transactions
  id             BIGINT PK AUTO_INCREMENT
  user_id        BIGINT NOT NULL FK → users(id)
  amount         BIGINT NOT NULL CHECK (amount <> 0)
  balance_after  BIGINT NOT NULL CHECK (balance_after >= 0)
  source_type    VARCHAR(30) NOT NULL                     -- CoinSourceType
  reference_id   VARCHAR(64) NULL
  description    VARCHAR(255) NULL
  event_key      VARCHAR(191) NOT NULL UNIQUE
  created_at     DATETIME(6) NOT NULL
  INDEX idx_coin_tx_user_time (user_id, created_at)

users            (không thêm cột equip)
  coin BIGINT NOT NULL DEFAULT 0 CHECK (coin >= 0)

experience_logs  (bổ sung cho booster)
  base_amount, bonus_amount INT NOT NULL, booster_activation_id BIGINT NULL FK → booster_activations(id)
```

Ghi chú:

- **Vì sao equip nằm trên `user_items` thay vì FK trên `users`:** giữ trạng thái trong Shop domain, không mở rộng bảng `users`; invariant "1 equipped / type" được DB enforce bằng generated column + unique index nên đúng đắn tương đương FK. Composite FK `(shop_item_id, item_type) REFERENCES shop_items(id, type)` đảm bảo tuyệt đối ở tầng DB rằng `user_items.item_type == shop_items.type`, loại bỏ hoàn toàn rủi ro sai lệch dữ liệu giữa quyền sở hữu và catalog.
- JPA: `equipped_slot` map `insertable = false, updatable = false`; tạo qua migration/`columnDefinition` MySQL.
- `user_inventories` hiện có trong code được thay bằng `user_items`; chưa có dữ liệu cần migrate.

---

## 10. API contract

Envelope/error theo NFR API Standard ([specs.md](../spec/specs.md) §8). Mọi endpoint yêu cầu JWT; `/api/admin/**` yêu cầu `ROLE_ADMIN`.

### 10.1. Learner

| Method | Endpoint | Mô tả |
| :--- | :--- | :--- |
| GET | `/api/shop/items?type=` | Catalog đang bán + trạng thái theo user. Không phân trang (catalog nhỏ). |
| POST | `/api/shop/items/{itemId}/purchase` | Mua 1 đơn vị. Header `Idempotency-Key`; body `{expectedPrice}` |
| GET | `/api/me/items` | Inventory: item sở hữu, equipped, activeBooster |
| PUT | `/api/me/equipment/{type}` | Equip `{itemId}`; `type ∈ {THEME, AVATAR_FRAME}` |
| DELETE | `/api/me/equipment/{type}` | Unequip → mặc định |
| POST | `/api/me/items/{itemId}/activate` | Kích hoạt XP Booster. Header `Idempotency-Key` |
| GET | `/api/me/wallet?cursor=&size=` | Balance + CoinTransaction mới nhất trước (cursor) |

`GET /api/me` bổ sung `equipped: { themeKey, avatarFrameUrl }` và `activeBooster` để mobile áp theme/frame ngay khi mở app.

Response `GET /api/shop/items` (mỗi phần tử):

```json
{
  "itemId": 12, "code": "XP_BOOSTER_X2_30M", "type": "XP_BOOSTER",
  "name": "Nhân Đôi KN", "description": "...", "iconUrl": "...", "price": 250,
  "availableUntil": null,
  "config": { "multiplier": 2, "durationMinutes": 30, "maxQuantity": 5 },
  "owned": true, "quantity": 2, "equipped": false,
  "canPurchase": true, "blockReason": null
}
```

- `config`: THEME `{themeKey}`; AVATAR_FRAME `{frameUrl}`; XP_BOOSTER như trên.
- `blockReason ∈ {INSUFFICIENT_COINS, ALREADY_OWNED, MAX_QUANTITY_REACHED}`; server tính sẵn, client không tự suy.

Response purchase:

```json
{ "transactionId": 981, "itemId": 12, "quantity": 3, "owned": true, "coinBalance": 2200, "replayed": false }
```

Response `GET /api/me/items`:

```json
{
  "serverNow": "2026-09-24T10:00:00Z",
  "activeBooster": { "itemId": 12, "multiplier": 2, "startedAt": "...", "expiresAt": "2026-09-24T10:18:42Z" },
  "items": [
    { "itemId": 12, "type": "XP_BOOSTER", "name": "...", "iconUrl": "...", "quantity": 2, "equipped": false, "config": { "durationMinutes": 30 } },
    { "itemId": 5, "type": "AVATAR_FRAME", "name": "...", "iconUrl": "...", "quantity": 1, "equipped": true, "config": { "frameUrl": "..." } }
  ]
}
```

Response activate: `{ "activation": { "itemId", "startedAt", "expiresAt" }, "quantity": 1, "serverNow": "...", "replayed": false }`.

### 10.2. Admin

| Method | Endpoint | Mô tả |
| :--- | :--- | :--- |
| GET | `/api/admin/shop/item-types` | Metadata cho dynamic form (§8.4) |
| GET | `/api/admin/shop/items?status=&type=&q=` | Danh sách (phân trang) |
| GET | `/api/admin/shop/items/{id}` | Chi tiết |
| POST | `/api/admin/shop/items` | Tạo → `DRAFT` |
| PATCH | `/api/admin/shop/items/{id}` | Sửa, kèm `version` |
| POST | `/api/admin/shop/items/{id}/publish` | `DRAFT → PUBLISHED` |
| POST | `/api/admin/shop/items/{id}/archive` | `PUBLISHED → ARCHIVED` |
| DELETE | `/api/admin/shop/items/{id}` | Chỉ `DRAFT` |
| POST | `/api/admin/shop/assets/upload-url` | Presigned PUT `{fileName}` (PNG/WebP) → `objectName` dùng làm `iconKey`/`frameAssetKey` |

Lỗi validation Admin: `error` = mã của vi phạm đầu tiên, `data.errors = [{field, code, message}]` liệt kê mọi vi phạm. PATCH chỉ báo `FIELD_LOCKED` khi field khóa **đổi giá trị**; gửi lại giá trị cũ được chấp nhận. Field không thuộc form (VD `status`, `publishedAt`) → `VALIDATION_ERROR`. Tồn tại/dung lượng asset kiểm ở publish; kích thước pixel/alpha chưa kiểm server-side.

Upload icon/frame dùng flow presign của Storage (SS-16) với purpose `SHOP_ASSET`; asset Shop không chứa dữ liệu cá nhân nên được phục vụ qua URL công khai/CDN dưới prefix `public/shop/` (FR-11.05 chỉ giới hạn media cá nhân).

### 10.3. Error codes

| Code | HTTP | Khi nào |
| :--- | :--- | :--- |
| `IDEMPOTENCY_KEY_REQUIRED` | 400 | Thiếu/sai định dạng header |
| `IDEMPOTENCY_KEY_REUSED` | 409 | Key đã dùng cho item/thao tác khác |
| `SHOP_ITEM_NOT_FOUND` | 404 | Không tồn tại hoặc DRAFT (với learner) |
| `ITEM_NOT_AVAILABLE` | 409 | Không PUBLISHED, không purchasable, ngoài sale window |
| `PRICE_CHANGED` | 409 | `expectedPrice` ≠ giá hiện tại; `details.currentPrice` |
| `ITEM_ALREADY_OWNED` | 409 | Cosmetic đã sở hữu |
| `MAX_QUANTITY_REACHED` | 409 | XP_BOOSTER đạt `maxQuantity` |
| `INSUFFICIENT_COINS` | 409 | Số dư < giá |
| `ITEM_NOT_OWNED` | 404 | Equip/activate item không sở hữu |
| `ITEM_NOT_EQUIPPABLE` | 400 | Type không khớp slot |
| `ITEM_NOT_ACTIVATABLE` | 400 | Activate item không phải XP_BOOSTER |
| `ITEM_QUANTITY_EMPTY` | 409 | `quantity = 0` |
| `BOOSTER_ALREADY_ACTIVE` | 409 | Đang có XP Booster active; `details.expiresAt` |
| `VALIDATION_ERROR` | 400 | Admin: field sai, kèm danh sách lỗi theo field |
| `SHOP_ITEM_CODE_EXISTS` | 409 | Admin: `code` đã được item khác dùng |
| `INVALID_SHOP_ITEM_CONFIG` | 400 | Admin: field theo type thiếu/sai/ngoài giới hạn, field của type khác khác null, `themeKey` đã được item khác dùng, asset sai |
| `INVALID_THEME_KEY` | 400 | Admin: `themeKey` ngoài registry hoặc `DEFAULT` |
| `INVALID_PRICE` | 400 | Admin: `price` ngoài 1–100 000 hoặc thiếu khi `purchasable` |
| `INVALID_AVAILABILITY_WINDOW` | 400 | Admin: `availableFrom ≥ availableUntil` |
| `FIELD_LOCKED` | 409 | Admin sửa field bị khóa; `details.fields` |
| `VERSION_CONFLICT` | 409 | Admin lệch `version`; `details.currentVersion` |
| `INVALID_STATUS_TRANSITION` | 409 | Admin publish/archive sai trạng thái |
| `ITEM_CANNOT_BE_DELETED` | 409 | Admin xóa item không phải `DRAFT` |

---

## 11. Mobile UX

### 11.1. Shop (MH-ECONOMY-02)

| Type | CTA | Sau khi mua |
| :--- | :--- | :--- |
| THEME | `Xem trước` (sheet preview bắt buộc) → `Mua · {price}` · đã sở hữu: `Đã sở hữu` / `Đang dùng` | Modal thành công + `Dùng ngay` (gọi equip) |
| AVATAR_FRAME | Preview trên avatar của chính user → `Mua · {price}` · đã sở hữu: `Đã sở hữu` / `Đang dùng` | Modal + `Trang bị ngay` |
| XP_BOOSTER | `Mua · {price}` + "Đang có: n/max" · đạt max: `Đã đạt tối đa` (disabled) | Modal + `Dùng ngay` (ẩn nếu đang có booster active) |

- Thiếu Coin: nút hiển thị mờ nhưng **bấm được** → `OutOfCoinModal` gợi ý làm nhiệm vụ (AF-12.1).
- Confirm modal: "Mua {name} với {price} Coin?". Mobile sinh `Idempotency-Key` (UUID) khi bấm xác nhận, giữ nguyên khi retry do lỗi mạng/timeout, bỏ sau khi nhận kết quả dứt điểm (2xx hoặc 4xx nghiệp vụ). Lỗi mạng không rõ kết quả → gọi lại purchase cùng key hoặc refetch `/api/me/wallet` + `/api/me/items`.
- `PRICE_CHANGED` → cập nhật giá trên card, mở lại confirm.
- Coin giảm: count-down 400ms, không dùng màu đỏ ([design.md](../../snapVocab-frontend/docs/design.md) §6.6).
- Nhóm lọc: `Tất cả` · `Bổ trợ` (XP_BOOSTER) · `Khung Avatar` · `Chủ đề`.

### 11.2. Inventory (MH-ECONOMY-03)

- Đầu màn: Booster Chip khi có `activeBooster` (`×2 · mm:ss`, design.md §6.9). Đếm ngược tính từ `expiresAt` và độ lệch `serverNow`; về 0 → refetch.
- Cosmetic: `Dùng`/`Trang bị` ↔ `Đang dùng` + `Bỏ`/`Tháo`. Item archived vẫn hiển thị và dùng được.
- XP_BOOSTER: số lượng + `Dùng`; khi có booster active, mọi booster hiện "Chờ booster hiện tại kết thúc" (disabled).
- Ẩn XP_BOOSTER `quantity = 0` không active. THEME với `themeKey` lạ → "Cần cập nhật ứng dụng".
- Empty state: CTA `Đến Cửa hàng`.

### 11.3. Nơi khác

- MH-PROFILE-01: avatar hiển thị frame equipped của chính user.
- MH-PROFILE-03 Settings: "Giao diện" = Sáng/Tối/Hệ thống; mục "Chủ đề" chỉ là link sang Inventory.
- XP Badge hiện hậu tố `×2` khi `activeBooster` tồn tại (design.md §6.5).
- Mobile cache `equipped.themeKey` cục bộ để áp theme ngay lúc khởi động; server là nguồn đúng sau khi `GET /api/me` trả về.

---

## 12. Acceptance criteria

1. Mua thành công: `users.coin` giảm đúng `price`, có đúng 1 `coin_transactions` `SHOP_PURCHASE`, `user_items` được tạo hoặc `quantity + 1`.
2. Gửi lại cùng `Idempotency-Key` N lần (tuần tự hoặc song song) → coin chỉ bị trừ 1 lần, mọi response sau là `replayed = true`.
3. Hai purchase song song có tổng giá > balance → đúng một thành công, balance không bao giờ âm.
4. Mua lại cosmetic đã sở hữu → `ITEM_ALREADY_OWNED`, coin không đổi.
5. XP_BOOSTER đạt `maxQuantity` → `MAX_QUANTITY_REACHED`.
6. Admin đổi giá giữa lúc xem và lúc mua → `PRICE_CHANGED`, coin không đổi.
7. Equip THEME/AVATAR_FRAME mới → item cũ cùng type tự unequip trong cùng transaction; DB không thể có 2 row equipped cùng type.
8. Kích hoạt booster khi đang có booster active → `BOOSTER_ALREADY_ACTIVE`, quantity không đổi.
9. Review/Quiz ghi XP trong thời gian active → `bonusAmount = baseAmount`; Mission/Chest/Save word → `bonusAmount = 0`; XP ghi sau `expiresAt` → không bonus.
10. Bonus XP xuất hiện trong Weekly XP/Leaderboard.
11. Booster vẫn hết hạn đúng `expiresAt` khi app bị đóng/khởi động lại.
12. Weekly/Daily reward cấp cosmetic đã sở hữu → cộng `fallbackCoin` theo reward config.
13. Admin không thể publish item thiếu field theo type, `themeKey` ngoài registry, hoặc tham số ngoài giới hạn; không sửa được field khóa sau publish.
14. Item archived: không xuất hiện trong Shop; người đã sở hữu vẫn equip/activate được.

---

## 13. Code cleanup backlog (giai đoạn implement)

Tài liệu đã đồng bộ; code/mock sau đây **chưa** được sửa và phải chỉnh khi implement:

| Vị trí | Việc cần làm |
| :--- | :--- |
| `snap-vocab-backend/.../domain/UserInventory.java` | Thay bằng `UserItem` (`user_items`) theo §9 |
| `snap-vocab-backend/.../domain/ShopItem.java` | `type` → `ShopItemType`; bổ sung field §9 |
| `snap-vocab-backend/.../domain/dto/user/UserDTO.java` | `coin: Integer` → `Long` |
| `snapAdmin/src/domains/economy/*`, `features/shop/*`, `features/settings/components/EconomyGuardrailsTab.tsx`, `domains/settings/*` | Bỏ Gems, rarity, stock, flash sale, discount, `streak_saver`/`chest`/`deck`, Streak Freeze; form theo §8 |
| `snapAdmin/src/features/missions/*`, `tests/missions.test.ts` | Reward item tham chiếu `shopItem.code` + `fallbackCoin`; bỏ `ITEM_STREAK_FREEZE`/`CONSUMABLE` |
| `snapAdmin/src/domains/{analytics,audit,issue-reports,learners}/mock-data.ts` | Bỏ số liệu/nội dung Streak Freeze, Gems |
| `snapVocab-frontend/app/(tabs)/shop.tsx`, `lib/economyState.ts` | Bỏ item không có runtime (Streak Freeze, Heart Refill, Coin Magnet, Refresh Ticket, Hint Token, Timer Boost, Snapy cosmetic); category theo §11.1; gọi API thật |
