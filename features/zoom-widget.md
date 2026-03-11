# Zoom Widget

---

## Table of Contents

### Part 01 - Background

### Part 02 - Specifications

### Part 03 - Operations

### Part 04 - Quality & Release

### Part 05 - References

# Part 01 - Background

---

## Overview

### What

Zoom Widget là cặp nút [+] / [−] cố định trên giao diện bản đồ, cho phép người dùng phóng to / thu nhỏ bản đồ theo từng bước zoom level. Widget phản ánh trạng thái zoom hiện tại và vô hiệu hóa nút khi đã đạt giới hạn min/max.

### Why

- **User need:** Người dùng trên desktop không có cử chỉ pinch-to-zoom; nút [+]/[−] là điều khiển zoom tường minh, đặc biệt quan trọng với người dùng lớn tuổi hoặc không quen dùng scroll wheel.
- **Business value:** Là thành phần UI cơ bản của mọi maps product — thiếu widget này làm giảm usability đáng kể trên web.

---

# Part 02 - Specifications

## Backend

_Không có backend riêng — hoàn toàn là client-side map interaction._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Zoom Widget Component:** Điều khiển zoom thủ công bằng việc ấn trực tiếp nút `[+]` và `[-]`.
- **Tương tác Gesture/Thiết bị:** Các hành động zoom bằng cuộn chuột, chạm thao tác (pinch), nhấp đúp con trỏ (double-click) và phím tắt bàn phím.
- **Hệ thống/API:** Tác động từ tính năng hoặc API gây cập nhật lại zoom level (fly-to) từ nguồn bên ngoài.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                       | Nền tảng    | Input                             | AC  |
| --- | --------------------------------------------- | ----------- | --------------------------------- | --- |
| T01 | Click nút [+]                                 | Web         | —                                 | B01 |
| T02 | Click nút [−]                                 | Web         | —                                 | B01 |
| T03 | Scroll wheel lên / xuống trên bản đồ          | Web         | `deltaY` từ wheel event           | B02 |
| T04 | Pinch gesture (2 ngón)                        | Mobile      | `scale` từ touch event            | B03 |
| T05 | Double-tap trên bản đồ                        | Mobile      | Touch position                    | B04 |
| T06 | Double-click trên bản đồ                      | Web         | Click position                    | B04 |
| T07 | Phím tắt `+` / `=` (zoom in), `-` (zoom out) | Web         | Keyboard event                    | B05 |
| T08 | Map zoom thay đổi từ nguồn bên ngoài          | Web, Mobile | Zoom level từ API / URL / fly-to  | B06 |

### States Inventory

| State       | Mô tả                                | Component                        |
| ----------- | ------------------------------------ | -------------------------------- |
| `normal`    | Zoom trong khoảng min–max            | Cả hai nút enabled               |
| `max_zoom`  | Đã đạt zoom level tối đa             | Nút [+] disabled (dimmed)        |
| `min_zoom`  | Đã đạt zoom level tối thiểu          | Nút [−] disabled (dimmed)        |
| `animating` | Đang trong quá trình zoom animation  | Nút vẫn clickable (queue input)  |

### Components, Responsive & Typography

#### Component Inventory

| Component                   | Dùng trong    |
| --------------------------- | ------------- |
| Zoom In button [+]          | T01, T07      |
| Zoom Out button [−]         | T02, T07      |
| Zoom level indicator (opt.) | T08           |

#### Responsive Behavior

| Breakpoint                 | Widget position         | Button size |
| -------------------------- | ----------------------- | ----------- |
| Mobile portrait (< 768px)  | Ẩn (dùng pinch gesture) | —           |
| Mobile landscape (< 768px) | Bottom-right            | 36×36px     |
| Tablet (768px–1024px)      | Bottom-right            | 36×36px     |
| Desktop (> 1024px)         | Bottom-right            | 40×40px     |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Click nút [+] / [−] (T01, T02)

```gherkin
Given  bản đồ đang hiển thị ở zoom level trung gian
When   người dùng click [+]
Then   zoom level tăng thêm 1 với animation smooth (duration 200ms)
And    bản đồ giữ nguyên tâm hiển thị

When   người dùng click [−]
Then   zoom level giảm 1 với animation smooth

Given  đang ở max zoom level
Then   nút [+] hiển thị disabled (opacity 0.4, cursor: not-allowed)
And    click vào nút [+] không có hiệu ứng gì

Given  đang ở min zoom level
Then   nút [−] disabled tương tự
```

#### AC-B02 · Scroll wheel (T03)

```gherkin
Given  con trỏ nằm trên vùng bản đồ
When   scroll wheel lên (deltaY < 0)
Then   zoom in xung quanh vị trí con trỏ (không phải map center)

When   scroll wheel xuống (deltaY > 0)
Then   zoom out xung quanh vị trí con trỏ
```

#### AC-B03 · Pinch gesture (T04)

```gherkin
Given  người dùng đặt 2 ngón tay trên bản đồ
When   spread (kéo ra xa nhau)
Then   zoom in liên tục theo tỷ lệ pinch scale

When   pinch (kéo vào gần nhau)
Then   zoom out liên tục theo tỷ lệ pinch scale
```

#### AC-B04 · Double-click / Double-tap (T05, T06)

```gherkin
Given  người dùng double-click / double-tap tại vị trí bất kỳ
Then   zoom in +1 và bản đồ center về vị trí click / tap
```

#### AC-B05 · Phím tắt keyboard (T07)

```gherkin
Given  bản đồ đang focused (không có input field nào đang active)
When   nhấn phím [+] hoặc [=]
Then   zoom in +1

When   nhấn phím [-]
Then   zoom out 1
```

#### AC-B06 · Đồng bộ trạng thái từ nguồn ngoài (T08)

```gherkin
Given  zoom level thay đổi qua API (flyTo, fitBounds...) hoặc deep link URL
Then   trạng thái disabled của nút [+]/[−] cập nhật đúng theo zoom level mới
```

### Flow — UI

## NFR & Performance

## Risks & Assumptions

---

# Part 03 - Operations

---

# Part 04 - Quality & Release

---

# Part 05 - References

## Document References

## Changelog
