# Context Menu

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

Context Menu là menu ngữ cảnh xuất hiện khi người dùng right-click trên bản đồ web. Menu cung cấp các hành động nhanh liên quan đến vị trí được click: sao chép tọa độ, chia sẻ vị trí, chỉ đường, tra địa chỉ, tìm gần đây, đo lường, và báo cáo dữ liệu.

### Why

- **User need:** Power users trên desktop cần truy cập nhanh các tính năng Maps mà không cần navigate qua nhiều bước UI.
- **Business value:** Context Menu là shortcut để vào các flows có giá trị cao (Directions, Reverse Geocoding, Nearby) — tăng feature discoverability.

## Unique Selling Propositions (USP)

| #  | USP                          | Mô tả                                                                                     | So sánh                                              |
| -- | ---------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Copy tọa độ đa format       | Hỗ trợ copy WGS84 (DD/DMS/UTM), VN2000, Plus Code — phục vụ đa ngành (GIS, xây dựng, logistics) | Google Maps chỉ copy DD                              |
| 2  | Đo lường khoảng cách + diện tích | Đo nhanh từ context menu — không cần mở tool riêng                                        | Google Maps có Measure distance nhưng cần nhiều bước |
| 3  | Báo cáo dữ liệu sai         | User báo lỗi dữ liệu ngay tại vị trí — cải thiện chất lượng data nhanh hơn               | Google Maps cũng có "Suggest an edit"                |
| 4  | Chia sẻ vị trí kèm state    | Share link kèm zoom/tilt/heading — người nhận thấy đúng góc nhìn                          | Google Maps share không giữ tilt/heading             |

---

# Part 02 - Specifications

## Backend

_Không có backend riêng — Context Menu điều phối các feature khác._

## UI/UX & Frontend

### Menu Items

| Group | Item                    | Action                                       | Feature liên quan       |
| ----- | ----------------------- | -------------------------------------------- | ----------------------- |
| Info  | `{Lat}, {Lon}`          | Copy tọa độ vào clipboard                    | —                       |
| —     | Chia sẻ vị trí này      | Copy link / mở Share dialog                  | map-url.md              |
| Nav   | Chỉ đường đến đây       | Mở Directions với destination = vị trí click | directions.md           |
| —     | Chỉ đường từ đây        | Mở Directions với origin = vị trí click      | directions.md           |
| —     | Đây là đâu?             | Reverse Geocoding tại vị trí click           | reverse-geocoding.md    |
| —     | Tìm kiếm gần đây        | Mở Search Nearby tại vị trí click            | search-nearby.md        |
| ——    | _(separator)_           | —                                            | —                       |
| Data  | Thêm địa điểm còn thiếu | Mở form thêm địa điểm                        | (external contribution) |
| —     | Báo cáo sai dữ liệu     | Mở form báo cáo                              | (external feedback)     |
| ——    | _(separator)_           | —                                            | —                       |
| Tools | In bản đồ               | Mở print dialog                              | —                       |
| —     | Đo khoảng cách          | Kích hoạt Measure Distance tool              | (measure.md)            |
| —     | Đo diện tích            | Kích hoạt Measure Area tool                  | (measure.md)            |
| —     | Đo bán kính             | Kích hoạt Measure Radius tool                | (measure.md)            |

### Triggers & Entry Points

| ID  | Trigger                                    | Nền tảng | Input                     | AC  |
| --- | ------------------------------------------ | -------- | ------------------------- | --- |
| T01 | Right-click trên vùng bản đồ trống         | Web      | `(lat, lng)` từ event     | B01 |
| T02 | Right-click trên POI marker                | Web      | `place_id` + `(lat, lng)` | B02 |
| T03 | Click một menu item                        | Web      | Item action               | B03 |
| T04 | Di chuyển bản đồ / click ra ngoài / Escape | Web      | —                         | B04 |

### States Inventory

| State    | Mô tả                         | Component                 |
| -------- | ----------------------------- | ------------------------- |
| `hidden` | Chưa right-click hoặc đã đóng | Menu ẩn                   |
| `open`   | Đang hiển thị                 | Dropdown menu tại con trỏ |

### Components, Responsive & Typography

#### Component Inventory

| Component                    | Dùng trong  |
| ---------------------------- | ----------- |
| Context Menu container       | T01–T04     |
| Menu item row (icon + label) | T03         |
| Separator                    | Giữa groups |
| Coordinate display row       | T01, T02    |

#### Responsive Behavior

_Context Menu chỉ hiển thị trên Web (desktop). Không có trên mobile — thay thế bởi Long Press (xem map-interaction.md §B07)._

| Breakpoint | Position                                                     |
| ---------- | ------------------------------------------------------------ |
| Desktop    | Tại vị trí con trỏ, tự điều chỉnh để không ra ngoài viewport |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Mở Context Menu (T01)

```gherkin
Given  người dùng right-click trên vùng bản đồ trống
Then   Context Menu xuất hiện tại vị trí con trỏ (offset 4px)
And    dòng đầu tiên hiển thị tọa độ (làm tròn 5 chữ số thập phân): "10.77691, 106.70092" (clickable để copy)
And    menu tự điều chỉnh vị trí để không bị cắt bởi viewport edge
And    nếu có Context Menu khác đang mở → đóng và mở menu mới tại vị trí mới
```

#### AC-B02 · Right-click trên POI marker (T02)

```gherkin
Given  người dùng right-click trên POI marker
Then   Context Menu mở với thêm item "Mở [Tên địa điểm]" ở đầu menu
And    item "Đây là đâu?" được thay bằng "Thông tin địa điểm" hoặc ẩn đi
```

#### AC-B03 · Click menu items (T03)

```gherkin
Given  người dùng click "{Lat}, {Lon}" (dòng tọa độ)
Then   tọa độ được copy vào clipboard
And    toast nhỏ: "Đã sao chép tọa độ"
And    menu đóng

Given  người dùng click "Chỉ đường đến đây"
Then   menu đóng, Directions Panel mở với destination = vị trí right-click (xem directions.md §B01)

Given  người dùng click "Đây là đâu?"
Then   menu đóng, Reverse Geocoding kích hoạt tại vị trí đó (xem reverse-geocoding.md §B08)

Given  người dùng click "Chia sẻ vị trí này"
Then   menu đóng, mở Share Dialog (Modal) tại giữa màn hình
And    Dialog hiển thị 2 tab: "Gửi liên kết" và "Nhúng bản đồ"
And    Phần nội dung hiển thị: 
       - Tọa độ (làm tròn 6 chữ số thập phân để đảm bảo độ chính xác cao nhất)
       - Địa chỉ đầy đủ (gọi Reverse Geocoding nếu chưa có - xem reverse-geocoding.md)
       - Input chứa link chia sẻ (rút gọn nếu có) + nút "Sao chép đường liên kết"
       - Các phím tắt chia sẻ nhanh: WhatsApp, X (Twitter), Gmail
And    Click nút đóng (X) hoặc click ra ngoài Modal để đóng dialog

Given  người dùng click "Tìm kiếm gần đây"
Then   menu đóng, Search Nearby mở tại vị trí đó (xem search-nearby.md §B01)
```

#### AC-B04 · Đóng Context Menu (T04)

```gherkin
Given  Context Menu đang mở
When   người dùng click ra ngoài menu hoặc di chuyển bản đồ hoặc nhấn Escape
Then   menu đóng không thực hiện action nào
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
