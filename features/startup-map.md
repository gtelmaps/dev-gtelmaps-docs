# Startup Map

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

Startup Map định nghĩa toàn bộ hành vi khởi tạo của Maps Viewer: cài đặt trạng thái ban đầu cho bản đồ (vị trí, zoom, layer, UI), xin quyền từ người dùng (location, cookie), và xử lý các entry point khác nhau (URL direct, deep link, embed). Startup xác định trải nghiệm first-impression của người dùng.

### Why

- **User need:** Người dùng kỳ vọng bản đồ load nhanh, hiển thị đúng khu vực họ cần ngay từ đầu (từ URL, lịch sử, hoặc vị trí GPS) mà không cần thao tác thêm.
- **Business value:** Startup performance và UX ảnh hưởng trực tiếp đến bounce rate và first-session retention.

---

# Part 02 - Specifications

## Backend

_Map tile serving; Session API (nếu có); Geolocation IP fallback API._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                               | Nền tảng    | Input                                              | AC  |
| --- | --------------------------------------------------------------------- | ----------- | -------------------------------------------------- | --- |
| T01 | Mở URL gốc `/{locale}/maps` không có params                          | Web         | —                                                  | B01 |
| T02 | Mở deep link với `/@lat,lng,zoom[,tilt[,heading]]`                  | Web         | URL params                                         | B02 |
| T03 | Mở deep link với entry type `?ttu=`, `?s=`, `?yt=`, `?ml=`, `?data=` | Web        | URL query params                                   | B03 |
| T04 | Bản đồ load — xin quyền vị trí (Geolocation permission)              | Web, Mobile | Browser permission API                             | B04 |
| T05 | Bản đồ load — xin đồng ý cookie / tracking                           | Web         | Cookie consent config                              | B05 |
| T06 | Quay lại trang sau khi navigate (browser back)                        | Web         | Session state / URL                                | B06 |
| T07 | Load lần đầu không có URL params — fallback về vị trí đã lưu        | Web         | LocalStorage state                                 | B07 |

### States Inventory

| State          | Mô tả                                         | Component                                  |
| -------------- | --------------------------------------------- | ------------------------------------------ |
| `loading`      | Tiles đang download                           | Skeleton / loading overlay                 |
| `ready`        | Bản đồ render xong, UI interactive            | Full map UI                                |
| `permission`   | Đang chờ người dùng respond location request  | Location permission prompt                 |
| `cookie`       | Đang chờ người dùng respond cookie consent    | Cookie consent banner                      |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong    |
| -------------------------------------- | ------------- |
| Map loading skeleton / progress bar    | loading state |
| Location permission prompt             | T04           |
| Cookie consent banner                  | T05           |
| Search Bar (auto-focus)                | B01           |

#### Responsive Behavior

| Breakpoint                 | Initial view behavior                        |
| -------------------------- | -------------------------------------------- |
| Mobile portrait (< 768px)  | Search bar hiển thị, không auto-focus        |
| Desktop (> 1024px)         | Search bar auto-focus ngay sau khi map ready |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Load URL gốc (T01)

```gherkin
Given  người dùng mở /{locale}/maps không có params
Then   bản đồ load tại vị trí mặc định (xem AC-B07 nếu có saved state)
And    Search Bar được auto-focus (desktop only)
And    zoom level mặc định = 13
And    URL được cập nhật thành /@lat,lng,zoom ngay sau khi map ready
```

#### AC-B02 · Deep link URL với tọa độ (T02)

```gherkin
Given  URL /{locale}/maps/@10.7769,106.7009,14z
Then   bản đồ load tại tọa độ 10.7769,106.7009 với zoom 14
And    nếu có tilt → bản đồ mở ở chế độ 3D tương ứng
And    nếu có heading → compass hiển thị, bản đồ xoay đúng hướng

Given  URL có param không hợp lệ (lat/lng out of range, zoom không hợp lệ)
Then   bản đồ load tại vị trí mặc định và bỏ qua param lỗi
```

#### AC-B03 · Entry type params (T03)

```gherkin
Given  URL có ?ttu= (transit URL) hoặc ?s= (search) hoặc ?yt= (youtube place) hoặc ?ml= (my location) hoặc ?data= (encoded data)
Then   hệ thống parse entry type và restore đúng trạng thái tương ứng
And    ví dụ: ?s=coffee → mở Search Results cho "coffee"
And    ví dụ: ?ml=lat,lng → center bản đồ về vị trí đó
```

#### AC-B04 · Xin quyền vị trí (T04)

```gherkin
Given  lần đầu mở ứng dụng
Then   browser permission prompt cho Geolocation xuất hiện sau khi map ready (không block render)

Given  người dùng cho phép (granted)
Then   bản đồ fly-to vị trí GPS hiện tại
And    My Location marker xuất hiện

Given  người dùng từ chối (denied)
Then   bản đồ giữ nguyên vị trí mặc định / saved
And    My Location button disabled với tooltip hướng dẫn bật lại

Given  người dùng đã từ chối trước đó (permission: denied)
Then   không hỏi lại, My Location button disabled
```

#### AC-B05 · Cookie consent (T05)

```gherkin
Given  lần đầu mở ứng dụng (chưa có cookie consent)
Then   Cookie Consent banner xuất hiện phía dưới bản đồ (không block map)
And    bản đồ vẫn có thể tương tác trong khi banner hiển thị

Given  người dùng chọn "Chấp nhận tất cả"
Then   banner đóng, consent được lưu vào cookie

Given  người dùng chọn "Từ chối"
Then   chỉ essential cookies được lưu, analytics disabled
And    banner đóng
```

#### AC-B06 · Browser back navigation (T06)

```gherkin
Given  người dùng nhấn nút Back của trình duyệt
Then   Maps Viewer restore trạng thái trước (URL-based history)
And    bản đồ animate về vị trí / zoom trước đó
And    panel (nếu có mở) restore lại
```

#### AC-B07 · Fallback về saved state (T07)

```gherkin
Given  không có URL params và có saved state trong LocalStorage
Then   bản đồ load tại vị trí / zoom / layer đã lưu lần cuối

Given  saved state bị corrupt hoặc expired
Then   bản đồ load về vị trí mặc định (Hà Nội hoặc TP.HCM theo locale)
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
