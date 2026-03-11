# Geolocation

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

Geolocation cho phép Maps Viewer xác định và theo dõi vị trí GPS của người dùng. Bao gồm: xin quyền vị trí, hiển thị My Location marker, center bản đồ về vị trí hiện tại, và chế độ Follow Location (bản đồ tự động theo người dùng di chuyển).

### Why

- **User need:** "Tôi đang ở đâu?" là câu hỏi phổ biến nhất trên Maps. Người dùng cần biết vị trí chính xác của mình trên bản đồ để điều hướng và khám phá xung quanh.
- **Business value:** Geolocation là prerequisite cho Search Nearby theo GPS, Navigation mode, và Fleet tracking realtime.

---

# Part 02 - Specifications

## Backend

_Geolocation API (browser built-in). IP geolocation API (fallback)._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **App Lifecycle (Vòng đời ứng dụng):** Yêu cầu cấp quyền vị trí lúc khởi động.
- **UI Component:** Nút `[My Location]` (Định vị) nằm trên giao diện điều khiển.
- **Sensors (Cảm biến máy):** Tín hiệu tự động thay đổi từ dịch vụ định vị GPS (hoặc Network IP fallback).

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                                   | Nền tảng    | Input                               | AC  |
| --- | --------------------------------------------------------- | ----------- | ----------------------------------- | --- |
| T01 | Khởi động app lần đầu — xác thực quyền vị trí           | Web, Mobile | Browser Permissions API             | B01 |
| T02 | Click / tap nút [My Location]                             | Web, Mobile | —                                   | B02 |
| T03 | Vị trí GPS thay đổi (watchPosition callback)              | Mobile, Web | `GeolocationPosition` object        | B03 |
| T04 | Bật chế độ Follow Location (tap lại My Location đang active) | Mobile  | —                                   | B04 |
| T05 | Người dùng pan bản đồ khi Follow mode đang on             | Mobile      | pan gesture                         | B05 |
| T06 | Accuracy thấp / GPS signal yếu                            | Web, Mobile | `accuracy` từ GeolocationPosition   | B06 |

### States Inventory

| State             | Mô tả                                          | Component                            |
| ----------------- | ---------------------------------------------- | ------------------------------------ |
| `unknown`         | Chưa xin quyền hoặc chưa có vị trí            | My Location button bình thường       |
| `requesting`      | Đang chờ permission prompt                     | Button spinner                       |
| `locating`        | Đã có quyền, đang fetch vị trí                | Pulsing marker placeholder           |
| `located`         | Có vị trí, bản đồ center                      | My Location marker + blue dot        |
| `following`       | Follow mode — bản đồ tự theo GPS              | Marker + button active + heading cone |
| `permission_denied` | Người dùng từ chối hoặc browser blocked     | Button disabled + tooltip            |
| `error`           | GPS error / timeout                            | Toast                                |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong         |
| -------------------------------------- | ------------------ |
| My Location button                     | T02–T05            |
| My Location marker (blue dot)          | located, following |
| Accuracy circle (semi-transparent)     | B06                |
| Heading cone (direction indicator)     | following state    |
| Permission tooltip                     | permission_denied  |
| Toast notification                     | error state        |

#### Responsive Behavior

| Breakpoint                 | My Location button position |
| -------------------------- | --------------------------- |
| Mobile portrait (< 768px)  | Bottom-right                |
| Desktop                    | Bottom-right (trên zoom)    |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Xác thực quyền khi startup (T01)

```gherkin
Given  lần đầu mở app
Then   browser permission prompt cho Geolocation xuất hiện (sau khi map ready, không block render)
And    nếu đã granted trước đó → skip prompt, thực hiện B02 tự động

Given  permission denied / blocked
Then   My Location button hiển thị disabled
And    tooltip khi hover: "Bật quyền vị trí trong cài đặt trình duyệt"
```

#### AC-B02 · Click My Location button (T02)

```gherkin
Given  permission đã granted
When   người dùng click My Location button
Then   button hiển thị loading spinner
And    gọi navigator.geolocation.getCurrentPosition()
And    sau khi có vị trí: bản đồ fly-to vị trí GPS (zoom 15, animation 600ms)
And    My Location marker xuất hiện (blue dot + accuracy circle)
And    button chuyển sang active state

Given  permission chưa được xin
Then   trigger permission prompt trước khi locate
```

#### AC-B03 · Cập nhật vị trí realtime (T03)

```gherkin
Given  đã có vị trí GPS
Then   watchPosition callback active
And    My Location marker cập nhật vị trí khi GPS thay đổi > 5m
And    accuracy circle resize theo accuracy value mới
```

#### AC-B04 · Follow Location mode (T04)

```gherkin
Given  My Location đang active (located state)
When   người dùng tap lại nút My Location
Then   bật Follow mode
And    bản đồ tự rotate theo compass heading của thiết bị (mobile)
And    heading cone xuất hiện trên marker
And    button chuyển sang "follow active" variant

Given  Follow mode đang on
When   GPS position thay đổi
Then   bản đồ tự pan để giữ My Location marker ở trung tâm
```

#### AC-B05 · Thoát Follow mode khi pan (T05)

```gherkin
Given  Follow mode đang on
When   người dùng pan bản đồ thủ công
Then   Follow mode tắt ngay lập tức
And    My Location marker vẫn hiển thị nhưng bản đồ không tự follow nữa
And    button trở về "located" state (không phải "follow")
```

#### AC-B06 · Accuracy thấp (T06)

```gherkin
Given  GPS accuracy > 100m (tín hiệu yếu)
Then   accuracy circle hiển thị bán kính lớn (trực quan cho người dùng thấy độ không chính xác)
And    không hiển thị toast cảnh báo (accuracy circle đủ để thông tin)

Given  GPS accuracy > 1000m (không có GPS, dùng IP/WiFi fallback)
Then   toast nhỏ: "Vị trí ước lượng — độ chính xác thấp"
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
