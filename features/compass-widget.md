# Compass Widget

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

Compass Widget là icon la bàn cố định trên giao diện bản đồ, hiển thị hướng Bắc hiện tại thông qua góc xoay của kim la bàn. Widget tự ẩn khi bản đồ hướng Bắc (heading = 0°) và xuất hiện khi bản đồ bị xoay, đồng thời cho phép người dùng reset về hướng Bắc bằng một click.

### Why

- **User need:** Khi người dùng xoay bản đồ để căn hướng di chuyển hoặc trong 3D/tilt view, họ dễ mất định hướng không gian. Compass Widget cung cấp tham chiếu hướng Bắc trực quan và phím tắt reset nhanh.
- **Business value:** Thành phần cơ bản của Maps UX; cải thiện spatial awareness trong Navigation mode và 3D view.

---

# Part 02 - Specifications

## Backend

_Không có backend riêng — hoàn toàn là client-side._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                            | Nền tảng    | Input                    | AC  |
| --- | ------------------------------------------------------------------ | ----------- | ------------------------ | --- |
| T01 | Bản đồ bị xoay (heading thay đổi từ 0°)                           | Web, Mobile | `heading` (degrees)      | B01 |
| T02 | Click / tap vào Compass Widget                                     | Web, Mobile | —                        | B02 |
| T03 | Xoay bản đồ bằng gesture (right-drag web / 2-finger rotate mobile) | Web, Mobile | `heading` mới realtime   | B03 |
| T04 | Bản đồ trở về heading = 0°                                         | Web, Mobile | `heading = 0`            | B04 |
| T05 | Phím tắt reset heading [N] hoặc Shift+[↑]                         | Web         | Keyboard event           | B02 |

### States Inventory

| State     | Mô tả                              | Component                              |
| --------- | ---------------------------------- | -------------------------------------- |
| `hidden`  | Heading = 0° — bản đồ hướng Bắc   | Widget ẩn (display: none)              |
| `visible` | Heading ≠ 0° — bản đồ đang xoay   | Widget hiển thị, kim xoay theo heading |

### Components, Responsive & Typography

#### Component Inventory

| Component                         | Dùng trong |
| --------------------------------- | ---------- |
| Compass icon (SVG, kim la bàn)    | T01–T05    |
| CSS rotate transform (realtime)   | T03        |

#### Responsive Behavior

| Breakpoint                 | Widget position                    | Size    |
| -------------------------- | ---------------------------------- | ------- |
| Mobile portrait (< 768px)  | Top-right                          | 40×40px |
| Mobile landscape (< 768px) | Top-right                          | 40×40px |
| Tablet (768px–1024px)      | Bottom-right (phía trên Zoom btns) | 40×40px |
| Desktop (> 1024px)         | Bottom-right (phía trên Zoom btns) | 40×40px |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị / ẩn theo heading (T01, T04)

```gherkin
Given  bản đồ đang ở heading = 0° (hướng Bắc)
Then   Compass Widget ẩn hoàn toàn (không chiếm không gian layout)

Given  bản đồ bị xoay sang heading ≠ 0°
Then   widget xuất hiện với fade-in animation (150ms)
And    kim la bàn xoay đúng góc để hướng Bắc thực tế luôn chỉ lên trên

Given  bản đồ reset về heading = 0°
Then   widget ẩn với fade-out animation (150ms)
```

#### AC-B02 · Click / tap để reset Bắc (T02, T05)

```gherkin
Given  Compass Widget đang visible
When   người dùng click / tap vào widget
Then   bản đồ animate về heading = 0° (smooth rotation, 300ms ease-out)
And    widget ẩn sau khi animation hoàn tất

Given  người dùng nhấn phím [N] hoặc Shift+[↑]
Then   tương tự hành vi click widget
```

#### AC-B03 · Cập nhật góc realtime khi xoay (T03)

```gherkin
Given  người dùng đang xoay bản đồ bằng gesture / keyboard
Then   kim la bàn cập nhật góc realtime theo heading bản đồ
And    không có delay > 16ms (đảm bảo 60fps sync)
And    widget không flicker trong quá trình xoay
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
