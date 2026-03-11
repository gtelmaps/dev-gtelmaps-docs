# Layer Toggler

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

Layer Toggler là panel điều khiển cho phép người dùng chuyển đổi giữa các loại bản đồ (Map type: Standard, Satellite, Terrain) và bật/tắt các overlay layer (Traffic, Transit, Bike, Air quality, Indoor...). Người dùng truy cập qua icon layers trên bản đồ.

### Why

- **User need:** Người dùng khác nhau cần loại bản đồ khác nhau — lái xe cần traffic overlay, đi bộ cần terrain, phân tích cần satellite. Việc chuyển đổi nhanh là yêu cầu cốt lõi.
- **Business value:** Satellite và Traffic layers là các layer premium; Layer Toggler là UX gateway để kích hoạt upsell và các tính năng có giá trị cao.

---

# Part 02 - Specifications

## Backend

_Tile server cung cấp tiles cho từng layer. Layer config (danh sách, icon, URL) từ Map Config API._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **UI Component:** Component nút `[Layers]` trên giao diện chính, hoặc thẻ chức năng chọn loại bản đồ nằm trong panel sau khi mở.
- **URL/Deep Link:** Hệ thống load loại bản đồ dựa trên tham số truyền vào từ URL.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                            | Nền tảng    | Input                      | AC  |
| --- | -------------------------------------------------- | ----------- | -------------------------- | --- |
| T01 | Click / tap icon [Layers] trên bản đồ              | Web, Mobile | —                          | B01 |
| T02 | Chọn Map Type (Standard / Satellite / Terrain)     | Web, Mobile | `map_type_id`              | B02 |
| T03 | Toggle Overlay layer (Traffic, Transit, Bike...)   | Web, Mobile | `layer_id` + `enabled`     | B03 |
| T04 | Đóng Layer Toggler panel                           | Web, Mobile | —                          | B04 |
| T05 | URL param `?layer=` khi load                       | Web         | `layer_id` từ URL          | B05 |

### States Inventory

| State    | Mô tả                                   | Component                           |
| -------- | --------------------------------------- | ----------------------------------- |
| `closed` | Panel đóng                              | Layer icon button bình thường       |
| `open`   | Panel mở                                | Layer picker panel                  |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong    |
| -------------------------------------- | ------------- |
| Layer icon button                      | T01           |
| Layer Toggler panel                    | T01–T04       |
| Map Type thumbnail grid (3 options)    | T02           |
| Overlay toggle switches                | T03           |
| Overlay layer icon + label             | T03           |

#### Responsive Behavior

| Breakpoint                 | Panel layout               | Position              |
| -------------------------- | -------------------------- | --------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet (half-screen) | —                     |
| Mobile landscape (< 768px) | Popover cạnh button        | Right side            |
| Tablet, Desktop            | Popover cạnh button        | Right side            |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Mở / đóng panel (T01)

```gherkin
Given  người dùng click icon [Layers]
Then   Layer Toggler panel mở với animation slide-in (200ms)
And    hiển thị Map Types section và Overlay Layers section

Given  panel đang mở, người dùng click icon [Layers] lại hoặc click ra ngoài
Then   panel đóng với animation slide-out
```

#### AC-B02 · Chọn Map Type (T02)

```gherkin
Given  Layer Toggler panel đang mở
When   người dùng click vào thumbnail "Satellite"
Then   map type chuyển sang Satellite (tiles reload ngay)
And    thumbnail "Satellite" được đánh dấu selected (border highlight)
And    URL cập nhật: thêm ?layer=satellite
And    Attribution cập nhật theo imagery provider của Satellite

Given  map type là Satellite
When   người dùng chọn "Standard"
Then   map type trở về Standard, imagery tiles bị unload
```

#### AC-B03 · Toggle Overlay layer (T03)

```gherkin
Given  người dùng toggle switch "Traffic" ON
Then   Traffic overlay render trên bản đồ (màu sắc tốc độ giao thông)
And    toggle switch chuyển sang active state

Given  người dùng toggle switch "Traffic" OFF
Then   Traffic overlay ẩn khỏi bản đồ
And    toggle switch về inactive

Given  người dùng bật nhiều overlay cùng lúc (Traffic + Transit)
Then   tất cả overlay được render đồng thời (không exclusive)
```

#### AC-B04 · Đóng panel (T04)

```gherkin
Given  Layer Toggler đang mở
When   người dùng nhấn Escape (web) hoặc click ngoài panel
Then   panel đóng, các layer đã chọn giữ nguyên
```

#### AC-B05 · Restore từ URL (T05)

```gherkin
Given  URL có ?layer=satellite hoặc ?layer=terrain
Then   map type tương ứng được chọn khi load
And    Layer Toggler hiển thị đúng trạng thái selected
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
