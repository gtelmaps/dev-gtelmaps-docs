# Map Interaction

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

Map Interaction định nghĩa toàn bộ các tương tác trực tiếp của người dùng với vùng bản đồ: pan, zoom, tilt, heading (xoay), click, double-click, long-press, context menu, và cursor/gesture behavior. Đây là tầng interaction cơ bản của Maps Viewer, được các feature khác (Reverse Geocoding, Place Detail, Directions...) kế thừa.

### Why

- **User need:** Bản đồ là canvas tương tác chính — mọi thao tác phải responsive, smooth và nhất quán giữa web và mobile.
- **Business value:** Trải nghiệm interaction mượt mà là yếu tố retention hàng đầu của Maps product.

## Unique Selling Propositions (USP)

| #   | USP                            | Mô tả                                                                            | So sánh                        |
| --- | ------------------------------ | -------------------------------------------------------------------------------- | ------------------------------ |
| 1   | Gesture nhất quán web + mobile | Cùng interaction model (pan, zoom, tilt, rotate) trên mọi nền tảng               | Google Maps tương đương        |
| 2   | Tilt + heading 3D smooth       | Hỗ trợ tilt/heading 3D mượt mà, encode vào URL để share                          | Google Maps cũng hỗ trợ 3D     |
| 3   | Cursor context-aware           | Cursor thay đổi theo vùng tương tác (grab, pointer, crosshair) — UX rõ ràng      | Google Maps cursor ít thay đổi |
| 4   | Long press mobile chuẩn 500ms  | Thống nhất timing long press cho Reverse Geocoding, Directions — haptic feedback | Google Maps tương đương        |

---

# Part 02 - Specifications

## Backend

_Không có backend — hoàn toàn là client-side map engine (MapLibre / Mapbox GL)._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**

- **Bản đồ (Map):** Tương tác với vùng không gian của bản đồ thông qua các thiết bị nhập liệu (chuột, bàn phím) hoặc màn hình cảm ứng.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                           | Nền tảng    | Input                             | AC  |
| --- | ------------------------------------------------- | ----------- | --------------------------------- | --- |
| T01 | Click / tap trên vùng bản đồ trống                | Web, Mobile | `(lat, lng)` từ click event       | B01 |
| T02 | Pan (drag chuột / swipe 1 ngón)                   | Web, Mobile | `dx, dy` delta                    | B02 |
| T03 | Zoom (scroll wheel / pinch / double-click/tap)    | Web, Mobile | zoom delta hoặc scale factor      | B03 |
| T04 | Tilt (right-drag dọc web / 2-finger swipe mobile) | Web, Mobile | tilt delta                        | B04 |
| T05 | Xoay heading (right-drag ngang / 2-finger rotate) | Web, Mobile | heading delta                     | B05 |
| T06 | Right-click trên bản đồ                           | Web         | `(lat, lng)` từ right-click event | B06 |
| T07 | Long press ≥ 500ms                                | Mobile      | `(lat, lng)` từ touch             | B07 |
| T08 | Hover di chuyển con trỏ trên bản đồ               | Web         | `(lat, lng)` realtime             | B08 |

### States Inventory

| State      | Mô tả                  | Cursor / Gesture feedback       |
| ---------- | ---------------------- | ------------------------------- |
| `idle`     | Không tương tác        | Cursor: `grab` (hoặc `default`) |
| `panning`  | Đang pan               | Cursor: `grabbing`              |
| `zooming`  | Đang zoom (animation)  | Cursor: `grab`                  |
| `tilting`  | Đang tilt              | Cursor: `row-resize`            |
| `rotating` | Đang xoay              | Cursor: `crosshair`             |
| `clicking` | Click / tap vừa xảy ra | Ripple effect tại điểm click    |

### Components, Responsive & Typography

#### Component Inventory

| Component              | Dùng trong                |
| ---------------------- | ------------------------- |
| Map canvas (WebGL)     | T01–T08                   |
| Click ripple animation | T01, T07                  |
| Context Menu           | T06 (xem context-menu.md) |
| Cursor CSS class       | T01–T05, T08              |

#### Responsive Behavior

| Breakpoint                | Pan            | Zoom           | Tilt / Rotate    |
| ------------------------- | -------------- | -------------- | ---------------- |
| Mobile portrait (< 768px) | 1-finger swipe | Pinch 2-finger | 2-finger gesture |
| Mobile landscape          | 1-finger swipe | Pinch 2-finger | 2-finger gesture |
| Tablet                    | 1-finger swipe | Pinch / scroll | 2-finger gesture |
| Desktop                   | Drag chuột     | Scroll wheel   | Right-drag       |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Click / tap trên bản đồ trống (T01)

```gherkin
Given  người dùng click / tap vào vùng bản đồ không có marker / POI
Then   Reverse Geocoding được trigger (xem reverse-geocoding.md §B01)
And    ripple animation tại điểm click (radius 20px, 300ms, fade out)

Given  người dùng click vào POI marker
Then   Place Detail được trigger (xem place-detail.md §B02)
And    click event không propagate xuống bản đồ (không trigger reverse geocoding)
```

#### AC-B02 · Pan (T02)

```gherkin
Given  người dùng drag chuột / swipe 1 ngón
Then   bản đồ follow con trỏ / ngón tay smoothly (1:1 ratio)
And    cursor thay đổi thành `grabbing` khi đang pan
And    inertia scrolling sau khi nhả (decelerate naturally 300–500ms)

Given  người dùng pan chạm đến giới hạn maxBounds (nếu có)
Then   bản đồ dừng lại tại boundary, không có "bounce" effect
```

#### AC-B03 · Zoom (T03)

```gherkin
Given  scroll wheel lên hoặc pinch out
Then   zoom in xung quanh vị trí con trỏ / điểm giữa 2 ngón
And    animation smooth (easing: cubic-bezier)

Given  zoom đã ở min hoặc max
Then   không thể zoom tiếp, không có visual feedback thừa
```

#### AC-B04 · Tilt (T04)

```gherkin
Given  right-drag dọc (web) hoặc 2-finger swipe dọc (mobile)
Then   tilt thay đổi liên tục (0° – 60°)
And    2D/3D widget cập nhật theo (xem 2d-3d-widget.md)
And    tilt = 0 → widget 2D/3D ẩn heading indicator
```

#### AC-B05 · Xoay heading (T05)

```gherkin
Given  right-drag ngang (web) hoặc 2-finger rotate (mobile)
Then   heading thay đổi liên tục (0°–360°)
And    Compass Widget xuất hiện và cập nhật realtime (xem compass-widget.md)
```

#### AC-B06 · Right-click (T06)

```gherkin
Given  người dùng right-click trên bản đồ
Then   Context Menu xuất hiện tại vị trí con trỏ (xem context-menu.md)
And    di chuyển bản đồ / click ra ngoài → đóng Context Menu
```

#### AC-B07 · Long press mobile (T07)

```gherkin
Given  người dùng giữ ngón tay ≥ 500ms trên bản đồ
Then   haptic feedback (nếu device hỗ trợ)
And    Reverse Geocoding được trigger tại vị trí đó (xem reverse-geocoding.md §B09)

Given  người dùng nhấc ngón tay trước 500ms
Then   không kích hoạt long press
```

#### AC-B08 · Hover (T08)

```gherkin
Given  con trỏ di chuyển qua POI marker
Then   cursor chuyển thành `pointer`
And    Tooltip preview trigger sau 300ms (xem place-detail.md §B01)

Given  con trỏ di chuyển ra ngoài marker
Then   cursor về `grab`
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
