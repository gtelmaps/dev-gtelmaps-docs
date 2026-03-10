# Recent Searches

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

Recent Searches lưu và hiển thị lịch sử tìm kiếm gần đây của người dùng (queries, địa điểm đã xem, tọa độ đã tra) trong dropdown khi focus vào Search Bar. Người dùng có thể tái sử dụng lịch sử, xóa từng mục hoặc xóa toàn bộ.

### Why

- **User need:** Người dùng thường tìm lại địa điểm đã xem trước đó — Recent Searches giúp họ không cần nhớ hoặc gõ lại.
- **Business value:** Tăng repeat engagement; lịch sử tìm kiếm là tín hiệu personalization quan trọng cho Recommendations và Saved Places.

---

# Part 02 - Specifications

## Backend

_Client-side LocalStorage. Đồng bộ lên server nếu người dùng đăng nhập (optional, v2)._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                              | Nền tảng    | Input                        | AC  |
| --- | ---------------------------------------------------- | ----------- | ---------------------------- | --- |
| T01 | Focus vào Search Bar (trống)                         | Web, Mobile | —                            | B01 |
| T02 | Click / tap vào một recent item                      | Web, Mobile | `query` hoặc `place_id`      | B02 |
| T03 | Click nút [×] xóa một mục trong danh sách           | Web, Mobile | Item index                   | B03 |
| T04 | Click "Xóa tất cả lịch sử"                           | Web, Mobile | —                            | B04 |
| T05 | Search / Place Detail hoàn tất → lưu vào history    | Web, Mobile | query hoặc place_id + name   | B05 |

### States Inventory

| State     | Mô tả                                | Component                           |
| --------- | ------------------------------------ | ----------------------------------- |
| `empty`   | Chưa có lịch sử                      | "Chưa có tìm kiếm nào gần đây"      |
| `has_items` | Có lịch sử                         | Danh sách recent items + clear all  |

### Components, Responsive & Typography

#### Component Inventory

| Component                             | Dùng trong     |
| ------------------------------------- | -------------- |
| Recent Searches section header        | T01            |
| Recent item row (icon + text + [×])   | T01–T03        |
| "Xóa tất cả" button                   | T04            |
| Empty state text                      | empty state    |

#### Responsive Behavior

_Hiển thị trong Autocomplete dropdown / Search overlay — responsive theo container._

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị khi focus Search Bar (T01)

```gherkin
Given  người dùng focus vào Search Bar khi input trống
Then   danh sách Recent Searches hiển thị (tối đa 5 items gần nhất)
And    mỗi item gồm: icon (clock/địa điểm), tên query hoặc tên địa điểm, [×] để xóa
And    nút "Xóa tất cả lịch sử" ở cuối danh sách

Given  không có lịch sử nào
Then   hiển thị "Chưa có tìm kiếm nào gần đây"
```

#### AC-B02 · Chọn recent item (T02)

```gherkin
Given  người dùng click / tap vào một recent item
Then   nếu item là query text → điền vào Search Bar và auto-submit search
And    nếu item là place_id → mở Place Detail trực tiếp (xem place-detail.md §B03)
And    item được đưa lên đầu danh sách (most recent)
```

#### AC-B03 · Xóa một mục (T03)

```gherkin
Given  người dùng click [×] trên một recent item
Then   item bị xóa khỏi danh sách ngay lập tức
And    danh sách còn lại cập nhật
And    LocalStorage cập nhật
```

#### AC-B04 · Xóa tất cả (T04)

```gherkin
Given  người dùng click "Xóa tất cả lịch sử"
Then   confirmation dialog hiện ra: "Xóa toàn bộ lịch sử tìm kiếm?"
And    nếu confirm → danh sách xóa sạch, LocalStorage clear
And    nếu cancel → danh sách giữ nguyên
```

#### AC-B05 · Lưu lịch sử mới (T05)

```gherkin
Given  người dùng hoàn tất một search query
Then   query được thêm vào đầu danh sách Recent Searches
And    nếu đã tồn tại query giống → di chuyển lên đầu (không tạo duplicate)
And    danh sách giới hạn tối đa 20 items (FIFO, xóa item cũ nhất khi full)

Given  người dùng xem Place Detail
Then   place_id + tên địa điểm được lưu vào Recent Searches dạng place item
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
