# Error Pages

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

Error Pages định nghĩa các trang lỗi HTTP và trạng thái đặc biệt của Maps Viewer: 404 Not Found, 500 Internal Server Error, 503 Service Unavailable, trang Offline (mất kết nối mạng), và trang Maintenance. Mỗi trang cung cấp thông tin rõ ràng và hành động khôi phục phù hợp.

### Why

- **User need:** Người dùng cần biết chuyện gì đang xảy ra và làm gì tiếp theo — trang lỗi trắng hoặc technical error message gây frustration và mất trust.
- **Business value:** Trang lỗi được thiết kế tốt giảm bounce rate và duy trì trust thương hiệu kể cả khi hệ thống có sự cố.

---

# Part 02 - Specifications

## Backend

_HTTP error response codes. Server-Side Rendering error handling._

## UI/UX & Frontend

### Error Page Inventory

| Code / Type  | Tiêu đề                           | Mô tả user-facing                                        | CTA                          |
| ------------ | --------------------------------- | --------------------------------------------------------- | ---------------------------- |
| 404          | Không tìm thấy trang              | "Trang bạn tìm kiếm không tồn tại hoặc đã bị xóa."      | [Về trang chủ]               |
| 500          | Lỗi hệ thống                      | "Đã có lỗi xảy ra từ phía chúng tôi. Đang được xử lý."  | [Thử lại] [Về trang chủ]    |
| 503          | Dịch vụ tạm ngưng                 | "Dịch vụ Maps tạm thời không khả dụng."                  | [Thử lại sau]                |
| Offline      | Mất kết nối                       | "Không có kết nối internet. Kiểm tra mạng và thử lại."   | [Thử lại]                    |
| Maintenance  | Bảo trì hệ thống                  | "GTEL Maps đang bảo trì. Dự kiến hoàn tất lúc [time]."  | [Thông báo khi xong] (opt)  |

### Triggers & Entry Points

| ID  | Trigger                                      | AC  |
| --- | -------------------------------------------- | --- |
| T01 | Server trả về HTTP 404                       | B01 |
| T02 | Server trả về HTTP 500 / 502 / 503           | B02 |
| T03 | Mất kết nối internet (offline event)         | B03 |
| T04 | Maintenance mode được bật trên server        | B04 |

### Components, Responsive & Typography

#### Component Inventory

| Component                                | Dùng trong     |
| ---------------------------------------- | -------------- |
| Error illustration (SVG, per error type) | T01–T04        |
| Error title (h1)                         | T01–T04        |
| Error description text                   | T01–T04        |
| CTA button(s)                            | T01–T04        |
| Maintenance countdown timer (optional)   | T04            |
| Offline retry auto-detection             | T03            |

#### Responsive Behavior

_Centered layout, full-page, tương thích mọi breakpoint._

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · 404 Not Found

```gherkin
Given  người dùng truy cập URL không tồn tại (ví dụ: /place/invalid_id)
Then   trang 404 hiển thị với illustration, tiêu đề và mô tả
And    nút [Về trang chủ] điều hướng về /{locale}/maps
And    HTTP status code trong response là 404 (SSR)
And    không index bởi search engines (noindex meta tag)
```

#### AC-B02 · 500 / 503 Server Error

```gherkin
Given  server trả về 500 hoặc 503
Then   trang lỗi tương ứng hiển thị
And    nút [Thử lại] reload trang hiện tại
And    nút [Về trang chủ] điều hướng về home
And    lỗi được log về monitoring service với request ID
```

#### AC-B03 · Offline state

```gherkin
Given  trình duyệt mất kết nối internet (navigator.onLine = false)
Then   offline banner / trang hiển thị (nếu đang ở maps thì overlay)
And    "Kiểm tra kết nối mạng của bạn"
And    app lắng nghe online event — khi có mạng trở lại, tự động retry và dismiss overlay
And    không cần người dùng refresh trang
```

#### AC-B04 · Maintenance mode

```gherkin
Given  server trả về response maintenance (503 + Retry-After header)
Then   trang Maintenance hiển thị với estimated time (nếu có)
And    không có CTA "Thử lại" (vì server đang down có chủ ý)
And    trang tự động refresh khi qua estimated time
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
