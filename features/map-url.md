# Map URL

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

Map URL định nghĩa cấu trúc URL của Maps Viewer — cách encode trạng thái bản đồ (vị trí, zoom, tilt, heading, layer, panel đang mở) vào URL, và cách parse URL để restore trạng thái tương ứng. URL là cơ chế chia sẻ, bookmark và deep link chính của ứng dụng.

### Why

- **User need:** Người dùng muốn bookmark hoặc chia sẻ link bản đồ và khi mở lại, bản đồ phải hiển thị đúng vị trí, zoom, và trạng thái như lúc chia sẻ.
- **Business value:** URL shareable là kênh viral growth quan trọng; deep link URL là yêu cầu tích hợp bắt buộc của các đối tác enterprise.

---

# Part 02 - Specifications

## Backend

_Không có backend riêng. URL được xử lý client-side bởi SPA router._

## UI/UX & Frontend

### URL Structure

```
https://{domain}/{locale}/maps/@{lat},{lng},{zoom}[z|m][,{tilt}a[,{heading}y]]
  [?layer={layer_id}]
  [?data={encoded_data}]
  [?ttu={transit_url}]
  [?s={search_query}]
  [?ml={my_location_lat,lng}]
  [&yt={youtube_place_id}]

Examples:
/@10.7769,106.7009,14z
/@10.7769,106.7009,14z,45a
/@10.7769,106.7009,14z,45a,180y
/place/ChIJ... (Place Detail deep link)
/search/coffee/@10.7769,106.7009,14z
/directions/@10.7769,106.7009,14z?origin=...&dest=...
```

### URL Parameter Reference

| Param       | Format              | Mô tả                                | Ví dụ               |
| ----------- | ------------------- | ------------------------------------ | ------------------- |
| `lat`       | decimal degrees     | Vĩ độ tâm bản đồ                    | `10.7769`           |
| `lng`       | decimal degrees     | Kinh độ tâm bản đồ                  | `106.7009`          |
| `zoom`      | number + `z`        | Zoom level (0–22)                   | `14z`               |
| `tilt`      | number + `a`        | Góc nghiêng 3D (0–60)               | `45a`               |
| `heading`   | number + `y`        | Hướng la bàn (0–360)                | `180y`              |
| `layer`     | string              | Layer đang active                   | `layer=satellite`   |
| `data`      | base64 encoded      | Custom data (pins, paths, polygons) | `data=...`          |
| `s`         | string              | Search query khởi tạo               | `s=coffee`          |
| `ml`        | `lat,lng`           | My Location override                | `ml=10.78,106.70`   |

### Triggers & Entry Points

| ID  | Trigger                                          | Input                              | AC  |
| --- | ------------------------------------------------ | ---------------------------------- | --- |
| T01 | Bản đồ pan / zoom / tilt / rotate                | New map state                      | B01 |
| T02 | Place Detail mở (`/place/[id]`)                 | `place_id`                         | B02 |
| T03 | Search Results mở (`/search/[query]`)            | query + coordinates                | B03 |
| T04 | Directions mở (`/directions/`)                   | origin, destination, mode          | B04 |
| T05 | Người dùng load URL từ bookmark / share          | Full URL                           | B05 |
| T06 | Browser Back / Forward navigation               | History stack                      | B06 |

### States Inventory

| State       | Mô tả                               |
| ----------- | ----------------------------------- |
| `syncing`   | URL đang được cập nhật sau state change |
| `restoring` | URL đang được parse để restore state |

### Acceptance Criteria

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · URL cập nhật khi bản đồ thay đổi (T01)

```gherkin
Given  người dùng pan / zoom / tilt / rotate bản đồ
Then   URL cập nhật realtime theo định dạng /@lat,lng,zooma
And    URL update được debounce 300ms (không cập nhật từng pixel)
And    History.pushState được dùng (không reload trang)
And    lat/lng làm tròn 4 chữ số thập phân
And    tilt và heading chỉ xuất hiện trong URL khi ≠ 0
```

#### AC-B02 · Place Detail URL (T02)

```gherkin
Given  Place Detail Panel mở
Then   URL chuyển thành /place/[place_id]
And    coordinate params (@lat,lng,zoom) vẫn được giữ nguyên sau place_id
And    đóng Place Detail → URL restore về /@lat,lng,zoom
```

#### AC-B03 · Search URL (T03)

```gherkin
Given  người dùng thực hiện search với query "phở bò"
Then   URL chuyển thành /search/ph%E1%BB%9F+b%C3%B2/@lat,lng,zoom
And    encoding tuân thủ RFC 3986 (encodeURIComponent)
```

#### AC-B04 · Directions URL (T04)

```gherkin
Given  người dùng mở Directions với origin và destination
Then   URL encode đầy đủ: /directions/Origin+Name/@lat,lng,zoom?origin=...&dest=...&mode=driving
```

#### AC-B05 · Parse và restore từ URL (T05)

```gherkin
Given  người dùng mở URL /@10.7769,106.7009,14z,45a,90y?layer=satellite
Then   bản đồ restore đúng: center (10.7769, 106.7009), zoom 14, tilt 45°, heading 90°, layer satellite

Given  URL có params không hợp lệ hoặc thiếu
Then   bản đồ load với giá trị mặc định cho params bị lỗi
And    URL được chuẩn hóa lại (redirect về URL hợp lệ)
```

#### AC-B06 · Browser history navigation (T06)

```gherkin
Given  người dùng thực hiện nhiều thao tác (pan, open place, search...)
Then   mỗi action tạo một history entry

Given  người dùng nhấn Back
Then   bản đồ restore trạng thái của history entry trước đó
And    nhấn Forward → restore lại trạng thái sau
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
