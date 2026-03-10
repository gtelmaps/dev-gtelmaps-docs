# Internationalization

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

Internationalization (i18n) định nghĩa cách Maps Viewer hỗ trợ nhiều ngôn ngữ và locale khác nhau: dịch UI strings, format số/ngày/khoảng cách theo locale, RTL layout support (nếu có), và cơ chế chuyển đổi ngôn ngữ. Locale mặc định là `vi-VN`.

### Why

- **User need:** Người dùng muốn xem Maps bằng ngôn ngữ mẹ đẻ; khách quốc tế cần tiếng Anh hoặc ngôn ngữ khác.
- **Business value:** i18n là yêu cầu bắt buộc cho expansion sang thị trường quốc tế và các khách hàng enterprise nước ngoài.

---

# Part 02 - Specifications

## Backend

_i18n resource files (JSON). Locale detection từ Accept-Language header và URL prefix._

## UI/UX & Frontend

### Supported Locales (Phase 1)

| Locale    | Ngôn ngữ         | URL prefix |
| --------- | ---------------- | ---------- |
| `vi-VN`   | Tiếng Việt       | `/vi/`     |
| `en-US`   | English (US)     | `/en/`     |

### Triggers & Entry Points

| ID  | Trigger                                              | Nền tảng    | Input                       | AC  |
| --- | ---------------------------------------------------- | ----------- | --------------------------- | --- |
| T01 | App load — detect locale từ URL prefix               | Web, Mobile | URL path                    | B01 |
| T02 | App load — fallback detect từ browser Accept-Language | Web        | HTTP header                 | B01 |
| T03 | Người dùng chọn ngôn ngữ trong Language Picker       | Web, Mobile | locale code                 | B02 |
| T04 | Locale thay đổi → reformat số, khoảng cách, ngày    | Web, Mobile | locale value                | B03 |

### States Inventory

| State     | Mô tả                    | Component                   |
| --------- | ------------------------ | --------------------------- |
| `vi-VN`   | Locale tiếng Việt        | UI strings + formats VN     |
| `en-US`   | Locale tiếng Anh         | UI strings + formats US     |

### Components, Responsive & Typography

#### Component Inventory

| Component                        | Dùng trong  |
| -------------------------------- | ----------- |
| Language Picker (dropdown/modal) | T03         |
| Language toggle button           | T03         |

#### i18n Format Rules

| Data type      | vi-VN format          | en-US format       |
| -------------- | --------------------- | ------------------ |
| Distance short | `350m`, `1,2km`       | `350m`, `0.8mi`    |
| Distance long  | `12,5km`              | `7.8mi`            |
| Date           | `10/03/2026`          | `Mar 10, 2026`     |
| Time           | `14:30`               | `2:30 PM`          |
| Number         | `1.234,56`            | `1,234.56`         |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Detect locale khi load (T01, T02)

```gherkin
Given  người dùng truy cập /vi/maps
Then   UI render bằng tiếng Việt
And    tất cả strings dùng vi-VN translation keys

Given  người dùng truy cập /en/maps
Then   UI render bằng tiếng Anh

Given  người dùng truy cập /maps (không có prefix)
Then   hệ thống đọc Accept-Language header từ browser
And    redirect về /{detected_locale}/maps
And    fallback về vi-VN nếu không hỗ trợ locale được detect
```

#### AC-B02 · Chuyển đổi ngôn ngữ (T03)

```gherkin
Given  người dùng chọn "English" trong Language Picker
Then   URL thay đổi từ /vi/maps → /en/maps (không reload full page nếu SPA)
And    tất cả UI strings cập nhật sang tiếng Anh ngay lập tức
And    locale preference được lưu vào cookie/LocalStorage

Given  người dùng reload trang sau khi chọn ngôn ngữ
Then   locale preference được restore từ cookie
```

#### AC-B03 · Format theo locale (T04)

```gherkin
Given  locale là vi-VN
Then   khoảng cách hiển thị là "1,2 km" (dấu phẩy là dấu thập phân)
And    đơn vị là m / km
And    ngày tháng format: "10/03/2026"

Given  locale là en-US
Then   khoảng cách hiển thị là "0.7 mi"
And    đơn vị là ft / mi
And    ngày tháng format: "Mar 10, 2026"
```

#### AC-B04 · Missing translation fallback

```gherkin
Given  một translation key không có trong ngôn ngữ hiện tại
Then   fallback về vi-VN string
And    nếu cũng không có trong vi-VN → hiển thị key name (dev mode) hoặc chuỗi rỗng (prod)
And    missing key được log về monitoring để team i18n biết và bổ sung
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
