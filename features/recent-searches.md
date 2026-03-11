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

Recent Searches là tính năng tự động lưu và hiển thị danh sách các từ khóa tìm kiếm cũng như địa điểm gần đây mà người dùng đã xem hoặc tương tác.

### Why

- **User need:** Người dùng thường tìm lại địa điểm đã xem trước đó — Recent Searches giúp họ không cần nhớ hoặc gõ lại.
- **Business value:** Tăng repeat engagement; lịch sử tìm kiếm là tín hiệu personalization quan trọng cho Recommendations và Saved Places.

## Unique Selling Propositions (USP)

> Đối thủ: Tham khảo & Đánh giá

Tính năng xem lại lịch sử tìm kiếm giúp cá nhân hóa và tiện dụng. Khác với các tính năng cơ bản, kiến trúc Recent Searches này mang tới các trải nghiệm vượt trội mà hiện nay các đối thủ chưa hoàn thiện:

- GTEL Maps: Đặc biệt hỗ trợ **gom nhóm theo hành chính**, **bulk select** (lưu hàng loạt, routing đa điểm, chia sẻ tập hợp điểm, xóa hàng loạt), và **visual sync** (hover highlight, zoom to bbox + padding).
- [Google Maps](https://www.google.com/maps): Có lưu lịch sử tìm kiếm rất tốt, là đối thủ duy nhất có trải nghiệm tương tự. Tuy nhiên, Google Maps chưa hỗ trợ gom nhóm theo tỉnh/thành phố và không có các công cụ bulk select (chọn nhiều điểm để thao tác cùng lúc) trực tiếp từ lịch sử.
- Các đơn vị nội địa (Vietmap, Map4D, v.v.): Hầu như chưa sở hữu tính năng Recent Searches hoàn chỉnh, hoặc rất cơ bản dạng lưu text query đơn thuần, hoàn toàn thiếu tính năng tương tác không gian bản đồ và quản lý hàng loạt.

---

# Part 02 - Specifications

## Backend

_Client-side LocalStorage. Đồng bộ lên server nếu người dùng đăng nhập (optional, v2)._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                          | Nền tảng    | Input                      | AC  |
| --- | ------------------------------------------------ | ----------- | -------------------------- | --- |
| T01 | Focus vào Search Bar (trống)                     | Web, Mobile | —                          | B01 |
| T02 | Click / tap vào một recent item                  | Web, Mobile | `query` hoặc `place_id`    | B02 |
| T03 | Click nút [×] xóa một mục trong danh sách        | Web, Mobile | Item index                 | B03 |
| T04 | Click "Xóa tất cả lịch sử"                       | Web, Mobile | —                          | B04 |
| T05 | Search / Place Detail hoàn tất → lưu vào history | Web, Mobile | query hoặc place_id + name | B05 |
| T06 | Hover chuột vào một rercern item                 | Web         | Item index / place_id      | B06 |
| T07 | Tick chọn checkbox trên nhiều recent items       | Web, Mobile | Array of IDs               | B07 |

### States Inventory

| State       | Mô tả           | Component                          |
| ----------- | --------------- | ---------------------------------- |
| `empty`     | Chưa có lịch sử | "Chưa có tìm kiếm nào gần đây"     |
| `has_items` | Có lịch sử      | Danh sách recent items + clear all |

### Components, Responsive & Typography

#### Component Inventory

| Component                                         | Dùng trong   |
| ------------------------------------------------- | ------------ |
| Recent Searches section header                    | T01          |
| Header gom nhóm Tỉnh/Thành phố                    | T01          |
| Recent item row (checkbox + icon + text + [×])    | T01–T03, T07 |
| "Xóa tất cả" button                               | T04          |
| Bulk Actions Toolbar (Save, Route, Share, Delete) | T07          |
| Empty state text                                  | empty state  |

#### Responsive Behavior

_Hiển thị trong Autocomplete dropdown / Search overlay — responsive theo container._

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị khi focus Search Bar (T01)

```gherkin
Given  người dùng focus vào Search Bar khi input trống
Then   danh sách Recent Searches hiển thị thông minh
And    các kết quả địa điểm được **gom nhóm theo đơn vị hành chính cấp Tỉnh/Thành phố** (VD: "Hà Nội", "Hồ Chí Minh")
And    mỗi item bao gồm: checkbox, icon (clock/địa điểm), tên query hoặc địa điểm, [×] để xóa
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

#### AC-B06 · Tương tác Hover và Zoom bản đồ (T06)

```gherkin
Given  danh sách Recent Searches đang mở trên màn hình
When   người dùng di chuột (hover) vào một kết quả liên quan đến địa điểm cụ thể
Then   marker hiển thị của địa điểm đó trên bản đồ tự động thay đổi trạng thái sang **highlight**
When   người dùng nhả hover (mouse out)
Then   marker trở về trạng thái bình thường

Given  người dùng kích hoạt load danh sách kết quả (hoặc click vào nhóm Tỉnh/Thành)
Then   bản đồ tự động tính toán và **Zoom to Bounding Box (Bbox) có cộng thêm padding** để đảm bảo vừa vặn toàn bộ các điểm kết quả trên khung cảnh (Viewport)
```

#### AC-B07 · Tính năng Bulk Select & Actions (T07)

```gherkin
Given  người dùng đang xem danh sách Recent Searches
When   người dùng chọn (tick checkbox) nhiều items (Bulk select)
Then   xuất hiện Bulk Actions Toolbar chứa các tính năng xử lý hàng loạt:
       - **Lưu vào Saved Places**: Lưu tất cả các địa điểm đang được chọn vào một bookmark list duy nhất
       - **Routing đa vị trí**: Tự động chuyển qua module Directions và điền các địa điểm đã chọn vào các waypoint/stop trên cùng 1 lộ trình
       - **Chia sẻ**: Khởi tạo một deep link hoặc hình thức chia sẻ danh sách các địa điểm này cho người khác (Share list)
       - **Xóa hàng loạt**: Xóa toàn bộ các địa điểm đã đánh dấu chọn khỏi danh sách lịch sử tìm kiếm
```

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
