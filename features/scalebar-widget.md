# Scalebar Widget

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

Scalebar Widget là thanh tỷ lệ bản đồ hiển thị cố định góc dưới bản đồ, cho biết khoảng cách thực tế tương ứng với một đoạn pixel trên màn hình. Widget cập nhật tự động khi zoom level hoặc latitude thay đổi (do Mercator projection distortion), và hiển thị đơn vị phù hợp với locale (m/km hoặc ft/mi).

### Why

- **User need:** Người dùng cần ước lượng khoảng cách trực quan — "từ điểm A đến B khoảng bao nhiêu km?" — mà không cần mở công cụ đo riêng.
- **Business value:** Thành phần chuẩn của Maps UX; đặc biệt quan trọng trong Fleet/Logistics khi cần ước tính khoảng cách nhanh trên bản đồ.

---

# Part 02 - Specifications

## Backend

_Không có backend riêng — tính toán client-side dựa trên projection._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                     | Nền tảng    | Input                        | AC  |
| --- | ------------------------------------------- | ----------- | ---------------------------- | --- |
| T01 | Bản đồ load lần đầu                         | Web, Mobile | zoom level + latitude        | B01 |
| T02 | Zoom level thay đổi                         | Web, Mobile | zoom level mới               | B01 |
| T03 | Bản đồ pan dọc (latitude thay đổi đáng kể)  | Web, Mobile | latitude mới của map center  | B02 |
| T04 | Locale / đơn vị đo thay đổi (km ↔ mi)      | Web, Mobile | locale setting               | B03 |

### States Inventory

| State     | Mô tả                           | Component                     |
| --------- | ------------------------------- | ----------------------------- |
| `visible` | Bản đồ đang hiển thị            | Scalebar với giá trị hiện tại |

### Components, Responsive & Typography

#### Component Inventory

| Component                       | Dùng trong |
| ------------------------------- | ---------- |
| Scalebar line + end caps (SVG)  | Tất cả     |
| Distance label (số + đơn vị)    | Tất cả     |

#### Responsive Behavior

| Breakpoint                 | Widget position | Width      |
| -------------------------- | --------------- | ---------- |
| Mobile portrait (< 768px)  | Bottom-left     | 60–80px    |
| Mobile landscape           | Bottom-left     | 80–100px   |
| Tablet (768px–1024px)      | Bottom-left     | 80–100px   |
| Desktop (> 1024px)         | Bottom-left     | 100–120px  |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Cập nhật theo zoom (T01, T02)

```gherkin
Given  bản đồ load hoặc zoom level thay đổi
Then   scalebar tính lại: distance = pixelWidth × metersPerPixel(zoom, lat)
And    giá trị hiển thị được làm tròn đến số "đẹp" (1, 2, 5, 10, 20, 50, 100, 200, 500...)
And    chiều dài thanh bar điều chỉnh để phù hợp với giá trị làm tròn (không cố định px)
And    cập nhật tức thời, không có animation lag
```

#### AC-B02 · Cập nhật theo latitude (T03)

```gherkin
Given  bản đồ pan dọc theo vĩ độ
When   latitude thay đổi > 5°
Then   scalebar recalculate theo Mercator distortion tại latitude mới
And    giá trị cập nhật đúng
```

#### AC-B03 · Chuyển đổi đơn vị (T04)

```gherkin
Given  locale là vi-VN hoặc metric system
Then   scalebar hiển thị đơn vị m / km

Given  locale là en-US hoặc imperial system
Then   scalebar hiển thị đơn vị ft / mi
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
