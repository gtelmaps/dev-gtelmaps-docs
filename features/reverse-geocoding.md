# Reverse Geocoding

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

Reverse Geocoding chuyển đổi tọa độ địa lý (lat, lng) thành địa chỉ hoặc mô tả vị trí tương ứng, trả về kết quả có cấu trúc theo hệ thống hành chính Việt Nam.

### Why

- **User need:** Người dùng cần xác định địa chỉ thực tế từ một vị trí trên bản đồ mà không phải tự đọc tọa độ thô.
- **Business value:** Là API nền tảng dùng bởi Maps Viewer, Fleet Management, Logistics và khách hàng enterprise; dự kiến có call volume lớn.
- **Data sovereignty:** Dữ liệu địa chỉ được lưu trữ và xử lý trên hạ tầng GTEL.

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                          | Nền tảng    | Input                             | AC  |
| --- | ------------------------------------------------ | ----------- | --------------------------------- | --- |
| T01 | Click / tap trên bản đồ                          | Web         | `(lat, lng)` từ map click event   | B01 |
| T02 | Right-click → Context Menu → "Đây là đâu?"       | Web         | `(lat, lng)` từ right-click event | B08 |
| T03 | Long press ≥ 500ms                               | Mobile      | `(lat, lng)` từ long press event  | B09 |
| T04 | Nhập tọa độ vào Search Bar                       | Web, Mobile | String `"lat, lng"`               | B11 |
| T05 | Deep link URL /search/[lat,lon]/@lat,lon,zoom... | Web         | URL query params                  | B12 |
| T06 | Vị trí GPS hiện tại → thông tin địa chỉ hiện tại | Mobile, Web | `(lat, lng)` từ Geolocation API   | B13 |

### States Inventory

| State          | Mô tả             | Component                      |
| -------------- | ----------------- | ------------------------------ |
| `idle`         | Chưa có tương tác | Không có popup / marker        |
| `loading`      | Chờ API response  | Popup / Bottom sheet + spinner |
| `success`      | Có địa chỉ        | Popup / Bottom sheet + actions |
| `zero_results` | Không tìm thấy    | Empty message                  |
| `error`        | API lỗi / timeout | Toast                          |

### Components, Responsive & Typography

#### Component Inventory

| Component                         | Dùng trong      |
| --------------------------------- | --------------- |
| Map Marker / ripple loading       | T01–T06         |
| Popup card                        | Web / tablet    |
| Bottom Sheet                      | Mobile portrait |
| Toast notification                | Error states    |
| Loading spinner (input coord)     | Loading state   |
| Context Menu item → "Đây là đâu?" | T02             |
| Search Bar coord input            | T04             |

#### Responsive Behavior

| Breakpoint                 | Popup style           | Action buttons |
| -------------------------- | --------------------- | -------------- |
| Mobile portrait (< 768px)  | Bottom Sheet          | Stack dọc      |
| Mobile landscape (< 768px) | Popup nhỏ cạnh marker | Inline ngang   |
| Tablet (768px–1024px)      | Popup tiêu chuẩn      | Inline ngang   |
| Desktop (> 1024px)         | Popup tiêu chuẩn      | Inline ngang   |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Click / tap (T01)

```gherkin
Given  người dùng đang xem bản đồ
When   click / tap vào vị trí bất kỳ (không chọn lên icon marker)
Then   hệ thống lấy lat/lng và gọi Reverse Geocoding API tự động
And    di chuyển bản đồ, popup biến mất
```

#### AC-B02 · Marker

```gherkin
Given  bất kỳ trigger nào kích hoạt
When   hệ thống nhận tọa độ
Then   ripple loading marker
And    marker xuất hiện trong 100ms
And    chỉ 1 marker tồn tại
```

#### AC-B03 · Loading state (input coordinate)

```gherkin
Given  API call đang thực hiện
When   chờ > 150ms
Then   loading indicator hiển thị tại popup / bottom sheet
And    text: "Đang tìm địa chỉ..."
```

> **Lý do 150ms:** NFR yêu cầu P95 ≤ 300ms (cache MISS). Với threshold 300ms, đa số request resolve trước khi loading hiển thị, gây flash UI.

#### AC-B04 · Success state

```gherkin
Given  API trả về thành công
Then   popup / bottom sheet hiển thị
And    có nút pin để khi di chuyển marker không tắt popup
```

#### AC-B05 · ZERO_RESULTS state

```gherkin
Given  API trả về không có kết quả
Then   hiển thị "Không tìm thấy địa chỉ cho vị trí này"
And    marker vẫn còn trên bản đồ
```

#### AC-B06 · Error state

```gherkin
Given  API lỗi 4xx/5xx hoặc timeout sau 5 giây (theo COMMON §5)
Then   toast "Không thể tải địa chỉ, vui lòng thử lại"
And    toast tự đóng sau 4 giây
```

#### AC-B08 · Context Menu (T02)

```gherkin
Given  người dùng right-click bản đồ
Then   Context Menu có mục "Đây là đâu?"

Given  chọn "Đây là đâu?"
Then   Context Menu đóng ngay + marker đặt + API được gọi

Given  di chuyển bản đồ
Then   tắt Context menu
```

#### AC-B09 · Long Press Mobile (T03)

```gherkin
Given  giữ ngón tay ≥ 500ms
Then   haptic feedback + marker + bottom sheet mở

Given  nhấc ngón tay trước 500ms
Then   không kích hoạt
```

#### AC-B11 · Search Bar (T05)

```gherkin
Given  nhập "10.7769, 106.7009" vào Search Bar + Enter
Then   hệ thống nhận diện tọa độ decimal degrees, di chuyển bản đồ và gọi API

Given  nhập sai format hoặc ngoài range
Then   hiển thị thông báo GTEL Maps can't find ...
And    không gọi API
```

#### AC-B12 · Deep Link URL (T06)

```gherkin
Given  URL https://{domain}/search/lat,lon/@....
Then   Redirect về https://{domain}/place/lat,lon/@...
And    Hiển thị panel thông tin chi tiết

Given  URL https://{domain}/search/lat,lon/@.... (lat, lon sai)
Then   Hiển thị panel không tìm thấy

Given  Nhập search bar lat,lon
Then   Sai lat,lon thì thông báo lỗi, đúng thì hiển thị panel chi tiết
```

#### AC-B13 · Deep Link URL (T07)

```gherkin
Given  Vị trí GPS hiện tại + Click search bar
Then   thông tin địa chỉ hoặc quận
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
