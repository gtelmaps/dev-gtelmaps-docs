# Category Quick Filters

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

Category Quick Filters là thanh chip filter nằm ngang phía trên bản đồ (hoặc dưới Search Bar), cho phép người dùng lọc và tìm kiếm các POI theo danh mục (nhà hàng, ATM, bệnh viện, trạm xăng...) bằng một thao tác tap/click, mà không cần nhập text. Mỗi chip khi được chọn sẽ kích hoạt Search Nearby theo category tương ứng.

### Why

- **User need:** Người dùng thường biết mình cần loại địa điểm gì (ăn uống, đổ xăng, rút tiền) nhưng không muốn gõ tìm kiếm — một tap nhanh trên chip tiết kiệm thời gian đáng kể.
- **Business value:** Quick Filters là bề mặt quảng cáo tự nhiên (Sponsored category placement); tăng discoverability cho các loại POI có traffic thấp; dữ liệu tap trên chip phản ánh intent người dùng theo thời gian thực.
- **Data sovereignty:** Cấu hình danh sách category và thứ tự hiển thị được quản lý trên hạ tầng GTEL (CMS-driven).

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                        | So sánh                                              |
| -- | -------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | CMS-driven category list         | Danh sách chip được quản lý qua CMS — PO thay đổi ngay, không cần deploy code               | Google Maps hard-code danh sách category              |
| 2  | Category theo ngữ cảnh VN        | Danh mục tối ưu cho thị trường VN: quán cà phê, bưu điện, chợ, trạm xe buýt...              | Google Maps dùng danh mục global, thiếu local context |
| 3  | Sponsored category placement     | Hỗ trợ quảng cáo tự nhiên — thương hiệu có thể mua vị trí chip ưu tiên                      | Google Maps không mở tính năng này cho bên thứ ba     |
| 4  | Realtime intent analytics        | Mỗi tap trên chip ghi nhận intent người dùng theo thời gian thực — data cho business insight | Google Maps không chia sẻ dữ liệu intent chi tiết    |
| 5  | Config per region                | Hỗ trợ danh sách chip khác nhau theo vùng miền (HCM vs Hà Nội vs nông thôn)                  | Google Maps dùng một danh sách toàn cầu              |

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Filter bar (Thanh tìm kiếm nhanh):** Thanh chứa các chip danh mục nằm ngang phía trên bản đồ hoặc bên dưới Search Bar.
- **Place Detail / Bản đồ:** Tương tác chuyển tiếp từ một địa điểm hoặc thao tác bản đồ (pan/zoom).

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                                                         | Nền tảng    | Input                                                   | AC  |
| --- | ------------------------------------------------------------------------------- | ----------- | ------------------------------------------------------- | --- |
| T01 | Click / tap vào Category chip trong Filter bar (trạng thái idle)                | Web, Mobile | `category_id` + `(lat, lng)` từ map center              | B01 |
| T02 | Click / tap lại vào Category chip đang active (toggle off)                      | Web, Mobile | `category_id` đang active                               | B02 |
| T03 | Chọn category từ Place Detail → [Nearby] với category được pre-fill             | Web, Mobile | `category_id` từ Place Detail + `(lat, lng)`            | B03 |
| T04 | URL deep link `?category=[category_id]` hoặc `/search/[category]/@lat,lng,zoom` | Web         | `category_id` + `(lat, lng)` từ URL params              | B04 |
| T05 | Scroll ngang Filter bar để xem thêm category                                    | Web, Mobile | —                                                       | B05 |
| T06 | Pan / zoom bản đồ → nút "Search this area" / tự động cập nhật                   | Web, Mobile | `(lat, lng)` từ map center mới                          | B06 |
| T07 | Click / tap nút "Search this area"                                              | Web, Mobile | `category_id` đang active + `(lat, lng)` map center mới | B06 |
| T08 | Cuộn danh sách kết quả xuống gần cuối (Infinity scroll)                         | Web, Mobile | `page` / `offset` API                                   | B12 |

### States Inventory

| State          | Mô tả                                          | Component                                    |
| -------------- | ---------------------------------------------- | -------------------------------------------- |
| `idle`         | Không có chip nào được chọn                    | Filter bar hiển thị tất cả chips bình thường |
| `active`       | Một chip đang được chọn                        | Chip active highlighted, Nearby loading/open |
| `loading`      | Chip vừa được click, đang gọi Nearby API       | Chip active + spinner nhỏ trên chip          |
| `success`      | Nearby API trả về kết quả                      | Chip active, markers + Result Panel hiển thị |
| `zero_results` | Không có POI theo category trong vùng hiện tại | Chip active, Result Panel empty state        |
| `error`        | Nearby API lỗi                                 | Chip reset về idle, Toast                    |

### Components, Responsive & Typography

#### Component Inventory

| Component                                    | Dùng trong               |
| -------------------------------------------- | ------------------------ |
| Filter Bar (horizontal scroll container)     | Tất cả states            |
| Category Chip (icon + label)                 | Tất cả states            |
| Category Chip — Active variant               | active, loading, success |
| Category Chip — Loading variant (spinner)    | loading state            |
| Scroll arrow indicator (trái / phải)         | Web — khi có overflow    |
| POI Markers theo category (màu / icon riêng) | success state            |
| Nearby Result Panel (kế thừa search-nearby)  | success, zero_results    |
| Toast notification                           | error state              |
| Checkbox "Update results when map moves"     | khi có filter active     |
| Nút "Search this area"                       | khi bản đồ pan / zoom    |
| Nút "Back to top"                            | khi cuộn list kết quả    |
| Infinity scroll loader                       | khi load thêm trang      |
| Marker highlight (khi hover)                 | success state (hover)    |

#### Responsive Behavior

| Breakpoint                 | Filter bar layout                             | Chip style                   |
| -------------------------- | --------------------------------------------- | ---------------------------- |
| Mobile portrait (< 768px)  | Scroll ngang toàn chiều rộng, dưới Search Bar | Compact (icon + short label) |
| Mobile landscape (< 768px) | Scroll ngang, dưới Search Bar                 | Standard                     |
| Tablet (768px–1024px)      | Scroll ngang, dưới Search Bar                 | Standard                     |
| Desktop (> 1024px)         | Scroll ngang + arrow indicators khi overflow  | Standard + arrow btn         |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Click / tap vào Category chip (T01)

```gherkin
Given  người dùng đang xem bản đồ, không có filter nào active
When   click / tap vào một Category chip (ví dụ: "Nhà hàng")
Then   chip chuyển sang active state (highlighted background + checkmark icon)
And    spinner nhỏ hiển thị trên chip trong khi gọi API
And    hệ thống gọi Search Nearby API với category tương ứng và map center hiện tại
And    URL cập nhật: thêm `?category=[category_id]` (không reload trang)
And    Nearby Result Panel / Bottom Sheet mở khi có kết quả

Given  chỉ một chip được active tại một thời điểm (single-select)
When   người dùng click chip B khi chip A đang active
Then   chip A về idle, chip B chuyển sang active
And    Search Nearby được gọi lại với category của chip B
```

#### AC-B02 · Toggle off chip đang active (T02)

```gherkin
Given  một Category chip đang ở active state
When   người dùng click / tap lại vào chip đó
Then   chip trở về idle state
And    Nearby Result Panel đóng
And    POI markers của category đó bị xóa khỏi bản đồ
And    URL xóa query param `?category=`
```

#### AC-B03 · Category pre-fill từ Place Detail (T03)

```gherkin
Given  người dùng đang xem Place Detail của một địa điểm có category "Cà phê"
When   click nút [Nearby] trong Place Detail
Then   Nearby Result Panel mở với category "Cà phê" được pre-fill
And    Category chip "Cà phê" trong Filter bar chuyển sang active state
And    tìm kiếm các địa điểm "Cà phê" xung quanh vị trí Place Detail đó
```

#### AC-B04 · Deep Link URL (T04)

```gherkin
Given  URL https://{domain}/?category=restaurant&@lat,lng,zoom hợp lệ
Then   bản đồ load tại tọa độ và zoom chỉ định
And    chip category tương ứng được active ngay khi load
And    Nearby Result Panel tự động mở với kết quả

Given  category_id trong URL không tồn tại trong danh sách chip
Then   filter bị bỏ qua (không active chip nào), bản đồ load bình thường
```

#### AC-B05 · Scroll ngang Filter bar (T05)

```gherkin
Given  số lượng category chip vượt quá chiều rộng viewport
Then   Filter bar có thể scroll ngang (touch scroll trên mobile)
And    trên desktop: hiển thị arrow button [<] [>] khi có nội dung ẩn
And    arrow button [<] ẩn khi đang ở đầu danh sách
And    arrow button [>] ẩn khi đang ở cuối danh sách

Given  người dùng scroll ngang trong khi có chip active
Then   chip active vẫn visible (auto-scroll chip active vào viewport)
```

#### AC-B06 · Search-on-move khi filter active (T06, T07)

```gherkin
Given  một Category chip đang active và Result Panel đang mở
And    checkbox "Update results when map moves" đang TẮT
And    nút "Search this area" mặc định đang ẨN
When   người dùng pan / zoom bản đồ ra ngoài vị trí search cũ (> 20% viewport shift)
Then   nút "Search this area" hiển thị (kế thừa search-nearby §B06)

Given  người dùng click nút "Search this area"
Then   Nearby API được gọi lại với category đang active + map center mới
And    markers và Result Panel cập nhật với kết quả mới
And    nút "Search this area" biến mất

Given  một Category chip đang active và Result Panel đang mở
And    checkbox "Update results when map moves" đang BẬT
When   người dùng pan / zoom bản đồ ra ngoài ngưỡng (> 20% viewport shift)
Then   không hiển thị nút "Search this area"
And    Nearby API tự động được gọi lại với category đang active + map center mới
And    markers và Result Panel tự động cập nhật với kết quả mới
```

#### AC-B07 · Loading state

```gherkin
Given  Category chip vừa được click và API đang chạy
When   chờ > 150ms
Then   spinner nhỏ hiển thị trong chip active (thay icon category)
And    Nearby Result Panel hiển thị skeleton list

Given  API resolve trong < 150ms
Then   spinner không hiển thị, kết quả render trực tiếp
```

#### AC-B08 · Success state

```gherkin
Given  Search Nearby API trả về kết quả cho category được chọn
Then   spinner trên chip dừng, chip giữ active với category icon
And    POI markers màu / icon riêng của category xuất hiện trên bản đồ
And    Nearby Result Panel hiển thị danh sách kết quả (kế thừa search-nearby §B08)
And    header của Result Panel hiển thị: "X địa điểm [Category] gần đây"
```

#### AC-B09 · ZERO_RESULTS state

```gherkin
Given  Search Nearby API trả về không có kết quả cho category trong vùng hiện tại
Then   chip giữ active state
And    Result Panel hiển thị empty state:
       text: "Không tìm thấy [Category] nào gần đây"
       nút: [Mở rộng tìm kiếm]
```

#### AC-B10 · Error state

```gherkin
Given  Search Nearby API lỗi 4xx/5xx hoặc timeout
Then   chip reset về idle state
And    Result Panel đóng
And    toast "Không thể tải kết quả, vui lòng thử lại"
And    toast tự đóng sau 4 giây
```

#### AC-B11 · Cấu hình danh sách category (CMS-driven)

```gherkin
Given  danh sách category và thứ tự được cấu hình từ CMS / backend
When   app load
Then   Filter bar hiển thị đúng danh sách category theo thứ tự CMS định nghĩa
And    category có flag sponsored hiển thị badge "Nổi bật" (nếu có)

Given  danh sách category không fetch được từ CMS (lỗi / offline)
Then   Filter bar dùng danh sách default hardcode (fallback)
And    không hiển thị badge sponsored
```

#### AC-B12 · Infinity scroll & Back to top (T08)

```gherkin
Given  Result Panel đang hiển thị danh sách kết quả
When   người dùng cuộn danh sách xuống dưới
Then   nút "Back to top" xuất hiện nổi trên danh sách

Given  người dùng cuộn xuống gần cuối danh sách
Then   hệ thống tự động trigger gọi API để tải thêm kết quả (Infinity scroll)
And    hiển thị loader báo đang tải ở cuối danh sách
And    kết quả mới được nạp và thêm vào cuối danh sách

Given  người dùng click nút "Back to top"
Then   danh sách tự động cuộn lên trên cùng (smooth scroll)
And    nút "Back to top" biến mất
```

#### AC-B13 · Hover kết quả result

```gherkin
Given  danh sách kết quả đang hiển thị trên thiết bị hỗ trợ hover (Web)
When   người dùng hover từng kết quả result trong danh sách
Then   marker hiện tại tương ứng trên bản đồ sẽ thay thế bằng marker highlight
And    khi người dùng bỏ hover, marker highlight trở về marker hiện tại bình thường
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
