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

| ID  | Trigger                                          | Nền tảng    | Input                             | AC riêng |
| --- | ------------------------------------------------ | ----------- | --------------------------------- | -------- |
| T01 | Click / tap trên bản đồ                          | Web         | `(lat, lng)` từ map click event   | B01      |
| T02 | Right-click → Context Menu → "Đây là đâu?"       | Web         | `(lat, lng)` từ right-click event | B08      |
| T03 | Long press ≥ 500ms                               | Mobile      | `(lat, lng)` từ long press event  | B09      |
| T04 | Nhập tọa độ vào Search Bar                       | Web, Mobile | String `"lat, lng"`               | B11      |
| T05 | Deep link URL /place/[lat,lon]/@lat,lon,zoom...  | Web         | URL query params                  | B12      |
| T06 | Vị trí GPS hiện tại → thông tin địa chỉ hiện tại | Mobile, Web | `(lat, lng)` từ Geolocation API   | B13      |

### States Inventory

### Components, Responsive & Typography

#### Component Inventory

Popup

- skeleton loading
- pin

#### Responsive Behavior

#### Typography

### Acceptance Criteria — UI

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
