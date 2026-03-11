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

## Unique Selling Propositions (USP)

| #   | USP                        | Mô tả                                                                                             | So sánh với Google Maps                      |
| --- | -------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| 1   | URL human-readable         | Cấu trúc `/@lat,lng,zoom` dễ đọc, dễ chỉnh tay trực tiếp trên address bar                         | Tương đương                                  |
| 2   | Full state restore         | URL encode đầy đủ: vị trí, zoom, tilt, heading, panel đang mở, search query, directions params    | Tương đương                                  |
| 3   | 3D state shareable         | Tilt + heading 3D được encode vào URL, người nhận thấy đúng góc nhìn 3D                           | Google Maps cũng hỗ trợ                      |
| 4   | Deep link cho mọi feature  | Place Detail, Search, Directions đều có URL riêng — bookmark / share / embed được tất cả          | Tương đương                                  |
| 5   | Enterprise-ready deep link | Đối tác có thể tích hợp deep link vào app nội bộ mà không cần SDK, chỉ cần URL                    | Google Maps yêu cầu API key cho embed        |
| 6   | Browser history native     | Back/Forward hoạt động tự nhiên qua mọi thao tác (pan, search, open place, directions)            | Tương đương                                  |
| 7   | Custom data qua URL        | Hỗ trợ param `data=` encode pins, paths, polygons — chia sẻ markup tùy chỉnh mà không cần account | Google Maps không hỗ trợ custom data qua URL |
| 8   | Graceful fallback          | URL thiếu hoặc sai params → load mặc định thay vì lỗi, tự chuẩn hóa lại URL hợp lệ                | Tương đương                                  |

---

# Part 02 - Specifications

## Backend

_Không có backend riêng. URL được xử lý client-side bởi SPA router._

## UI/UX & Frontend

### URL Structure

```
https://{domain}/{locale}/maps/@{lat},{lng},{zoom}z[,{tilt}t[,{heading}h]]
  [?data={encoded_data}]

Examples:
/@10.7769,106.7009,14z
/@10.7769,106.7009,14z,45t
/@10.7769,106.7009,14z,45t,180h
/place/ChIJ... (Place Detail deep link)
/search/coffee/@10.7769,106.7009,14z
/directions/@10.7769,106.7009,14z?origin=...&dest=...
```

### URL Parameter Reference

| Param     | Format                  | Mô tả                               | Ví dụ      |
| --------- | ----------------------- | ----------------------------------- | ---------- |
| `lat`     | decimal degrees         | Vĩ độ tâm bản đồ                    | `10.7769`  |
| `lng`     | decimal degrees         | Kinh độ tâm bản đồ                  | `106.7009` |
| `zoom`    | number + `z`            | Zoom level (0–22)                   | `14z`      |
| `tilt`    | number + `t`            | Góc nghiêng 3D (0–60)               | `45t`      |
| `heading` | number + `h`            | Hướng la bàn (0–360)                | `180h`     |
| `data`    | base64 encoded protobuf | Custom data (pins, paths, polygons) | `data=...` |

### Triggers & Entry Points

| ID  | Trigger                                 | Input                     | AC  |
| --- | --------------------------------------- | ------------------------- | --- |
| T01 | Bản đồ pan / zoom / tilt / rotate       | New map state             | B01 |
| T02 | Place Detail mở (`/place/[id]`)         | `place_id`                | B02 |
| T03 | Search Results mở (`/search/[query]`)   | query + coordinates       | B03 |
| T04 | Directions mở (`/directions/`)          | origin, destination, mode | B04 |
| T05 | Người dùng load URL từ bookmark / share | Full URL                  | B05 |
| T06 | Browser Back / Forward navigation       | History stack             | B06 |

### States Inventory

| State       | Mô tả                                   |
| ----------- | --------------------------------------- |
| `syncing`   | URL đang được cập nhật sau state change |
| `restoring` | URL đang được parse để restore state    |

### Acceptance Criteria

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · URL cập nhật khi bản đồ thay đổi (T01)

```gherkin
Given  người dùng pan / zoom / tilt / rotate bản đồ
Then   URL cập nhật realtime theo định dạng /@{lat},{lng},{zoom}z[,{tilt}t[,{heading}h]]
And    URL update được debounce 300ms (không cập nhật từng pixel)
And    History.pushState được dùng (không reload trang)
And    lat/lng làm tròn tối đa 7 chữ số thập phân, zoom, tilt, heading làm tròn tối đa 2 chữ số thập phân
And    tilt, heading chỉ xuất hiện trên URL khi người dùng có thao tác tilt hoặc thay đổi heading (rotate) bản đồ
```

#### AC-B02 · Place Detail URL (T02)

```gherkin
Given  Place Detail Panel mở
Then   URL chuyển thành /place/[place_id]
And    coordinate params (@{lat},{lng},{zoom}z) vẫn được giữ nguyên sau place_id
And    đóng Place Detail → URL restore về /@{lat},{lng},{zoom}z
```

#### AC-B03 · Search URL (T03)

```gherkin
Given  người dùng thực hiện search với query "phở bò"
Then   URL chuyển thành /search/ph%E1%BB%9F+b%C3%B2/@{lat},{lng},{zoom}z
And    encoding tuân thủ RFC 3986 (encodeURIComponent)
```

#### AC-B04 · Directions URL (T04)

```gherkin
Given  người dùng mở Directions với origin và destination
Then   URL encode đầy đủ: /directions/Origin+Name/@{lat},{lng},{zoom}z?origin=...&dest=...&mode=driving
```

#### AC-B05 · Parse và restore từ URL (T05)

```gherkin
Given  người dùng mở URL /@10.7769,106.7009,14z,45t,90h
Then   bản đồ restore đúng: center (10.7769, 106.7009), zoom 14, tilt 45°, heading 90°

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
