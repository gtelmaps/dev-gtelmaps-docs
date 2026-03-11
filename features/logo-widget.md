# Logo Widget

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

Logo Widget là thành phần hiển thị logo GTEL Maps cố định trên bản đồ. Logo là clickable link điều hướng về trang chủ Maps, và thay đổi variant (full / icon-only) theo breakpoint. Widget cũng hỗ trợ white-label: đối tác enterprise có thể thay thế bằng logo riêng thông qua config.

### Why

- **User need:** Nhận diện thương hiệu và điểm điều hướng về trang chủ khi người dùng muốn reset về trạng thái ban đầu.
- **Business value:** Brand visibility trên mọi instance Maps; white-label config phục vụ khách hàng enterprise (Fleet, Logistics, đối tác B2B).

---

# Part 02 - Specifications

## Backend

_Không có backend riêng. White-label config đọc từ app config / embed params._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Logo UI:** Widget chứa hình ảnh Logo hiển thị cố định ở góc dưới UI bản đồ.
- **Theme/Config App:** Tính năng thay đổi giao diện theo config (light/dark mode) hay thay đổi bộ cấu hình white-label từ đối tác.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                       | Nền tảng    | Input                  | AC  |
| --- | --------------------------------------------- | ----------- | ---------------------- | --- |
| T01 | Bản đồ load lần đầu                           | Web, Mobile | —                      | B01 |
| T02 | Click / tap vào logo                          | Web, Mobile | —                      | B02 |
| T03 | White-label config được cung cấp qua embed    | Web         | `logo_url`, `href`     | B03 |
| T04 | Theme thay đổi (light / dark)                 | Web, Mobile | theme value            | B04 |

### States Inventory

| State        | Mô tả                                    | Component                           |
| ------------ | ---------------------------------------- | ----------------------------------- |
| `default`    | Logo GTEL Maps tiêu chuẩn               | Logo SVG + text (desktop)           |
| `icon_only`  | Chỉ icon, không có text (mobile/compact) | Logo icon SVG                       |
| `white_label`| Logo đối tác thay thế                   | `<img>` từ URL do đối tác cung cấp  |

### Components, Responsive & Typography

#### Component Inventory

| Component                  | Dùng trong          |
| -------------------------- | ------------------- |
| Logo SVG (light variant)   | Default, light theme |
| Logo SVG (dark variant)    | Dark theme           |
| White-label `<img>`        | T03                  |

#### Responsive Behavior

| Breakpoint                 | Logo variant     | Position     |
| -------------------------- | ---------------- | ------------ |
| Mobile portrait (< 768px)  | Icon only        | Bottom-left  |
| Mobile landscape (< 768px) | Icon only        | Bottom-left  |
| Tablet (768px–1024px)      | Full (icon+text) | Bottom-left  |
| Desktop (> 1024px)         | Full (icon+text) | Bottom-left  |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị khi load (T01)

```gherkin
Given  ứng dụng Maps Viewer khởi động
Then   logo hiển thị đúng vị trí theo breakpoint
And    logo load cùng lúc với bản đồ (không lazy load)
```

#### AC-B02 · Click / tap logo (T02)

```gherkin
Given  người dùng click / tap vào logo
Then   điều hướng về trang chủ Maps (href="/{locale}/maps" hoặc config)
And    mở trong cùng tab (không mở tab mới)
And    nếu đang ở trang chủ, không reload trang (SPA navigation)
```

#### AC-B03 · White-label config (T03)

```gherkin
Given  embed config cung cấp logo_url và href
Then   logo GTEL bị thay thế bởi <img src="{logo_url}">
And    click logo điều hướng đến href của đối tác
And    nếu logo_url load lỗi, ẩn logo (không hiển thị broken image)
```

#### AC-B04 · Theme adaptation (T04)

```gherkin
Given  theme là dark mode
Then   logo dùng variant màu trắng / dark-compatible

Given  theme là light mode
Then   logo dùng variant màu tiêu chuẩn
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
