# Autocomplete Geocoding

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

Autocomplete Geocoding gợi ý danh sách địa chỉ, địa điểm, và danh mục khi người dùng đang gõ vào Search Bar. Suggestions được trả về theo thứ tự độ liên quan kết hợp với vị trí hiện tại (location bias) và lịch sử tìm kiếm.

### Why

- **User need:** Người dùng không cần nhập đầy đủ địa chỉ — autocomplete giúp hoàn thành query nhanh hơn 60–70% số ký tự cần gõ, giảm friction đáng kể.
- **Business value:** Autocomplete cải thiện search conversion rate; là điểm inject Promoted Suggestions; cung cấp tín hiệu intent realtime.

## Unique Selling Propositions (USP)

| #  | USP                                | Mô tả                                                                                        | So sánh                                              |
| -- | ---------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Hỗ trợ không dấu + có dấu         | Gõ "nguyen hue" vẫn gợi ý "Nguyễn Huệ" — normalize tiếng Việt tự động                       | Google Maps cũng hỗ trợ nhưng kém chính xác cho VN   |
| 2  | Promoted Suggestions               | Hỗ trợ inject kết quả quảng cáo vào danh sách gợi ý — monetization native                    | Google Maps không mở tính năng này cho bên thứ ba     |
| 3  | Location bias + lịch sử cá nhân    | Kết hợp vị trí hiện tại và recent searches để xếp hạng gợi ý — chính xác hơn theo ngữ cảnh   | Google Maps có location bias nhưng không dùng history |
| 4  | Session token billing              | Nhóm autocomplete + select thành 1 session — tiết kiệm chi phí cho enterprise                | Google Maps cũng có session token                    |
| 5  | Self-hosted, latency thấp nội địa  | Server đặt tại VN — P95 ≤ 200ms, nhanh hơn so với call đến server Google ở nước ngoài         | Google Maps server ở Singapore/US                    |

---

# Part 02 - Specifications

## Backend

_Autocomplete API (Places Autocomplete endpoint)._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Search Bar (Thanh tìm kiếm):** Component nằm ở ngoài màn hình map chính (Web/Mobile). Người dùng click/tap vào đây và bắt đầu thao tác để kích hoạt tính năng Autocomplete.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                       | Nền tảng    | Input                                   | AC  |
| --- | --------------------------------------------- | ----------- | --------------------------------------- | --- |
| T01 | Gõ ≥ 1 ký tự hợp lệ vào Search Bar            | Web, Mobile | Text string (debounce 300ms)            | B01 |
| T02 | Gõ thêm ký tự / xóa bớt ký tự (còn ≥ 1 ký tự) | Web, Mobile | Text string mới                         | B01 |
| T03 | Nhấn phím ↑ / ↓ để điều hướng trong dropdown  | Web         | Keyboard event                          | B02 |
| T04 | Click / tap vào một suggestion                | Web, Mobile | `place_id` hoặc `address` từ suggestion | B03 |
| T05 | Nhấn Escape hoặc blur khỏi Search Bar         | Web, Mobile | —                                       | B04 |
| T06 | Gõ tọa độ (decimal degrees hoặc DMS format)   | Web, Mobile | Coordinate string                       | B05 |
| T07 | Click "Add a missing place to GTEL Maps"      | Web, Mobile | Click event                             | B06 |
| T08 | Xóa hết ký tự (Search Bar rỗng)               | Web, Mobile | Text rỗng                               | B01 |

### States Inventory

| State          | Mô tả                               | Component                          |
| -------------- | ----------------------------------- | ---------------------------------- |
| `hidden`       | Search Bar trống hoặc không focused | Dropdown ẩn                        |
| `loading`      | Đang gọi Autocomplete API           | Skeleton rows trong dropdown       |
| `suggestions`  | Có kết quả gợi ý                    | Dropdown với danh sách items       |
| `zero_results` | Không có gợi ý phù hợp              | "Add a missing place to GTEL Maps" |
| `error`        | API lỗi                             | Dropdown ẩn (silent fail)          |

### Components, Responsive & Typography

#### Component Inventory

| Component                                         | Dùng trong                         |
| ------------------------------------------------- | ---------------------------------- |
| Autocomplete dropdown container                   | Tất cả states                      |
| Suggestion item (icon + primary + secondary text) | suggestions                        |
| Suggestion item — History variant                 | Icon clock cho kết quả từ Lịch sử  |
| Suggestion item — Geometry variant                | Icon phân biệt Điểm / Đường / Vùng |
| Suggestion item — POI variant                     | POI suggestions                    |
| Suggestion item — Address variant                 | Address suggestions                |
| Suggestion item — Category variant                | "Tìm [category] gần đây"           |
| Suggestion item — Coordinate variant              | B05                                |
| Suggestion item — Add missing place               | zero_results                       |
| Skeleton rows                                     | loading state                      |
| Keyboard focus highlight                          | B02                                |

#### Responsive Behavior

| Breakpoint                | Dropdown layout                           |
| ------------------------- | ----------------------------------------- |
| Mobile portrait (< 768px) | Full-width, chiếm toàn search overlay     |
| Tablet, Desktop           | Dropdown bên dưới Search Bar, max 6 items |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị suggestions khi gõ (T01, T02)

```gherkin
Given  người dùng gõ ≥ 1 ký tự hợp lệ vào Search Bar
Then   Autocomplete API được gọi sau debounce 300ms
And    dropdown hiển thị tối đa 6 suggestions
And    mỗi suggestion gồm: icon (phân biệt: từ history, geometry Điểm/Đường/Vùng, hoặc danh mục POI), primary text (tên/địa chỉ), secondary text (thành phố/tỉnh)
And    location bias áp dụng: ưu tiên kết quả gần map center

Given  người dùng gõ thêm hoặc xóa bớt ký tự (nhưng text vẫn ≥ 1 ký tự)
Then   Autocomplete API được gọi lại (debounce reset)
And    dropdown cập nhật kết quả mới (previous request bị cancel nếu chưa resolve)

Given  người dùng xóa hết ký tự trong Search Bar (text rỗng)
Then   dropdown đóng lại (tắt autocomplete)
And    trạng thái chuyển về hidden
```

#### AC-B02 · Keyboard navigation (T03)

```gherkin
Given  dropdown đang hiển thị suggestions
When   người dùng nhấn phím [↓]
Then   highlight chuyển xuống item kế tiếp
And    Search Bar input tự động điền (fill) bằng primary text của item được highlight

When   nhấn phím [↑]
Then   highlight lên item trước
And    Search Bar input tự động điền (fill) bằng primary text của item được highlight
And    nếu đã ở item đầu tiên → về lại text gốc người dùng đã gõ ban đầu

When   nhấn [Enter] trên item đang highlight
Then   tương tự click vào item đó (B03)
```

#### AC-B03 · Chọn suggestion (T04)

```gherkin
Given  người dùng click / tap vào suggestion
Then   Search Bar điền text của suggestion
And    dropdown đóng
And    nếu suggestion có place_id → Place Detail mở (xem place-detail.md §B03)
And    nếu suggestion là address → Forward Geocoding thực hiện
And    suggestion được lưu vào Recent Searches
```

#### AC-B04 · Đóng dropdown (T05)

```gherkin
Given  dropdown đang mở
When   người dùng nhấn Escape hoặc click ra ngoài
Then   dropdown đóng
And    Search Bar giữ nguyên text hiện tại
```

#### AC-B05 · Nhập tọa độ (T06)

```gherkin
Given  người dùng nhập chuỗi có dạng tọa độ ("10.77, 106.70" hoặc "10°46'N 106°42'E")
Then   dropdown hiển thị suggestion đặc biệt: "Xem tọa độ 10.77, 106.70"
And    chọn suggestion → Reverse Geocoding tại tọa độ đó (xem reverse-geocoding.md §B11)
```

#### AC-B06 · Add a missing place (T07)

```gherkin
Given  kết quả trả về là zero_results
Then   dropdown hiển thị tùy chọn "Add a missing place to GTEL Maps"
When   người dùng click / tap vào tùy chọn này
Then   dropdown đóng
And    form thêm địa điểm được mở ra
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
