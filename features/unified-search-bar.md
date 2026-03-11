# Unified Search Bar

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

Unified Search Bar là thanh tìm kiếm chính trên GTEL Maps Viewer, tích hợp nhiều nguồn dữ liệu vào một điểm nhập duy nhất. Hỗ trợ các loại tìm kiếm:

| #   | Loại tìm kiếm               | Ví dụ                                                                                  |
| --- | --------------------------- | -------------------------------------------------------------------------------------- |
| 1   | Địa chỉ đầy đủ (mới + cũ)   | `136 Nguyễn Huệ, Phường Bến Nghé, TP.HCM` (bỏ cấp quận huyện theo địa chỉ mới)         |
| 2   | Tên đường                   | `Nguyễn Huệ`                                                                           |
| 3   | Tìm lân cận                 | `quán ăn gần đây`, `ATM gần tôi`                                                       |
| 4   | Địa điểm / POI              | `Bitexco`, `Bưu điện thành phố`                                                        |
| 5   | Tọa độ WGS84 (nhiều format) | DD: `10.7769, 106.7009` · DMS: `10°46'36.8"N 106°42'3.2"E` · UTM: `48P 694251 1191866` |
| 6   | Tọa độ VN2000               | `VN2000: 1191866, 694251`                                                              |
| 7   | Plus Code                   | `7P28QPG4+4R`                                                                          |
| 8   | Postal Code                 | `700000`                                                                               |
| 9   | Giao lộ                     | `Nguyễn Huệ × Lê Lợi`                                                                  |
| 10  | Công trình xây dựng         | Entrance / cổng, cầu, hầm: `Cầu Thủ Thiêm`, `Hầm Thủ Thiêm`                            |
| 11  | Ranh giới hành chính        | Quốc gia, tỉnh/thành, quận/huyện, phường/xã: `Quận 1`, `Tỉnh Bình Dương`               |
| 12  | Thương hiệu                 | `Highlands Coffee`, `Circle K`                                                         |
| 13  | Chỉ đường                   | `từ Bitexco đến sân bay Tân Sơn Nhất`                                                  |

### Why

- **User need:** Người dùng cần một nơi duy nhất để tìm kiếm bất kỳ thông tin gì trên bản đồ — địa chỉ, địa danh, tọa độ, doanh nghiệp, chỉ đường — mà không phải phân biệt loại query.
- **Business value:** Search Bar là entry point chính cho mọi tương tác trên Maps Viewer; conversion rate và engagement phụ thuộc trực tiếp vào chất lượng tìm kiếm.
- **Platform integration:** Kết nối và orchestrate nhiều API backend (Forward Geocoding, Reverse Geocoding, Place Search, Autocomplete, Directions) thành một trải nghiệm liền mạch.

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**

- **Search Bar:** Gõ text, nhập tọa độ, voice search.
- **Category Chips:** Gợi ý nhanh theo danh mục (Ăn uống, ATM...).
- **Recent Searches:** Lịch sử tìm kiếm gần đây.
- **Deep Link URL:** Truy cập qua URL `/search/{query}`.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                             | Nền tảng    | Input                 | AC  |
| --- | ----------------------------------- | ----------- | --------------------- | --- |
| T01 | Focus Search Bar (click / tap)      | Web, Mobile | —                     | B07 |
| T02 | Gõ text ≥ 2 ký tự                   | Web, Mobile | Partial text          | B01 |
| T03 | Chọn suggestion trong dropdown      | Web, Mobile | `place_id`            | B04 |
| T04 | Nhấn Enter (full text search)       | Web, Mobile | Full query string     | B14 |
| T05 | Nhập tọa độ WGS84 (DD/DMS/UTM)      | Web, Mobile | `lat,lng` / DMS / UTM | B06 |
| T06 | Nhập tọa độ VN2000                  | Web, Mobile | `VN2000: x, y`        | B16 |
| T07 | Nhập Plus Code                      | Web, Mobile | `7P28QPG4+4R`         | B17 |
| T08 | Nhập Postal Code                    | Web, Mobile | `700000`              | B18 |
| T09 | Nhấn category chip                  | Web, Mobile | Category type         | B09 |
| T10 | Nhấn microphone icon (voice search) | Web, Mobile | Speech transcript     | B10 |
| T11 | Deep link URL `/search/{query}`     | Web, Mobile | URL params            | B11 |
| T12 | Nhấn recent search item             | Web, Mobile | Stored query          | B07 |
| T13 | Nhấn clear (X) button               | Web, Mobile | —                     | B15 |
| T14 | Nhập query chỉ đường                | Web, Mobile | `từ A đến B`          | B19 |

### States Inventory

| State               | Mô tả                        | Component                        |
| ------------------- | ---------------------------- | -------------------------------- |
| `idle`              | Thanh search chưa được focus | Search Bar collapsed (mobile)    |
| `focused_empty`     | Focused, chưa gõ gì          | Recent searches + category chips |
| `typing`            | Đang gõ, chờ debounce        | Input text, cancel old request   |
| `loading_suggest`   | Debounce xong, chờ API       | Dropdown skeleton / spinner      |
| `suggestions`       | Có predictions               | Dropdown list                    |
| `zero_suggest`      | Autocomplete trả rỗng        | "Không tìm thấy gợi ý"           |
| `loading_results`   | Đang gọi Forward Geocoding   | Results panel skeleton           |
| `results`           | Có kết quả tìm kiếm          | Results panel + markers          |
| `zero_results`      | Forward Geocoding trả rỗng   | "Không tìm thấy kết quả"         |
| `error`             | API lỗi / timeout            | Toast lỗi                        |
| `voice_listening`   | Đang nghe giọng nói          | Microphone active + animation    |
| `coordinate_detect` | Input là tọa độ hợp lệ       | Trigger reverse geocoding flow   |

### Components, Responsive & Typography

#### Component Inventory

| Component               | Dùng trong    |
| ----------------------- | ------------- |
| Search Bar input        | All triggers  |
| Clear (X) button        | Khi có input  |
| Microphone icon         | Voice search  |
| Autocomplete dropdown   | T02           |
| Suggestion item         | Dropdown      |
| Recent search item      | T01 focus     |
| Category chip           | T01, T06      |
| Results panel (sidebar) | Desktop       |
| Results panel (sheet)   | Mobile        |
| Result list item        | Results panel |
| Voice listening overlay | T07           |

#### Responsive Behavior

| Breakpoint                 | Search Bar             | Autocomplete dropdown | Results panel       |
| -------------------------- | ---------------------- | --------------------- | ------------------- |
| Mobile portrait (< 768px)  | Full width, expandable | Full width overlay    | Bottom sheet        |
| Mobile landscape (< 768px) | Full width             | Full width            | Bottom sheet        |
| Tablet (768px–1024px)      | Fixed width ~400px     | Below search bar      | Left sidebar ~400px |
| Desktop (> 1024px)         | Fixed width ~400px     | Below search bar      | Left sidebar ~400px |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Autocomplete trigger (T02)

```gherkin
Given  user focus Search Bar và gõ ≥ 2 ký tự
When   ngừng gõ 300ms (debounce)
Then   gọi Autocomplete API với input hiện tại
And    hiển thị dropdown với predictions

Given  user gõ 1 ký tự
Then   không gọi API, giữ nguyên empty state hoặc recent searches
```

#### AC-B02 · Autocomplete dropdown rendering

```gherkin
Given  Autocomplete API trả về predictions
Then   dropdown hiển thị tối đa 5 items
And    mỗi item có icon + main_text (bold matched substring) + secondary_text
And    dropdown xuất hiện trong ≤ 100ms sau khi nhận response
```

#### AC-B03 · Autocomplete keyboard navigation

```gherkin
Given  dropdown đang hiển thị
When   nhấn Arrow Down / Arrow Up
Then   highlight di chuyển qua các suggestion items

When   nhấn Escape
Then   dropdown đóng, focus quay lại input, text giữ nguyên
```

#### AC-B04 · Select suggestion (T03)

```gherkin
Given  dropdown hiển thị, user click / tap suggestion
Then   text trong Search Bar thay bằng description của suggestion
And    dropdown đóng
And    gọi Forward Geocoding / Place Detail với place_id
And    lưu query vào recent searches
```

#### AC-B05 · Map response to selection

```gherkin
Given  Forward Geocoding trả về kết quả với geometry
Then   bản đồ flyTo vị trí kết quả
And    nếu có viewport → fitBounds(viewport)
And    nếu chỉ có location → flyTo(location, zoom=17)
And    marker xuất hiện tại vị trí
```

#### AC-B06 · Coordinate input WGS84 (T05)

```gherkin
Given  user nhập tọa độ WGS84 dạng DD: "10.7769, 106.7009"
When   pattern match detected
Then   bypass autocomplete → gọi Reverse Geocoding API
And    bản đồ flyTo tọa độ + marker + popup địa chỉ

Given  user nhập tọa độ WGS84 dạng DMS: "10°46'36.8\"N 106°42'3.2\"E"
Then   parse → convert sang DD → gọi Reverse Geocoding API

Given  user nhập tọa độ WGS84 dạng UTM: "48P 694251 1191866"
Then   parse → convert sang DD → gọi Reverse Geocoding API

Given  nhập tọa độ ngoài range hợp lệ
Then   xử lý như text search bình thường (Forward Geocoding)
```

#### AC-B07 · Focus empty state (T01)

```gherkin
Given  user click / focus vào Search Bar, input rỗng
Then   hiển thị recent searches (nếu có) + category chips

Given  không có recent searches
Then   chỉ hiển thị category chips
```

#### AC-B08 · Recent searches management

```gherkin
Given  recent searches đang hiển thị
When   user nhấn icon X trên một item
Then   item đó bị xóa khỏi danh sách và localStorage

When   user nhấn "Xóa tất cả"
Then   tất cả recent searches bị xóa

When   user nhấn vào một recent search item
Then   text fill vào Search Bar + trigger Forward Geocoding
```

#### AC-B09 · Category chips (T06)

```gherkin
Given  user nhấn chip "Ăn uống"
When   chip activated
Then   gọi Forward Geocoding / Place Search với types=restaurant,cafe
And    location bias = map center hiện tại
And    Results panel hiển thị kết quả
```

#### AC-B10 · Voice search (T07)

```gherkin
Given  user nhấn microphone icon
When   browser hỗ trợ Web Speech API
Then   overlay "Đang nghe..." hiển thị + mic animation

Given  10 giây không có speech
Then   tự đóng, thông báo "Không nhận được giọng nói"

Given  browser không hỗ trợ Web Speech API
Then   microphone icon bị ẩn
```

#### AC-B11 · Deep link search (T08)

```gherkin
Given  URL https://{domain}/search/Nguyen+Hue/@10.77,106.70,15
Then   Search Bar fill text "Nguyen Hue"
And    bản đồ center tại 10.77, 106.70, zoom 15
And    auto trigger Forward Geocoding

Given  URL https://{domain}/search/ (query rỗng)
Then   hiển thị empty state, không gọi API
```

#### AC-B12 · Results panel rendering

```gherkin
Given  Forward Geocoding trả về ≥ 1 results
Then   Results panel mở (sidebar desktop / bottom sheet mobile)
And    mỗi result item hiển thị: tên, địa chỉ, khoảng cách (nếu có)
And    markers đánh số tương ứng trên bản đồ
```

#### AC-B13 · Results panel interaction

```gherkin
Given  Results panel đang hiển thị
When   user click / tap một result item
Then   bản đồ flyTo vị trí đó

When   user hover một result item (desktop)
Then   marker tương ứng highlight trên bản đồ
```

#### AC-B20 · Explore nearby điểm đến

```gherkin
Given  Results panel đang hiển thị với một điểm đến đã chọn
When   user nhấn "Explore nearby"
Then   gọi Nearby Search xung quanh điểm đến đó
And    hiển thị danh sách POI lân cận (ăn uống, khách sạn, ATM...)
And    markers lân cận xuất hiện trên bản đồ
```

#### AC-B21 · Explore new places along this route

```gherkin
Given  Results panel đang hiển thị kết quả chỉ đường (route đã có)
When   user nhấn "Explore new places along this route"
Then   gọi Search Along Route với route geometry
And    hiển thị danh sách POI dọc theo tuyến đường
And    markers xuất hiện dọc route trên bản đồ
```

#### AC-B14 · Enter full text search (T04)

```gherkin
Given  user nhấn Enter khi input rỗng
Then   không làm gì

Given  user nhấn Enter với query text bất kỳ
When   không có suggestion được highlight
Then   hệ thống detect loại query và gọi API phù hợp
And    hiển thị kết quả trong Results panel
```

**Các loại query hỗ trợ khi Enter:**

| Query                              | Detect thành         | Hành vi                                     |
| ---------------------------------- | -------------------- | ------------------------------------------- |
| `136 Nguyễn Huệ, Bến Nghé, TP.HCM` | Địa chỉ (mới/cũ)     | Forward Geocoding                           |
| `Nguyễn Huệ`                       | Tên đường            | Forward Geocoding → results list            |
| `quán ăn gần đây`                  | Lân cận              | Nearby Search + location bias = user/map    |
| `Bitexco`                          | Địa điểm / POI       | Forward Geocoding / Place Search            |
| `Nguyễn Huệ × Lê Lợi`              | Giao lộ              | Intersection Search                         |
| `Cầu Thủ Thiêm`                    | Công trình xây dựng  | Forward Geocoding (type=infrastructure)     |
| `Quận 1`                           | Ranh giới hành chính | Admin Boundary Search → fitBounds           |
| `Highlands Coffee`                 | Thương hiệu          | Brand/Chain Search → results list + markers |

#### AC-B15 · Clear button (T13)

```gherkin
Given  Search Bar có text
When   nhấn clear (X) button
Then   text bị xóa
And    dropdown đóng, results panel đóng
And    markers kết quả bị xóa
And    hiển thị empty state (recent + chips)
And    focus vẫn ở Search Bar
```

#### AC-B16 · Tọa độ VN2000 (T06)

```gherkin
Given  user nhập "VN2000: 1191866, 694251"
When   pattern match VN2000 detected
Then   convert VN2000 → WGS84 DD
And    gọi Reverse Geocoding API
And    bản đồ flyTo + marker + popup địa chỉ
```

#### AC-B17 · Plus Code (T07)

```gherkin
Given  user nhập "7P28QPG4+4R"
When   pattern match Plus Code detected (có dấu +)
Then   decode Plus Code → lat, lng
And    gọi Reverse Geocoding API
And    bản đồ flyTo + marker + popup địa chỉ

Given  Plus Code không hợp lệ
Then   xử lý như text search bình thường
```

#### AC-B18 · Postal Code (T08)

```gherkin
Given  user nhập "700000"
When   pattern match Postal Code detected (6 chữ số, thuộc danh sách postal code VN)
Then   gọi Forward Geocoding / lookup postal → trả về khu vực tương ứng
And    bản đồ fitBounds ranh giới khu vực
```

#### AC-B19 · Chỉ đường (T14)

```gherkin
Given  user nhập "từ Bitexco đến sân bay Tân Sơn Nhất"
When   pattern match chỉ đường detected (chứa "từ...đến", "to...from"...)
Then   parse origin + destination
And    chuyển sang Directions mode
And    auto fill origin / destination vào Directions panel
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
