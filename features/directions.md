# Directions

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

Directions cung cấp chỉ đường từ điểm xuất phát đến điểm đích với nhiều phương tiện (lái xe, đi bộ, xe đạp, phương tiện công cộng, máy bay). Bao gồm: nhập origin/destination, lựa chọn mode, hiển thị nhiều tuyến đường thay thế, ETA, và tìm kiếm dọc tuyến đường (Search Along Route).

### Why

- **User need:** Người dùng cần biết "làm sao để đến đó?" — với route tối ưu, thời gian di chuyển, và phương tiện phù hợp.
- **Business value:** Directions là tính năng có engagement cao nhất trên Maps; là bề mặt chính cho Traffic ads và Transit ads; prerequisite cho Navigation mode.

---

# Part 02 - Specifications

## Backend

_Routing API (multi-modal). Transit API. Traffic data API._

## UI/UX & Frontend

### Mode Transitions

Directions có các trạng thái UI chính: **Input mode** (nhập điểm đi/đến) → **Results mode** (chọn tuyến đường) → **Navigation mode** (turn-by-turn).

### Transportation Modes

| Mode        | Icon | Mô tả                                           |
| ----------- | ---- | ----------------------------------------------- |
| Best        | ⚡   | Tự động chọn mode tối ưu dựa trên khoảng cách  |
| Driving     | 🚗   | Lái xe — kết hợp traffic realtime               |
| Transit     | 🚌   | Phương tiện công cộng (bus, metro, tàu)         |
| Walking     | 🚶   | Đi bộ                                           |
| Cycling     | 🚲   | Xe đạp                                          |
| Flight      | ✈️   | Chuyến bay (liên tỉnh / quốc tế)               |

### Triggers & Entry Points

| ID  | Trigger                                                          | Nền tảng    | Input                                  | AC  |
| --- | ---------------------------------------------------------------- | ----------- | -------------------------------------- | --- |
| T01 | Click nút [Chỉ đường] trong Place Detail Action Bar             | Web, Mobile | `place_id` → destination               | B01 |
| T02 | Context Menu → "Chỉ đường đến đây"                              | Web, Mobile | `(lat, lng)` → destination             | B01 |
| T03 | Context Menu → "Chỉ đường từ đây"                               | Web, Mobile | `(lat, lng)` → origin                  | B01 |
| T04 | Deep link URL `/directions/...?origin=...&dest=...&mode=`        | Web         | origin, destination, mode từ URL       | B02 |
| T05 | Nhập origin / destination trong Directions Panel                 | Web, Mobile | Text query (geocoded)                  | B03 |
| T06 | Swap origin ↔ destination                                       | Web, Mobile | —                                      | B04 |
| T07 | Thêm điểm dừng (multi-stop)                                     | Web, Mobile | `place_id` hoặc `(lat, lng)`           | B05 |
| T08 | Click vào route alternative trên bản đồ hoặc result list        | Web, Mobile | Route index                            | B06 |

### States Inventory

| State          | Mô tả                                      | Component                               |
| -------------- | ------------------------------------------ | --------------------------------------- |
| `input`        | Đang nhập origin/destination               | Directions input panel                  |
| `loading`      | Đang gọi Routing API                       | Panel + route skeleton trên bản đồ      |
| `results`      | Có tuyến đường                             | Route list + polyline trên bản đồ       |
| `selected`     | Đã chọn một tuyến                          | Route highlighted, details hiển thị     |
| `zero_results` | Không có tuyến đường                       | "Không tìm thấy tuyến đường"            |
| `error`        | Routing API lỗi                            | Toast                                   |

### Components, Responsive & Typography

#### Component Inventory

| Component                                 | Dùng trong            |
| ----------------------------------------- | --------------------- |
| Directions Panel (side panel)             | Web / tablet          |
| Directions Bottom Sheet                   | Mobile                |
| Origin / Destination input fields         | input state           |
| Swap button [⇅]                           | B04                   |
| Add stop button [+]                       | B05                   |
| Mode selector tabs                        | input, results state  |
| Route polyline (primary + alternatives)   | results, selected     |
| Route card (duration + distance + via)    | results state         |
| Traffic layer overlay                     | Driving mode          |
| Transit step-by-step list                 | Transit mode          |
| Search Along Route banner                 | selected state        |

#### Responsive Behavior

| Breakpoint                 | Layout                       |
| -------------------------- | ---------------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet full-height     |
| Tablet (768px–1024px)      | Side panel (360px)           |
| Desktop (> 1024px)         | Side panel (400px)           |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Mở Directions với destination (T01, T02, T03)

```gherkin
Given  người dùng click [Chỉ đường] trong Place Detail
Then   Directions Panel mở với destination đã điền
And    origin trống (hoặc "Vị trí của tôi" nếu GPS available)
And    mode mặc định = Best
And    URL cập nhật: /directions/

Given  Context Menu → "Chỉ đường đến đây"
Then   tọa độ right-click được set làm destination

Given  Context Menu → "Chỉ đường từ đây"
Then   tọa độ right-click được set làm origin
```

#### AC-B02 · Deep link URL (T04)

```gherkin
Given  URL /directions/?origin=lat,lng&dest=place_id&mode=driving
Then   Directions Panel mở, Routing API gọi với params tương ứng
And    kết quả route hiển thị ngay khi load
```

#### AC-B03 · Nhập origin / destination (T05)

```gherkin
Given  người dùng click vào input field origin/destination
Then   Autocomplete dropdown mở cho field đó (xem autocomplete-geocoding.md)
And    "Vị trí của tôi" là suggestion đầu tiên trong danh sách

Given  người dùng chọn hoặc nhập xong cả hai
Then   Routing API được gọi tự động
```

#### AC-B04 · Swap origin ↔ destination (T06)

```gherkin
Given  cả origin và destination đã có giá trị
When   người dùng click nút swap [⇅]
Then   origin và destination đổi chỗ cho nhau
And    Routing API được gọi lại với params mới
```

#### AC-B05 · Multi-stop routing (T07)

```gherkin
Given  người dùng click [+ Thêm điểm dừng]
Then   input field mới xuất hiện giữa origin và destination
And    Routing API được gọi với waypoints
And    tối đa 8 điểm dừng
```

#### AC-B06 · Route alternatives (T08)

```gherkin
Given  Routing API trả về nhiều tuyến
Then   tất cả tuyến hiển thị trên bản đồ (primary đậm, alternatives mờ hơn)
And    mỗi route card hiển thị: thời gian ETA, khoảng cách, via (đường chính)
And    traffic indicator (màu sắc) trên tuyến đường Driving mode

When   người dùng click route alternative
Then   route đó trở thành primary (đậm), các route khác mờ đi
And    route details (turn-by-turn hoặc transit steps) cập nhật
```

#### AC-B07 · Search Along Route

```gherkin
Given  một route đã được chọn
Then   banner "Tìm dọc tuyến đường" hiển thị
When   người dùng click banner và nhập query (vd: "cây xăng")
Then   tìm kiếm POI nằm gần tuyến đường (bán kính 500m từ route)
And    kết quả hiển thị trên bản đồ + list
```

#### AC-B08 · Error state

```gherkin
Given  Routing API lỗi hoặc timeout
Then   toast "Không thể tìm tuyến đường, vui lòng thử lại"
And    toast tự đóng sau 4 giây

Given  không tìm thấy tuyến đường giữa hai điểm
Then   panel hiển thị "Không tìm thấy tuyến đường cho phương tiện này"
And    gợi ý thử mode khác
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
