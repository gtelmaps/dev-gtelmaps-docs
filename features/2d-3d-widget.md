# 2D / 3D Widget

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

2D/3D Widget là nút toggle cho phép người dùng chuyển đổi giữa chế độ bản đồ phẳng (2D — tilt = 0°) và chế độ góc nhìn nghiêng (3D — tilt = 45°–60°). Khi ở chế độ 3D, các tòa nhà và địa hình được render với chiều cao (extruded), mang lại trải nghiệm khám phá không gian đô thị trực quan hơn.

### Why

- **User need:** Người dùng muốn xem bản đồ đô thị theo góc nhìn 3D để nhận diện tòa nhà, landmark dễ hơn; đặc biệt hữu ích khi điều hướng bộ hoặc khám phá khu vực mới.
- **Business value:** 3D mode là differentiator của GTEL Maps so với các đối thủ; tăng engagement và thời gian session.

---

# Part 02 - Specifications

## Backend

_3D rendering phụ thuộc vào building extrusion layer và terrain layer được cấu hình trên tile server._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                             | Nền tảng    | Input                         | AC  |
| --- | --------------------------------------------------- | ----------- | ----------------------------- | --- |
| T01 | Click / tap nút [2D] → chuyển sang 3D               | Web, Mobile | —                             | B01 |
| T02 | Click / tap nút [3D] → chuyển về 2D                 | Web, Mobile | —                             | B01 |
| T03 | URL param `tilt=45` khi load                        | Web         | `tilt` value từ URL           | B02 |
| T04 | Thay đổi tilt bằng gesture (right-drag dọc trên web, 2-finger swipe up/down mobile) | Web, Mobile | `tilt` value | B03 |
| T05 | Zoom out đến ngưỡng không hỗ trợ 3D (zoom < 12)    | Web, Mobile | zoom level                    | B04 |

### States Inventory

| State        | Mô tả                                  | Component                            |
| ------------ | -------------------------------------- | ------------------------------------ |
| `2d`         | Tilt = 0°, bản đồ phẳng               | Nút hiển thị label "3D"              |
| `3d`         | Tilt = 45°–60°, góc nghiêng            | Nút hiển thị label "2D"              |
| `unavailable`| Zoom quá thấp, 3D không khả dụng      | Nút disabled hoặc ẩn                 |

### Components, Responsive & Typography

#### Component Inventory

| Component                   | Dùng trong |
| --------------------------- | ---------- |
| Toggle button [2D] / [3D]   | T01–T05    |

#### Responsive Behavior

| Breakpoint                 | Widget position                    | Size    |
| -------------------------- | ---------------------------------- | ------- |
| Mobile portrait (< 768px)  | Bottom-right (trên compass)        | 36×36px |
| Mobile landscape (< 768px) | Bottom-right                       | 36×36px |
| Tablet (768px–1024px)      | Bottom-right (trên compass)        | 40×40px |
| Desktop (> 1024px)         | Bottom-right (trên compass)        | 40×40px |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Toggle 2D ↔ 3D (T01, T02)

```gherkin
Given  bản đồ đang ở chế độ 2D (tilt = 0°)
When   người dùng click nút [3D]
Then   bản đồ animate tilt lên 45° (smooth animation 400ms)
And    building extrusion layer bật lên
And    nút chuyển label thành [2D]
And    URL cập nhật tilt param

Given  bản đồ đang ở chế độ 3D
When   người dùng click nút [2D]
Then   bản đồ animate tilt về 0° (smooth animation 400ms)
And    building extrusion layer tắt
And    nút chuyển label thành [3D]
```

#### AC-B02 · Deep link với tilt param (T03)

```gherkin
Given  URL chứa tilt=45 (hoặc tilt=60)
Then   bản đồ load ở chế độ 3D với tilt tương ứng
And    nút hiển thị label [2D]
```

#### AC-B03 · Gesture tilt (T04)

```gherkin
Given  bản đồ đang hiển thị
When   người dùng right-drag dọc (web) hoặc 2-finger swipe up (mobile)
Then   tilt tăng liên tục theo gesture
And    widget cập nhật label [2D] khi tilt > 0°
```

#### AC-B04 · Disable khi zoom thấp (T05)

```gherkin
Given  zoom level < 12 (ngưỡng không render 3D buildings)
Then   nút [3D] hiển thị disabled
And    tooltip khi hover: "Thu nhỏ bản đồ để xem chế độ 3D"

Given  người dùng zoom in vào zoom ≥ 12
Then   nút [3D] enabled trở lại
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
