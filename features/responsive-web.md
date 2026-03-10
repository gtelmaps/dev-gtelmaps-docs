# Responsive Web

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

Responsive Web định nghĩa hệ thống breakpoint, layout grid, và hành vi adaptive của Maps Viewer trên các kích thước màn hình khác nhau — từ mobile portrait (320px) đến desktop ultrawide (2560px+). Bao gồm cả các quy tắc về panel stacking, widget repositioning, và typography scaling.

### Why

- **User need:** Người dùng truy cập Maps trên nhiều thiết bị khác nhau — bản đồ và UI phải tối ưu trên mọi màn hình.
- **Business value:** Responsive design là yêu cầu baseline cho SEO, Core Web Vitals, và PWA compliance.

---

## Known Bugs

- **Console scroll bug (Mobile):** Khi mở DevTools console trên mobile Chrome → kéo lên xuống, màu nền console chuyển sang đen. **(Pending fix)**

---

# Part 02 - Specifications

## Backend

_Không có backend — CSS/layout hoàn toàn client-side._

## UI/UX & Frontend

### Breakpoint System

| Name               | Range          | Target devices                       |
| ------------------ | -------------- | ------------------------------------ |
| `xs` (mobile-sm)   | 320px–479px    | Small phones (iPhone SE)             |
| `sm` (mobile)      | 480px–767px    | Standard phones                      |
| `md` (tablet)      | 768px–1023px   | Tablets, large phones landscape      |
| `lg` (desktop)     | 1024px–1279px  | Laptops, small desktop               |
| `xl` (desktop-lg)  | 1280px–1535px  | Standard desktop monitors            |
| `2xl` (ultrawide)  | 1536px+        | Large monitors, ultrawide            |

### Layout Rules Per Breakpoint

| Feature              | xs / sm (Mobile)           | md (Tablet)             | lg / xl / 2xl (Desktop)    |
| -------------------- | -------------------------- | ----------------------- | -------------------------- |
| Search Bar           | Full-width, top            | Fixed 400px, top-center | Fixed 460px, top-left      |
| Place Detail         | Bottom Sheet               | Side Panel 360px        | Side Panel 400px           |
| Search Results       | Bottom Sheet               | Side Panel 360px        | Side Panel 400px           |
| Nearby Results       | Bottom Sheet               | Side Panel 360px        | Side Panel 400px           |
| Directions Panel     | Bottom Sheet full-height   | Side Panel 360px        | Side Panel 400px           |
| Weather Widget       | Bottom-right (small)       | Top-right               | Top-right                  |
| Zoom Widget          | Hidden                     | Bottom-right            | Bottom-right               |
| Compass Widget       | Top-right                  | Bottom-right            | Bottom-right               |
| Category Filter bar  | Scroll horiz, full-width   | Scroll horiz            | Scroll horiz + arrows      |
| Floor Selector       | Right side (compact)       | Right side              | Right side                 |
| Attribution          | Collapsed [©]              | Full text               | Full text                  |
| Context Menu         | N/A (Long press instead)   | N/A                     | Right-click                |

### Triggers & Entry Points

| ID  | Trigger                                    | AC  |
| --- | ------------------------------------------ | --- |
| T01 | Viewport resize (window resize event)      | B01 |
| T02 | Device orientation change (portrait/landscape) | B02 |
| T03 | Virtual keyboard open (mobile)             | B03 |

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Viewport resize (T01)

```gherkin
Given  người dùng resize cửa sổ trình duyệt
Then   layout cập nhật theo breakpoint tương ứng không cần reload
And    panels / widgets reposition smooth
And    bản đồ canvas resize đúng để fill viewport
```

#### AC-B02 · Orientation change (T02)

```gherkin
Given  người dùng xoay thiết bị từ portrait → landscape
Then   layout cập nhật trong vòng 100ms
And    bản đồ canvas resize để tận dụng không gian ngang
And    Bottom Sheet chuyển thành side panel nếu đủ width

Given  xoay ngược lại landscape → portrait
Then   layout restore về Bottom Sheet behavior
```

#### AC-B03 · Virtual keyboard (T03)

```gherkin
Given  người dùng focus vào Search Bar trên mobile
When   virtual keyboard mở (thu nhỏ viewport)
Then   bản đồ canvas thu nhỏ theo viewport mới (không bị che)
And    Search Bar vẫn visible phía trên keyboard
And    Bottom Sheet (nếu có) không bị che bởi keyboard

Given  keyboard đóng
Then   layout restore lại kích thước ban đầu
```

#### AC-B04 · Console scroll bug (Known Bug)

```gherkin
Bug: Mở DevTools console trên mobile Chrome → scroll bản đồ → background console chuyển màu đen.
Root cause: Đang điều tra — nghi do z-index conflict giữa WebGL canvas và console overlay.
Status: Pending fix.
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
