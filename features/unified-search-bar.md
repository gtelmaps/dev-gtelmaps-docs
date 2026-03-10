# Unified Search Bar

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

Unified Search Bar là thanh tìm kiếm trung tâm của Maps Viewer, tích hợp tất cả các loại tìm kiếm: địa chỉ (forward geocoding), địa điểm (POI), tọa độ (reverse geocoding preview), tìm kiếm giọng nói (voice search), và điều hướng nhanh đến các tính năng (Nearby, Directions). Đây là entry point chính của người dùng vào Maps.

### Why

- **User need:** Người dùng muốn một ô nhập duy nhất cho mọi loại tìm kiếm — không phân biệt địa chỉ, địa điểm, hay tọa độ.
- **Business value:** Search Bar là điểm đo lường intent rõ nhất; tích hợp search ads, promoted results và là KPI hàng đầu (search volume, CTR, conversion).

---

# Part 02 - Specifications

## Backend

_Search API (Autocomplete + Forward Geocoding); Reverse Geocoding API; Voice-to-Text API._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                           | Nền tảng    | Input                            | AC  |
| --- | ----------------------------------------------------------------- | ----------- | -------------------------------- | --- |
| T01 | Click / tap vào Search Bar (focus)                               | Web, Mobile | —                                | B01 |
| T02 | Gõ ký tự vào Search Bar                                          | Web, Mobile | Text input (≥ 1 ký tự hợp lệ)   | B02 |
| T03 | Submit tìm kiếm (Enter / tap nút Search)                         | Web, Mobile | Confirmed query                  | B03 |
| T04 | Click nút [×] xóa nội dung Search Bar                            | Web, Mobile | —                                | B04 |
| T05 | Click nút [Mic] — Voice Search                                   | Web, Mobile | —                                | B05 |
| T06 | Click nút [←] Back trong search mode (mobile)                    | Mobile      | —                                | B06 |
| T07 | Bản đồ click / Place Detail mở → điền tên vào Search Bar        | Web, Mobile | `place_name` hoặc address string | B07 |
| T08 | Auto-focus khi app load (desktop)                                | Web         | —                                | B08 |

### States Inventory

| State          | Mô tả                                          | Component                              |
| -------------- | ---------------------------------------------- | -------------------------------------- |
| `idle`         | Search Bar không focused, có placeholder       | Bar thu nhỏ (mobile) / full width (web) |
| `focused_empty`| Focused, chưa nhập gì                          | Recent searches + Nearby suggestions   |
| `typing`       | Đang nhập, có text                             | Autocomplete dropdown hiển thị         |
| `results`      | Đã submit, đang hiển thị kết quả               | Search Results Panel mở                |
| `place_filled` | Tên địa điểm / địa chỉ được điền từ map click | Bar hiển thị tên, [×] button active    |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong                       |
| -------------------------------------- | -------------------------------- |
| Search Input field                     | Tất cả states                    |
| Search icon / Submit button            | idle, typing                     |
| Clear button [×]                       | typing, place_filled             |
| Voice Search button [Mic]              | idle, focused_empty (xem voice-search.md) |
| Back button [←]                        | Mobile focused state             |
| Autocomplete dropdown                  | typing (xem autocomplete-geocoding.md) |
| Recent Searches list                   | focused_empty (xem recent-searches.md) |
| Loading indicator (in bar)             | results loading                  |

#### Responsive Behavior

| Breakpoint                 | Layout                                        | Focus behavior                    |
| -------------------------- | --------------------------------------------- | --------------------------------- |
| Mobile portrait (< 768px)  | Full-width bar, tap expands to search overlay | Keyboard + overlay open           |
| Mobile landscape (< 768px) | Full-width, compact height                   | Similar to portrait               |
| Tablet (768px–1024px)      | Fixed width (400px) top-center               | Dropdown in-place                 |
| Desktop (> 1024px)         | Fixed width (460px) top-left                 | Dropdown in-place, auto-focus     |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Focus vào Search Bar (T01)

```gherkin
Given  người dùng click / tap vào Search Bar
Then   bar nhận focus, cursor đặt vào trong input
And    placeholder ẩn đi
And    Recent Searches hiển thị nếu có lịch sử (xem recent-searches.md)
And    Mobile: search overlay mở (bản đồ bị dim), Back button xuất hiện
```

#### AC-B02 · Gõ text — Autocomplete (T02)

```gherkin
Given  người dùng gõ ≥ 1 ký tự
Then   Autocomplete API được gọi (debounce 300ms)
And    Dropdown hiển thị suggestions (xem autocomplete-geocoding.md)
And    [×] button xuất hiện ở cuối bar
And    gõ tọa độ hợp lệ (vd: "10.77, 106.70") → suggestion "Tìm tọa độ này" xuất hiện
```

#### AC-B03 · Submit tìm kiếm (T03)

```gherkin
Given  người dùng nhấn Enter hoặc tap nút [Search]
Then   Forward Geocoding / Search API được gọi (xem forward-geocoding.md)
And    kết quả hiển thị trong Search Results Panel
And    query được lưu vào Recent Searches
And    URL cập nhật: /search/[encoded_query]/@lat,lng,zoom

Given  query là tọa độ (lat,lng)
Then   Reverse Geocoding được gọi thay thế (xem reverse-geocoding.md §B11)
```

#### AC-B04 · Xóa nội dung [×] (T04)

```gherkin
Given  Search Bar đang có text
When   người dùng click [×]
Then   input bị xóa sạch
And    dropdown đóng
And    focused_empty state (hiện recent searches)
And    Search Results Panel đóng (nếu đang mở)
```

#### AC-B05 · Voice Search (T05)

```gherkin
Given  người dùng click [Mic]
Then   Voice Search mode kích hoạt (xem voice-search.md)
And    kết quả voice → điền vào Search Bar và auto-submit
```

#### AC-B06 · Back button mobile (T06)

```gherkin
Given  Search overlay đang mở trên mobile
When   người dùng tap nút [←]
Then   overlay đóng, keyboard ẩn
And    Search Bar trở về idle state
And    bản đồ dim bị gỡ bỏ
```

#### AC-B07 · Điền tên từ bản đồ (T07)

```gherkin
Given  người dùng click POI marker hoặc map click
Then   tên địa điểm / địa chỉ được điền vào Search Bar
And    bar chuyển sang place_filled state
And    [×] button active để clear
```

#### AC-B08 · Auto-focus desktop (T08)

```gherkin
Given  app load xong trên desktop
Then   Search Bar nhận focus tự động
And    cursor sẵn sàng nhập ngay
And    không auto-focus trên mobile (tránh mở keyboard không mong muốn)
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
