# Administrative Boundaries

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

Administrative Boundaries (Ranh giới hành chính) là tính năng cho phép người dùng xem và chọn các cấp đơn vị hành chính (**Tỉnh/Thành phố**, **Phường/Xã**) thông qua một Dropdown menu hoặc tìm kiếm từ khóa trực tiếp trên thanh Search Bar. Khi một đơn vị hành chính được chọn, bản đồ sẽ tự động di chuyển (pan/zoom) và làm nổi bật ranh giới (polygon) của khu vực đó.

### Why

- **User need:** Người dùng cần tra cứu vị trí địa lý, phạm vi của một tỉnh thành hoặc xã phường cụ thể một cách nhanh chóng, thay vì phải tìm kiếm thủ công qua text.
- **Business value:** Phục vụ tốt cho các nghiệp vụ như quy hoạch, bất động sản, đối tác chính phủ, logistics khi cần nắm rõ phạm vi ranh giới các cấp một cách rõ ràng theo cấu trúc 2 cấp hiện đại.
- **Data sovereignty:** Nguồn dữ liệu ranh giới hành chính chuẩn xác từ các cơ quan có thẩm quyền và được quản lý trên nền tảng GTEL.

## Unique Selling Propositions (USP)

> Đối thủ: Tham khảo & Đánh giá

Tính năng cần thiết khi có nhu cầu tra cứu, thống kê vị trí địa lý, phạm vi của một tỉnh hoặc phường/xã cụ thể một cách nhanh chóng, người dùng đang muốn duyệt qua các đơn vị hành chính thay vì phải tìm kiếm thủ công qua text.

- GTEL Maps: Tra cứu + thống kê qua **Badge** + tích hợp search bar + **Breadcrumb 2 cấp** (Tỉnh/Thành > Phường/Xã) nổi bật.
- [Google Maps](https://www.google.com/maps): Không có tính năng này, tích hợp trên thanh search bar | ranh giới độ chính xác thấp
- [Bản đồ TP.HCM](https://bando.tphcm.gov.vn): Rất tốt + không tích hợp search bar
- [Map4D](https://map.map4d.vn): Tích hợp trên search bar, có dropdown menu, không hỗ trợ cấp xã | ranh giới sai so với satnhap.bando.com.vn (P. Cầu Ông Lãnh) + độ chính xác thấp
- Không đơn vị nào khác tại Việt Nam có tính năng tương tự, chỉ dừng ở mức Category chip.

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                       | Nền tảng    | Input         | AC  |
| --- | ------------------------------------------------------------- | ----------- | ------------- | --- |
| T01 | Click vào nút/icon "Ranh giới hành chính" hoặc thanh Dropdown | Web, Mobile | N/A           | B01 |
| T02 | Chọn một "Tỉnh/Thành phố" từ Dropdown                         | Web, Mobile | `province_id` | B02 |
| T03 | Chọn tiếp một "Phường/Xã" từ Dropdown                         | Web, Mobile | `ward_id`     | B03 |
| T04 | Click [X] để xóa khu vực đang chọn                            | Web, Mobile | N/A           | B04 |
| T05 | Nhập tên đơn vị hành chính vào thanh Search Bar               | Web, Mobile | `keyword`     | B05 |

### States Inventory

| State     | Mô tả                                       | Component                             |
| --------- | ------------------------------------------- | ------------------------------------- |
| `idle`    | Chưa mở/chưa chọn đơn vị nào                | Nút Dropdown bình thường              |
| `loading` | Đang tải danh sách hoặc ranh giới (polygon) | Loader/Skeleton trong menu            |
| `success` | Đã load và hiển thị ranh giới trên bản đồ   | Polygon ranh giới sáng lên + Zoom fit |
| `error`   | Lỗi kết nối hoặc không tìm thấy dữ liệu     | Toast hiển thị lỗi                    |

### Components, Responsive & Typography

#### Component Inventory

| Component                       | Dùng trong    |
| ------------------------------- | ------------- |
| Admin Boundary Dropdown Menu    | T01–T04       |
| Polygon Feature (Ranh giới)     | Success state |
| Clear Button (Nút xóa lựa chọn) | T04           |
| Autocomplete Dropdown item      | T05           |
| Administrative Polygon Icon     | T05           |
| Breadcrumb (2 cấp hành chính)   | Success state |
| Badge Thống kê số lượng         | T01–T03       |

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Mở Dropdown chọn ranh giới (T01)

```gherkin
Given  ứng dụng Maps Viewer đang mở
When   người dùng click vào Dropdown "Ranh giới hành chính"
Then   hiển thị danh sách các "Tỉnh/Thành phố" (Cấp 1)
And    các "Thành phố trực thuộc trung ương" được sắp xếp ưu tiên hiển thị lên đầu danh sách
And    các tỉnh thành còn lại được sắp xếp theo thứ tự bảng chữ cái (Alphabet)
And    mỗi mục trong danh sách có kèm theo **Badge thống kê số lượng** đơn vị hành chính trực thuộc (VD: Hà Nội - hiển thị Badge [579 Phường/Xã])
```

#### AC-B02 · Chọn Tỉnh/Thành phố (T02)

```gherkin
Given  Dropdown đang mở ở danh sách Cấp 1
When   người dùng chọn một "Tỉnh/Thành phố"
Then   bản đồ tự động zoom và pan đến khu vực Tỉnh/Thành phố đó
And    hiển thị polygon chứa ranh giới khu vực được chọn với màu sắc nổi bật
And    hiển thị **Breadcrumb 2 cấp** định vị (VD: "Hà Nội")
And    hiển thị danh sách "Phường/Xã" (Cấp 2) thuộc Tỉnh/Thành phố đó trên Dropdown
```

#### AC-B03 · Chọn Phường/Xã (T03)

```gherkin
Given  đang chọn một Tỉnh/Thành phố (danh sách Cấp 2 hiển thị)
When   người dùng chọn tiếp một "Phường/Xã"
Then   bản đồ tiếp tục tự động zoom fit vừa vặn với ranh giới của đơn vị hành chính đó
And    cập nhật polygon hiển thị rõ nét ranh giới khu vực mới
And    cập nhật và hiển thị **Breadcrumb 2 cấp** định vị (VD: "Hà Nội > Phường Hàng Bài")
```

#### AC-B04 · Xóa lựa chọn (T04)

```gherkin
Given  bản đồ đang hiển thị ranh giới một đơn vị hành chính
When   người dùng click nút xóa (X) trên thanh chọn hoặc breadcrumb
Then   polygon ranh giới biến mất
And    Dropdown trở về trạng thái rỗng (hoặc cấp cao nhất)
And    Breadcrumb bị ẩn đi
And    bản đồ giữ nguyên vị trí hoặc trở về mức zoom mặc định (tuỳ spec chi tiết)
```

#### AC-B05 · Tìm kiếm ranh giới hành chính qua Search Bar (T05)

```gherkin
Given  người dùng đang nhập từ khóa trên thanh Search Bar
When   có kết quả trả về là một đơn vị hành chính (Tỉnh, Xã)
Then   trong danh sách Autocomplete hiển thị kết quả đó với một icon đặc trưng (Polygon icon) để nhận biết đây là kết quả ranh giới khu vực
And    kết quả kèm theo **Breadcrumb 2 cấp** thu gọn
When   người dùng chọn kết quả này
Then   bản đồ tự động zoom fit và hiển thị polygon ranh giới của đơn vị hành chính đó
And    Dropdown ranh giới (nếu đang bật) sẽ tự động đồng bộ (sync) theo khu vực vừa chọn và hiển thị đầy đủ Breadcrumb tương ứng
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
