# Floor Selector

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

Floor Selector là widget cho phép người dùng điều hướng giữa các tầng trong tòa nhà khi bản đồ hiển thị indoor map (bản đồ tầng lầu). Widget xuất hiện tự động khi zoom đến tòa nhà có indoor data và ẩn khi zoom out khỏi tòa nhà.

### Why

- **User need:** Tại các trung tâm thương mại, sân bay, bệnh viện lớn — người dùng cần điều hướng trong tòa nhà theo tầng (tìm cửa hàng, cổng ra, thang máy).
- **Business value:** Indoor maps tăng giá trị của Maps Viewer cho các dự án enterprise (mall, airport, hospital navigation).

---

# Part 02 - Specifications

## Backend

_Indoor floor data được cung cấp qua Indoor Tiles API._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                | Nền tảng    | Input                              | AC  |
| --- | ------------------------------------------------------ | ----------- | ---------------------------------- | --- |
| T01 | Zoom in vào tòa nhà có indoor data (zoom ≥ 17)        | Web, Mobile | Building ID từ tile data           | B01 |
| T02 | Tap / click vào tòa nhà khác có indoor data           | Web, Mobile | Building ID mới                    | B02 |
| T03 | Click / tap vào một floor trong danh sách             | Web, Mobile | `floor_id` được chọn              | B03 |
| T04 | Zoom out khỏi ngưỡng indoor (zoom < 16)               | Web, Mobile | zoom level                         | B04 |
| T05 | Click vào indoor POI trên tầng khác                   | Web, Mobile | `floor_id` từ POI data             | B05 |

### States Inventory

| State      | Mô tả                                     | Component                        |
| ---------- | ----------------------------------------- | -------------------------------- |
| `hidden`   | Không có indoor data trong viewport       | Widget ẩn                        |
| `visible`  | Đang trong tòa nhà có indoor data         | Danh sách floor buttons          |
| `active`   | Đang xem một tầng cụ thể                  | Floor button active highlighted  |

### Components, Responsive & Typography

#### Component Inventory

| Component                          | Dùng trong      |
| ---------------------------------- | --------------- |
| Floor Selector panel (vertical)    | T01–T05         |
| Floor button (label + active state)| T03             |
| Building name label (header)       | T01, T02        |

#### Responsive Behavior

| Breakpoint                 | Widget position | Layout       |
| -------------------------- | --------------- | ------------ |
| Mobile portrait (< 768px)  | Right side      | Compact list |
| Tablet, Desktop            | Right side      | Standard list|

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Xuất hiện khi vào indoor (T01)

```gherkin
Given  người dùng zoom in đến zoom ≥ 17 vào tòa nhà có indoor data
Then   Floor Selector xuất hiện với danh sách tầng của tòa nhà
And    tầng ground floor (hoặc tầng phổ biến nhất) được chọn mặc định
And    indoor tile của tầng đó được render trên bản đồ
And    header hiển thị tên tòa nhà
```

#### AC-B02 · Chuyển sang tòa nhà khác (T02)

```gherkin
Given  Floor Selector đang hiển thị cho tòa nhà A
When   người dùng tap vào tòa nhà B (cùng trong viewport)
Then   Floor Selector cập nhật với danh sách tầng của tòa nhà B
And    tầng mặc định của tòa nhà B được chọn
```

#### AC-B03 · Chọn tầng (T03)

```gherkin
Given  Floor Selector đang hiển thị
When   người dùng click / tap vào tầng [2]
Then   indoor tile của tầng 2 render trên bản đồ (tầng cũ ẩn đi)
And    floor button tầng 2 highlight active
And    POI markers của tầng 2 hiển thị
```

#### AC-B04 · Ẩn khi zoom out (T04)

```gherkin
Given  Floor Selector đang hiển thị
When   người dùng zoom out đến zoom < 16
Then   Floor Selector ẩn với fade-out animation
And    indoor tiles bị ẩn, outdoor tiles hiển thị lại bình thường
```

#### AC-B05 · Auto-jump tầng khi click indoor POI (T05)

```gherkin
Given  người dùng click vào indoor POI ở tầng 3 (trong khi đang xem tầng 1)
Then   Floor Selector tự động chọn tầng 3
And    indoor tile cập nhật sang tầng 3
And    Place Detail của POI mở ra
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
