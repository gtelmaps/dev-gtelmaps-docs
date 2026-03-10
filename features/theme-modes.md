# Theme Modes

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

Theme Modes cho phép Maps Viewer chuyển đổi giữa ba chế độ giao diện: Light (sáng), Dark (tối), và System (theo cài đặt OS). Bao gồm cả map tile style thay đổi theo theme và CSS color scheme của toàn bộ UI.

### Why

- **User need:** Dark mode giảm mỏi mắt khi dùng Maps ban đêm hoặc trong môi trường tối; System mode tự động thích nghi với thói quen OS của người dùng.
- **Business value:** Dark mode là tính năng được yêu cầu nhiều nhất trong maps UX surveys; ảnh hưởng tích cực đến evening/night session retention.

---

# Part 02 - Specifications

## Backend

_Map tile server cung cấp dark style tiles riêng. Theme preference lưu LocalStorage._

## UI/UX & Frontend

### Theme Options

| Theme    | Mô tả                                            | Map style       |
| -------- | ------------------------------------------------ | --------------- |
| `light`  | Giao diện sáng (mặc định)                        | Standard light  |
| `dark`   | Giao diện tối                                    | Dark map tiles  |
| `system` | Tự động theo `prefers-color-scheme` của OS       | light hoặc dark |

### Triggers & Entry Points

| ID  | Trigger                                                  | Nền tảng    | Input                       | AC  |
| --- | -------------------------------------------------------- | ----------- | --------------------------- | --- |
| T01 | Người dùng chọn theme trong Settings / Header menu       | Web, Mobile | `theme` value               | B01 |
| T02 | OS thay đổi color scheme (system auto-detect)            | Web, Mobile | `prefers-color-scheme` media | B02 |
| T03 | App load — restore theme từ preference đã lưu            | Web, Mobile | LocalStorage value          | B03 |
| T04 | Thời gian trong ngày (auto-dark sau 18h) — option        | Web, Mobile | Current time                | B04 |

### States Inventory

| State    | Mô tả                  | CSS class / data-attr    |
| -------- | ---------------------- | ------------------------ |
| `light`  | Light theme active     | `data-theme="light"`     |
| `dark`   | Dark theme active      | `data-theme="dark"`      |
| `system` | Theo OS preference     | `data-theme="system"`    |

### Components, Responsive & Typography

#### Component Inventory

| Component                         | Dùng trong    |
| --------------------------------- | ------------- |
| Theme toggle button / picker      | T01           |
| CSS custom properties (--color-*) | Tất cả        |
| Map style switcher (tile URL)     | T01–T04       |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Chuyển đổi theme thủ công (T01)

```gherkin
Given  người dùng chọn "Dark" trong theme picker
Then   CSS data-theme="dark" áp dụng vào root element ngay lập tức (không reload)
And    map tiles chuyển sang dark style
And    logo, icon, text colors cập nhật theo dark palette
And    preference được lưu vào LocalStorage
And    transition smooth (CSS transition 200ms trên color properties)
```

#### AC-B02 · Auto-detect từ OS (T02)

```gherkin
Given  theme preference là "System"
And    OS đang dùng dark mode
Then   app hiển thị dark theme tự động

Given  user thay đổi OS sang light mode khi app đang mở
Then   app chuyển về light theme không cần reload (media query listener)
```

#### AC-B03 · Restore theme khi load (T03)

```gherkin
Given  lần trước người dùng đã chọn dark theme
When   app reload
Then   dark theme được apply ngay từ khi render (tránh flash of wrong theme)
And    không có FOUC (Flash Of Unstyled Content)
```

#### AC-B04 · Auto-dark theo giờ (T04, optional)

```gherkin
Given  người dùng bật tính năng "Tự động tối theo giờ"
When   giờ hiện tại >= 18:00 hoặc < 06:00
Then   theme tự động chuyển sang dark
When   giờ hiện tại trong khoảng 06:00–18:00
Then   theme tự động chuyển về light
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
