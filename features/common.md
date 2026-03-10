# Common

---

## Table of Contents

### Part 01 - Shared Specifications

### Part 02 - NFR Defaults

### Part 03 - Error Codes

### Part 04 - Glossary

---

# Part 01 - Shared Specifications

---

## Overview

File này định nghĩa các spec dùng chung, được các feature khác tham chiếu qua ký hiệu `(theo COMMON §N)`.

---

## §1 · Breakpoints

| Name    | Min width | Max width |
| ------- | --------- | --------- |
| mobile  | 0px       | 767px     |
| tablet  | 768px     | 1023px    |
| desktop | 1024px    | —         |

---

## §2 · Debounce & Throttle Defaults

| Context                       | Type      | Value  |
| ----------------------------- | --------- | ------ |
| Search / Autocomplete input   | Debounce  | 300ms  |
| Map pan → URL update          | Debounce  | 300ms  |
| Map pan → Weather refetch     | Throttle  | trigger on idle (map.on('idle')) + 50km threshold |
| State → LocalStorage write    | Debounce  | 500ms  |
| Hover → tooltip show          | Debounce  | 300ms  |
| Hover → tooltip hide          | Debounce  | 150ms  |

---

## §3 · Animation Durations

| Animation                        | Duration  | Easing           |
| -------------------------------- | --------- | ---------------- |
| Panel open / close               | 250ms     | ease-out         |
| Map fly-to                       | 600ms     | ease-in-out      |
| Map zoom click (+/−)             | 200ms     | cubic-bezier     |
| Map heading reset (compass)      | 300ms     | ease-out         |
| Tooltip fade in                  | 150ms     | ease             |
| Theme transition (color)         | 200ms     | ease             |
| Skeleton shimmer cycle           | 1.5s      | linear (loop)    |
| Skeleton → content swap          | 150ms     | fade             |

---

## §4 · Skeleton Loading Threshold

Tất cả loading states áp dụng ngưỡng 150ms trước khi hiển thị skeleton:

```
if (responseTime < 150ms) → render trực tiếp (no skeleton flash)
if (responseTime ≥ 150ms) → show skeleton until data ready
```

---

## §5 · API Timeout & Retry

| Loại API            | Timeout   | Retry policy              |
| ------------------- | --------- | ------------------------- |
| Search / Geocoding  | 5 giây    | 1 retry tự động sau 1 giây |
| Place Detail        | 5 giây    | 1 retry tự động           |
| Routing             | 10 giây   | Không retry tự động       |
| Weather             | 5 giây    | Silent fail, dùng cache   |
| Autocomplete        | 3 giây    | Silent fail (no toast)    |
| Tile requests       | 15 giây   | Browser retry tự nhiên    |

---

## §6 · Toast Notifications

| Property     | Value       |
| ------------ | ----------- |
| Auto-close   | 4 giây      |
| Position     | Bottom-center (mobile) / Bottom-right (desktop) |
| Max stacked  | 3 toasts    |
| Dismiss      | Click hoặc swipe-away |

---

## §7 · Empty State Standard

Mọi empty state trong app phải có:
1. Illustration SVG (per feature context)
2. Title: ngắn gọn, mô tả vấn đề
3. Subtitle: giải thích nguyên nhân hoặc gợi ý
4. CTA button (nếu có hành động khôi phục)

---

## §8 · Z-Index Scale

| Layer                     | z-index range  |
| ------------------------- | -------------- |
| Map canvas                | 0              |
| Map overlay (markers...)  | 1–99           |
| Widgets (zoom, compass...) | 100–199       |
| Side panels               | 200–299        |
| Bottom sheets             | 300–399        |
| Modals / dialogs          | 400–499        |
| Tooltips                  | 500–599        |
| Toasts                    | 600–699        |
| Loading overlays          | 700–799        |

---

## §9 · Accessibility (A11y) Baseline

- Tất cả interactive elements phải có `aria-label` hoặc visible text
- Focus trap trong modals và bottom sheets
- Color contrast ratio ≥ 4.5:1 (WCAG AA)
- Keyboard navigable: Tab, Shift+Tab, Enter, Escape, Arrow keys
- Toast announcements qua `aria-live="polite"`

---

# Part 02 - NFR Defaults

| NFR                          | Target                          |
| ---------------------------- | ------------------------------- |
| First Contentful Paint       | ≤ 1.5s (desktop), ≤ 2.5s (3G) |
| Time To Interactive          | ≤ 3s (desktop)                  |
| API P95 response (geocoding) | ≤ 300ms (cache MISS)            |
| API P95 response (place)     | ≤ 500ms                         |
| Tile load time               | ≤ 200ms / tile (P95)            |
| Map frame rate               | ≥ 60fps pan/zoom                |
| Bundle size (initial JS)     | ≤ 200KB gzipped                 |

---

# Part 03 - Error Codes

| Code  | Mô tả                         | User message                                      |
| ----- | ----------------------------- | ------------------------------------------------- |
| E001  | Network timeout               | "Kết nối mất quá lâu. Vui lòng thử lại."         |
| E002  | API 4xx (client error)        | "Yêu cầu không hợp lệ."                          |
| E003  | API 5xx (server error)        | "Lỗi hệ thống. Đang được xử lý."                 |
| E004  | Zero results                  | "Không tìm thấy kết quả." (per feature)           |
| E005  | Geolocation denied            | "Bật quyền vị trí để dùng tính năng này."        |
| E006  | Geolocation unavailable       | "Không thể xác định vị trí của bạn."             |
| E007  | WebGL not supported           | "Trình duyệt không hỗ trợ bản đồ 3D."           |
| E008  | Offline                       | "Không có kết nối internet."                     |
| E009  | Chunk load failed             | "Cập nhật mới có sẵn. Vui lòng tải lại trang."  |
| E010  | Microphone denied             | "Bật quyền microphone để dùng tìm kiếm giọng nói."|

---

# Part 04 - Glossary

| Term           | Định nghĩa                                                             |
| -------------- | ---------------------------------------------------------------------- |
| POI            | Point of Interest — địa điểm có thể tìm kiếm trên bản đồ             |
| place_id       | Unique identifier của một POI trong hệ thống GTEL Maps                |
| Geocoding      | Chuyển đổi text → tọa độ (Forward) hoặc tọa độ → text (Reverse)      |
| map center     | Tọa độ tâm viewport của bản đồ hiện tại                               |
| zoom level     | Cấp độ phóng to / thu nhỏ (0 = toàn cầu, 22 = chi tiết tối đa)      |
| tilt           | Góc nghiêng camera (0° = 2D, 60° = max 3D)                           |
| heading        | Hướng la bàn bản đồ (0° = Bắc, 90° = Đông, 180° = Nam, 270° = Tây)  |
| Bottom Sheet   | Panel trượt từ đáy màn hình lên (mobile UI pattern)                   |
| Skeleton       | Placeholder animation trong khi nội dung đang load                    |
| Debounce       | Trì hoãn function call cho đến khi ngừng trigger một khoảng thời gian  |
| TTL            | Time To Live — thời gian hợp lệ của cache                            |
| FOUC           | Flash Of Unstyled Content — nháy giao diện sai trước khi CSS load    |
| Location bias  | Ưu tiên kết quả search gần vị trí hiện tại của người dùng            |
| White-label    | Tùy chỉnh giao diện theo thương hiệu của đối tác (logo, màu sắc)     |
