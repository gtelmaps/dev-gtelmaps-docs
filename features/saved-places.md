# Saved Places

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

Saved Places cho phép người dùng lưu địa điểm vào các danh sách cá nhân (Yêu thích, Muốn đến, Đã đến, danh sách tùy chỉnh). Địa điểm đã lưu hiển thị với marker riêng trên bản đồ và có thể truy cập nhanh qua Search Bar hoặc menu Saved Places.

### Why

- **User need:** Người dùng muốn lưu địa điểm để truy cập lại mà không cần tìm kiếm lại — "quán ăn yêu thích", "địa điểm muốn ghé thăm".
- **Business value:** Saved Places tăng DAU/WAU retention; là entry point cho Directions và Sharing; dữ liệu lưu là tín hiệu cá nhân hóa mạnh nhất.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                     | So sánh                                              |
| -- | -------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Guest-mode saved places          | Lưu địa điểm vào localStorage cho guest users — không cần đăng nhập                       | Google Maps yêu cầu Google account                   |
| 2  | Custom lists                     | Tạo danh sách tùy chỉnh (không giới hạn tên) ngoài 3 list mặc định                       | Google Maps cũng có custom lists                     |
| 3  | Marker riêng trên bản đồ         | Địa điểm đã lưu hiển thị icon đặc biệt trên bản đồ — nhận diện nhanh                     | Google Maps cũng hiển thị saved places markers       |
| 4  | Quick access từ Search Bar       | Gõ vào search → gợi ý từ saved places trước — truy cập nhanh hơn                          | Google Maps cũng tích hợp nhưng ít ưu tiên           |

---

# Part 02 - Specifications

## Backend

_Saved Places API (requires authentication). LocalStorage fallback cho guest users._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                        | Nền tảng    | Input                              | AC  |
| --- | -------------------------------------------------------------- | ----------- | ---------------------------------- | --- |
| T01 | Click nút [Lưu] trong Place Detail Action Bar                  | Web, Mobile | `place_id`                         | B01 |
| T02 | Click nút [Lưu] trong Search Result item                       | Web, Mobile | `place_id`                         | B01 |
| T03 | Click nút [♥] đang active để bỏ lưu (toggle off)              | Web, Mobile | `place_id`                         | B02 |
| T04 | Mở Saved Places panel từ menu / Search Bar                     | Web, Mobile | —                                  | B03 |
| T05 | Click item trong Saved Places list                             | Web, Mobile | `place_id`                         | B04 |
| T06 | Thêm / đổi tên / xóa danh sách tùy chỉnh                      | Web, Mobile | `list_name`                        | B05 |

### States Inventory

| State       | Mô tả                                    | Component                             |
| ----------- | ---------------------------------------- | ------------------------------------- |
| `unsaved`   | Địa điểm chưa được lưu                   | Nút [Lưu] outline                     |
| `saved`     | Địa điểm đã được lưu vào ít nhất 1 list | Nút [Lưu] filled / active             |
| `saving`    | Đang gọi API lưu                         | Nút spinner ngắn (optimistic update)  |

### Components, Responsive & Typography

#### Component Inventory

| Component                                  | Dùng trong     |
| ------------------------------------------ | -------------- |
| Save button [Lưu] (Place Detail)           | T01            |
| Save button inline (Search Result item)    | T02            |
| List picker bottom sheet / popover         | T01, T02       |
| Saved Places panel                         | T04            |
| Saved Place marker (trên bản đồ)           | T04            |
| Saved Place list item                      | T05            |
| Custom list management UI                  | T06            |

#### Responsive Behavior

| Breakpoint                 | Save flow            | Saved Places panel      |
| -------------------------- | -------------------- | ----------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet list picker | Bottom Sheet full   |
| Desktop                    | Popover list picker  | Side panel (400px)      |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Lưu địa điểm (T01, T02)

```gherkin
Given  người dùng click [Lưu] trong Place Detail hoặc Result item
Then   list picker mở với các danh sách: Yêu thích ♥, Muốn đến 🕐, Đã đến ✓, + tùy chỉnh
And    người dùng chọn danh sách → địa điểm được lưu (optimistic update)
And    nút [Lưu] chuyển sang filled/active state ngay lập tức
And    toast: "Đã lưu vào [Tên danh sách]"

Given  người dùng chưa đăng nhập
Then   hiện prompt đăng nhập trước khi lưu
And    sau khi đăng nhập → tiếp tục flow lưu
```

#### AC-B02 · Bỏ lưu (T03)

```gherkin
Given  địa điểm đang được lưu
When   người dùng click nút [Lưu] đang active
Then   list picker mở với danh sách hiện tại (checked)
And    người dùng bỏ check → địa điểm xóa khỏi danh sách đó
And    nếu xóa khỏi tất cả danh sách → nút về unsaved state
And    marker Saved Places ẩn khỏi bản đồ
```

#### AC-B03 · Mở Saved Places panel (T04)

```gherkin
Given  người dùng mở Saved Places panel
Then   panel hiển thị danh sách (Yêu thích, Muốn đến, Đã đến, custom lists)
And    click vào danh sách → hiển thị các địa điểm trong danh sách đó
And    markers của tất cả saved places trong danh sách hiển thị trên bản đồ (màu riêng)
And    bản đồ fit bounds để thấy tất cả markers
```

#### AC-B04 · Click saved place item (T05)

```gherkin
Given  người dùng click vào một item trong Saved Places list
Then   bản đồ fly-to địa điểm đó
And    Place Detail mở (xem place-detail.md §B03)
```

#### AC-B05 · Quản lý danh sách tùy chỉnh (T06)

```gherkin
Given  người dùng click [+ Tạo danh sách mới] trong list picker
Then   input để nhập tên danh sách xuất hiện
And    submit → tạo danh sách mới và địa điểm được thêm vào ngay

Given  người dùng long-press / right-click một danh sách tùy chỉnh
Then   menu: [Đổi tên] [Xóa danh sách]
And    xóa danh sách → xác nhận trước khi xóa
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
