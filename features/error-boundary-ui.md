# Error Boundary UI

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

Error Boundary UI định nghĩa cơ chế bắt và xử lý lỗi runtime ở tầng React component (React Error Boundaries) và các unhandled exceptions trong Maps Viewer. Mục tiêu là ngăn lỗi một component làm crash toàn bộ ứng dụng, thay vào đó hiển thị fallback UI phù hợp và log lỗi về monitoring.

### Why

- **User need:** Khi một tính năng bị lỗi, người dùng không nên thấy trang trắng — họ cần được thông báo rõ ràng và có cách khôi phục (retry, reload).
- **Business value:** Error boundaries giảm tỷ lệ full-crash của app; cải thiện error observability qua structured logging.

---

# Part 02 - Specifications

## Backend

_Error logging / monitoring service (Sentry hoặc tương đương)._

## UI/UX & Frontend

### Error Boundary Hierarchy

```
App (Root Boundary)
├── Map Canvas Boundary
│   └── Map render errors → fallback: "Không thể tải bản đồ"
├── Search Panel Boundary
│   └── Search/Autocomplete errors → fallback: retry inline
├── Place Detail Boundary
│   └── Panel render errors → fallback: "Không thể hiển thị địa điểm"
├── Directions Boundary
│   └── Route panel errors → fallback: retry inline
└── Widget Boundaries (Weather, Floors...)
    └── Widget errors → widget ẩn (silent)
```

### Triggers & Entry Points

| ID  | Trigger                                                   | AC  |
| --- | --------------------------------------------------------- | --- |
| T01 | React component throw trong render / lifecycle            | B01 |
| T02 | Unhandled Promise rejection                               | B02 |
| T03 | Map engine (WebGL) crash / context lost                   | B03 |
| T04 | Chunk load failure (lazy-loaded module)                   | B04 |

### States Inventory

| State          | Mô tả                                   | Component                          |
| -------------- | --------------------------------------- | ---------------------------------- |
| `normal`       | Không có lỗi                            | UI bình thường                     |
| `component_error` | Component con lỗi, boundary bắt được | Fallback UI trong boundary scope   |
| `map_crash`    | Map canvas không render được            | Full-map fallback overlay          |
| `chunk_error`  | Lazy chunk load thất bại                | Prompt reload trang                |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong          |
| -------------------------------------- | ------------------- |
| Component fallback (inline)            | T01                 |
| Map crash overlay                      | T03                 |
| Chunk error banner                     | T04                 |
| Retry button                           | T01–T04             |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Component lỗi — Boundary bắt (T01)

```gherkin
Given  một React component (ví dụ: Place Detail Panel) throw error trong render
Then   Error Boundary của panel đó bắt error
And    fallback UI hiển thị trong scope của panel: "[Icon cảnh báo] Có lỗi xảy ra. [Thử lại]"
And    phần còn lại của app (bản đồ, search bar) vẫn hoạt động bình thường
And    lỗi được log về monitoring service với stack trace đầy đủ
```

#### AC-B02 · Unhandled Promise rejection (T02)

```gherkin
Given  một async operation reject mà không có catch handler
Then   global error handler bắt rejection
And    toast lỗi chung hiển thị nếu ảnh hưởng đến UX
And    lỗi được log về monitoring service
And    app không crash
```

#### AC-B03 · WebGL context lost (T03)

```gherkin
Given  GPU context bị mất (tab bị ẩn lâu, GPU crash, thiếu VRAM)
Then   Map canvas phát hiện "webglcontextlost" event
And    overlay hiển thị: "Bản đồ bị gián đoạn. [Tải lại bản đồ]"
And    click "Tải lại bản đồ" → reinitialize WebGL context
And    trạng thái bản đồ (position, zoom) được restore sau khi reinit
```

#### AC-B04 · Lazy chunk load failure (T04)

```gherkin
Given  lazy-loaded module (React.lazy) fail to load (network error)
Then   banner dính đầu trang: "Cập nhật mới có sẵn. Vui lòng tải lại trang."
And    nút [Tải lại] reload full page
And    không hiển thị stack trace cho end user
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
