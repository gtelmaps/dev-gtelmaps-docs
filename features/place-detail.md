# Place Detail

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

Place Detail hiển thị thông tin chi tiết của một sự vật/hiện tượng trên bản đồ (POI, địa chỉ, hoặc đơn vị hành chính). Tính năng này **hỗ trợ multilayout** (hiển thị giao diện thay đổi linh hoạt tùy theo loại đối tượng) nhằm cung cấp nhóm thông tin phù hợp nhất theo ngữ cảnh khi người dùng chọn từ bản đồ, danh sách tìm kiếm, hoặc qua deep link.

### Why

- **User need:** Người dùng cần xem nhanh thông tin đầy đủ của một địa điểm — từ hover để preview nhanh, đến click để xem toàn bộ chi tiết — mà không cần rời khỏi bản đồ.
- **Business value:** Place Detail là trung tâm tương tác của Maps Viewer; tăng session depth, là điểm gắn kết quảng cáo địa điểm (Promoted Places) và là API nền tảng cho Fleet, Logistics, và khách hàng enterprise.
- **Data sovereignty:** Dữ liệu địa điểm được lưu trữ và xử lý trên hạ tầng GTEL.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                        | So sánh                                              |
| -- | -------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Multilayout theo loại đối tượng  | Giao diện thay đổi tùy loại (POI, địa chỉ, đơn vị hành chính) — không dùng layout cố định    | Google Maps cũng có nhưng ít tùy biến cho VN          |
| 2  | Hover preview + click detail     | Hover xem nhanh, click xem đầy đủ — giảm số bước tương tác                                   | Google Maps chỉ có click, không có hover preview      |
| 3  | Promoted Places ready            | Kiến trúc sẵn sàng cho quảng cáo địa điểm (Promoted Places) — monetization native            | Google Maps có Promoted Pins nhưng không mở cho VN    |
| 4  | Dữ liệu POI bản địa VN          | Thông tin POI từ nguồn địa phương: số nhà, ngõ/hẻm, tên thường gọi — chi tiết hơn cho VN     | Google Maps phụ thuộc community contribution          |
| 5  | Self-hosted, không giới hạn call | Không bị rate limit theo pricing tier — phù hợp cho enterprise tích hợp Fleet/Logistics       | Google Places API tính phí theo request               |

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Bản đồ (Map):** Marker của POI đang hiển thị trên bản đồ.
- **Danh sách (List):** Các mục tiêu trong Search Results, Nearby Places, Saved Places, hoặc Recent Searches.
- **URL / Deep Link:** Đường dẫn trực tiếp có chứa `place_id`.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                         | Nền tảng    | Input                        | AC  |
| --- | ----------------------------------------------- | ----------- | ---------------------------- | --- |
| T01 | Hover vào POI marker (tooltip preview)          | Web         | `place_id` từ marker hover   | B01 |
| T02 | Click vào POI marker trên bản đồ                | Web, Mobile | `place_id` từ marker click   | B02 |
| T03 | Click vào item trong danh sách Search Results   | Web, Mobile | `place_id` từ search result  | B03 |
| T04 | Click vào item trong danh sách Nearby Places    | Web, Mobile | `place_id` từ nearby result  | B03 |
| T05 | Click vào item trong danh sách Saved Places     | Web, Mobile | `place_id` từ saved list     | B03 |
| T06 | Click vào item trong danh sách Recent / History | Web, Mobile | `place_id` từ history list   | B03 |
| T07 | Truy cập deep link `/place/[place_id]`          | Web         | `place_id` từ URL path param | B04 |

### States Inventory

| State       | Mô tả                         | Component                              |
| ----------- | ----------------------------- | -------------------------------------- |
| `idle`      | Chưa có tương tác             | Không có tooltip / panel               |
| `tooltip`   | Hover — hiển thị preview nhẹ  | Tooltip card nhỏ cạnh marker           |
| `loading`   | Chờ Place Detail API response | Panel / Bottom sheet + skeleton loader |
| `success`   | Có đầy đủ thông tin địa điểm  | Panel / Bottom sheet + actions         |
| `not_found` | `place_id` không tồn tại      | Empty state panel                      |
| `error`     | API lỗi / timeout             | Toast                                  |

### Multi-layout Structure

Place Detail áp dụng cấu trúc **Multilayout** tùy thuộc vào loại đối tượng mà người dùng đang tương tác:

1. **POI / Address Layout (Địa điểm thông thường / Địa chỉ):**
   - Hiển thị cho các nhà hàng, quán cà phê, vị trí kinh doanh, hoặc một địa chỉ nhà cụ thể.
   - Tập trung vào tương tác cá nhân và doanh nghiệp: Hình ảnh (Gallery), Giờ mở cửa, Số điện thoại, Website, Review/Đánh giá tiện ích.

2. **Administrative Boundary Layout (Đơn vị hành chính):**
   - Hiển thị khi người dùng tìm kiếm hoặc chọn một đơn vị hành chính (Tỉnh/Thành phố, Quận/Huyện, Phường/Xã).
   - Tập trung vào tính chất quản lý và khu vực: 
     - Phân loại cấp hành chính (Thành phố trực thuộc trung ương, Tỉnh, Quận, Huyện, v.v.).
     - Breadcrumb vị trí (Ví dụ: Việt Nam > TP. HCM > Quận 1).
     - Thông tin thống kê chức năng (Diện tích, Dân số, Mật độ dân số) nếu có dữ liệu.
     - Danh sách các khu vực/đơn vị hành chính trực thuộc.
   - Hành vi bản đồ tương ứng: Kết hợp tính năng [Administrative Boundaries](./administrative-boundaries.md) để highlight ranh giới (polygon) của khu vực.

### Components, Responsive & Typography

#### Component Inventory

| Component                                                       | Dùng trong    |
| --------------------------------------------------------------- | ------------- |
| POI Marker (highlighted state)                                  | T01–T06       |
| Tooltip Card (hover preview)                                    | T01           |
| Place Detail Panel (side panel)                                 | Web / tablet  |
| Place Detail Bottom Sheet                                       | Mobile        |
| Skeleton Loader                                                 | Loading state |
| Photo Gallery / Carousel                                        | Success state |
| Action Buttons (Directions, Save, Nearby, Send to phone, Share) | Success state |
| Toast notification                                              | Error state   |
| Empty State illustration                                        | not_found     |

#### Responsive Behavior

| Breakpoint                 | Layout hiển thị               | Action buttons |
| -------------------------- | ----------------------------- | -------------- |
| Mobile portrait (< 768px)  | Bottom Sheet (drag handle)    | Stack dọc      |
| Mobile landscape (< 768px) | Side panel hẹp (30% width)    | Inline ngang   |
| Tablet (768px–1024px)      | Side panel tiêu chuẩn (360px) | Inline ngang   |
| Desktop (> 1024px)         | Side panel tiêu chuẩn (400px) | Inline ngang   |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hover vào POI marker — Tooltip Preview (T01)

```gherkin
Given  người dùng đang xem bản đồ có POI markers
When   hover chuột vào một POI marker và dừng ≥ 300ms
Then   tooltip card nhỏ hiển thị cạnh (cần UX định nghĩa vị trí) marker
And    tooltip chứa: tên địa điểm, category + icon, địa chỉ + nút (direction & places saved)
And    tooltip không block tương tác với bản đồ phía sau

Given  chuột rời khỏi marker hoặc tooltip
Then   tooltip đóng sau 150ms delay (tránh flicker khi di chuyển sang tooltip)

Given  hover vào marker nhưng dừng < 300ms (di chuyển nhanh)
Then   tooltip không xuất hiện

Given  người dùng hover lại vào một POI marker đã được tải tooltip thành công trước đó
Then   tooltip hiển thị với dữ liệu đã được cache (không gọi lại API)
And    không hiển thị trạng thái loading
```

#### AC-B02 · Click vào POI marker trên bản đồ (T02)

```gherkin
Given  người dùng click / tap vào POI marker
Then   marker mất, thay thế bằng marker highlight, có hiệu ứng ripple
And    điền tên địa điểm vào search bar và hiển thị indicator loading
And    bản đồ pan nhẹ để Place Detail Panel không che marker (nếu cần)
And    hệ thống gọi Place Detail API với place_id tương ứng
And    URL cập nhật thành /place/[place_id] (không reload trang)

Given  tooltip đang mở khi click marker
Then   tooltip đóng ngay và Place Detail Panel mở thay thế

Given  người dùng click marker khác khi panel đang mở
Then   panel cập nhật nội dung địa điểm mới (không đóng-mở lại panel)
```

#### AC-B03 · Click vào danh sách kết quả (T03–T06)

```gherkin
Given  người dùng click vào item trong Search Results / Nearby / Saved / Recent
Then   bản đồ fly-to marker tương ứng và highlight marker đó + mức zoom 17
And    điền tên địa điểm vào search bar và hiển thị indicator loading
And    Place Detail Panel / Bottom Sheet mở với skeleton loader
And    hệ thống gọi Place Detail API với place_id của item
And    URL cập nhật thành /place/[place_id]

Given  item trong danh sách không còn place_id hợp lệ (đã xóa)
Then   hiển thị empty state "Địa điểm này không còn tồn tại"
```

#### AC-B04 · Deep Link URL /place/[place_id] (T07)

```gherkin
Given  URL https://{domain}/place/[place_id] hợp lệ
Then   bản đồ load và center vào địa điểm tương ứng
And    Place Detail Panel tự động mở với skeleton loader trong khi chờ dữ liệu API
And    cập nhật hiển thị thông tin đầy đủ sau khi load xong

Given  URL https://{domain}/place/[place_id] với place_id không hợp lệ
Then   hiển thị panel "Không tìm thấy địa điểm"
And    có nút "Quay lại bản đồ" để dismiss panel

Given  URL có thêm tham số zoom /place/[place_id]/@lat,lon,zoomz
Then   bản đồ mở tại zoom level chỉ định
And    Place Detail Panel vẫn mở bình thường
```

#### AC-B05 · Skeleton Loading state

```gherkin
Given  Place Detail API call đang thực hiện
When   chờ > 150ms
Then   skeleton loader hiển thị tại panel / bottom sheet
And    skeleton có placeholder cho: ảnh, tên, địa chỉ, action buttons

Given  API resolve trong < 150ms
Then   skeleton không xuất hiện (render trực tiếp kết quả)
```

> **Lý do 150ms:** Ngưỡng này tránh skeleton flash cho response nhanh, đồng thời cung cấp visual feedback kịp thời cho response chậm hơn.

#### AC-B06 · Success state (Multi-layout)

```gherkin
Given  Place Detail API trả về thành công đối tượng thuộc loại "POI / Address"
Then   panel hiển thị layout POI / Địa chỉ, bao gồm:
       - Ảnh địa điểm (carousel nếu nhiều ảnh)
       - Tên địa điểm (h1)
       - Category / loại địa điểm
       - Địa chỉ đầy đủ
       - Tọa độ (Plus Code / Lat-Long)
       - Giờ hoạt động, Số điện thoại, Website (nếu có)
       - Thêm nhãn (Add a label)
And    Action buttons hiển thị đầy đủ bộ công cụ (VD: Directions, Save, Nearby, Send to phone, Share)
And    nút [×] để đóng panel, URL trở về trạng thái trước
And    Place Detail Panel hỗ trợ scroll dọc để hiển thị các thông tin bên dưới (độc lập với bản đồ)

Given  Place Detail API trả về thành công đối tượng thuộc loại "Administrative Boundary" (Ranh giới hành chính)
Then   panel hiển thị layout Đơn vị hành chính, bao gồm:
       - Tên đơn vị hành chính (h1)
       - Phân loại hành chính (VD: Thành phố trực thuộc Trung ương, Quận/Huyện, Phường/Xã...)
       - Cấp bậc trực thuộc (VD: Nằm trong TP.HCM)
       - Thông tin thống kê ranh giới: Diện tích, quy mô, dân số (nếu có)
       - Danh sách các đơn vị cấp dưới trực thuộc
       - Tọa độ điểm trung tâm
And    bản đồ đồng thời hiển thị Polygon highlight ranh giới của đơn vị hành chính đó
And    Action buttons có thể được rút gọn phù hợp với loại dữ liệu này (VD: Directions, Share, Save)
And    nút [×] để đóng panel, URL trở về trạng thái trước
And    Place Detail Panel hỗ trợ scroll dọc để hiển thị các thông tin bên dưới

Given  người dùng click / tap vào icon copy cạnh các trường thông tin (địa chỉ, tọa độ, số điện thoại, website)
Then   hệ thống copy giá trị tương ứng vào clipboard
And    hiển thị toast thông báo "Đã sao chép vào khay nhớ tạm" (tự ẩn sau khoảng 3s)

Given  người dùng click chọn mục "Thêm nhãn" (Add a label)
Then   mở popup/form cho phép người dùng nhập nhãn mới hoặc chọn nhãn có sẵn (Nhà, Cơ quan...)
And    sau khi lưu, nhãn sẽ được hiển thị ngay trên chi tiết địa điểm và ghi nhận vào hệ thống
```

#### AC-B07 · NOT_FOUND state

```gherkin
Given  API trả về place_id không tồn tại (404)
Then   panel hiển thị empty state illustration
And    text: "Không tìm thấy địa điểm"
And    subtext: "Địa điểm này có thể đã được xóa hoặc không còn hoạt động."
And    nút "Quay lại bản đồ"
```

#### AC-B08 · Error state

```gherkin
Given  API lỗi 4xx/5xx hoặc timeout sau 5 giây (theo COMMON §5)
Then   toast "Không thể tải thông tin địa điểm, vui lòng thử lại"
And    toast tự đóng sau 4 giây
And    panel giữ nguyên (không đóng) để người dùng có thể retry
```

#### AC-B09 · Đóng Place Detail Panel

```gherkin
Given  Place Detail Panel đang mở
When   người dùng nhấn nút [×]
Then   panel đóng, marker bỏ highlight
And    URL trở về trạng thái trước (back state, không reload)

Given  Place Detail Panel đang mở
When   người dùng click vào vùng bản đồ trống (không phải marker)
Then   panel đóng
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
