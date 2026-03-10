# Search Nearby

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

Search Nearby tìm kiếm các địa điểm (POI) trong bán kính xung quanh một vị trí tham chiếu (vị trí hiện tại, marker đang chọn, hoặc trung tâm bản đồ), trả về danh sách kết quả theo khoảng cách hoặc độ liên quan, và hiển thị marker trên bản đồ.

### Why

- **User need:** Người dùng cần biết "xung quanh đây có gì?" — tìm quán ăn, ATM, bệnh viện, trạm xăng... gần vị trí hiện tại hoặc một địa điểm đang xem mà không cần nhập query phức tạp.
- **Business value:** Search Nearby là entry point chính cho Promoted Places và ads địa điểm; tăng engagement session, là API nền tảng cho Fleet (tìm trạm dừng) và Logistics (tìm kho gần nhất).
- **Data sovereignty:** Dữ liệu địa điểm và kết quả tìm kiếm được xử lý trên hạ tầng GTEL.

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                                     | Nền tảng    | Input                                             | AC  |
| --- | --------------------------------------------------------------------------- | ----------- | ------------------------------------------------- | --- |
| T01 | Click nút [Nearby] trong Action Bar của Place Detail Panel                  | Web, Mobile | `place_id` hoặc `(lat, lng)` của địa điểm đang mở | B01 |
| T02 | Click vào Category chip trong Quick Filter bar (áp dụng nearby)            | Web, Mobile | `category` + `(lat, lng)` từ map center           | B02 |
| T03 | Nhập query vào Search Bar → chọn kết quả dạng "Tìm [query] gần đây"        | Web, Mobile | `query` + `(lat, lng)` từ vị trí hiện tại         | B03 |
| T04 | Click nút [Khám phá khu vực này] khi bản đồ ở trạng thái idle              | Web, Mobile | `(lat, lng)` từ map center                        | B04 |
| T05 | Deep link URL `/search/[query]?near=lat,lng` hoặc `/@lat,lng,zoom`         | Web         | `query` + `(lat, lng)` từ URL params              | B05 |
| T06 | Sau khi bản đồ pan / zoom → "Tìm kiếm trong khu vực này" (search-on-move) | Web, Mobile | `(lat, lng)` từ map center mới                    | B06 |

### States Inventory

| State          | Mô tả                             | Component                                |
| -------------- | --------------------------------- | ---------------------------------------- |
| `idle`         | Chưa có tìm kiếm                  | Không có result panel / markers          |
| `loading`      | Chờ Nearby API response           | Result panel + skeleton list             |
| `success`      | Có kết quả                        | Result list + POI markers trên bản đồ    |
| `zero_results` | Không có địa điểm trong bán kính  | Empty state panel                        |
| `error`        | API lỗi / timeout                 | Toast                                    |
| `expanded`     | Người dùng kéo xem thêm kết quả  | Bottom sheet full-screen (mobile)        |

### Components, Responsive & Typography

#### Component Inventory

| Component                              | Dùng trong                  |
| -------------------------------------- | --------------------------- |
| Nearby Result Panel (side panel)       | Web / tablet                |
| Nearby Result Bottom Sheet             | Mobile                      |
| POI Marker cluster trên bản đồ         | T01–T06 (success state)     |
| Result List Item (thumbnail + details) | Success state               |
| Skeleton List (3–5 items)              | Loading state               |
| Distance badge trên mỗi result item    | Success state               |
| Sort / Filter chips (Gần nhất, Rating) | Success state               |
| "Tìm kiếm trong khu vực này" banner    | T06 (search-on-move)        |
| Empty State illustration               | zero_results state          |
| Toast notification                     | Error state                 |
| Category chip (Quick Filters)          | T02                         |

#### Responsive Behavior

| Breakpoint                 | Layout hiển thị                        | Result list         |
| -------------------------- | -------------------------------------- | ------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet (peek 40%, full drag)     | Scroll dọc trong BS |
| Mobile landscape (< 768px) | Side panel hẹp (35% width)             | Scroll dọc          |
| Tablet (768px–1024px)      | Side panel tiêu chuẩn (360px)          | Scroll dọc          |
| Desktop (> 1024px)         | Side panel tiêu chuẩn (400px)          | Scroll dọc          |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Nearby từ Place Detail (T01)

```gherkin
Given  Place Detail Panel đang mở
When   người dùng click nút [Nearby]
Then   chuyển sang Nearby Result Panel với category mặc định "Tất cả"
And    bán kính tìm kiếm mặc định 500m xung quanh địa điểm đó
And    URL cập nhật thành /search/nearby?place_id=[id] hoặc /@lat,lng,zoom
And    Place Detail Panel đóng, nhường chỗ cho Result Panel
```

#### AC-B02 · Nearby từ Category Quick Filter (T02)

```gherkin
Given  người dùng đang xem bản đồ (không cần có Place Detail mở)
When   click vào một Category chip (ví dụ: "Nhà hàng", "ATM", "Bệnh viện")
Then   hệ thống lấy tọa độ trung tâm bản đồ hiện tại làm điểm tham chiếu
And    gọi Nearby API với category tương ứng
And    hiển thị POI markers có category icon phân biệt trên bản đồ
And    Nearby Result Panel / Bottom Sheet mở với danh sách kết quả
And    Category chip được đánh dấu active

Given  người dùng click lại vào Category chip đang active
Then   filter bị hủy, markers ẩn, Result Panel đóng
```

#### AC-B03 · Nearby từ Search Bar (T03)

```gherkin
Given  người dùng nhập query vào Search Bar
When   autocomplete hiển thị suggestion dạng 'Tìm "[query]" gần đây'
And    người dùng chọn suggestion đó
Then   hệ thống dùng vị trí GPS hiện tại (hoặc map center nếu GPS không có) làm điểm tham chiếu
And    gọi Nearby API với query và tọa độ tham chiếu
And    kết quả hiển thị trên Result Panel và bản đồ

Given  vị trí GPS chưa được cấp quyền
Then   hệ thống dùng map center làm fallback (không yêu cầu location permission ở bước này)
```

#### AC-B04 · Nearby từ "Khám phá khu vực này" (T04)

```gherkin
Given  bản đồ đang ở trạng thái idle (không có panel nào mở)
When   người dùng click nút [Khám phá khu vực này]
Then   gọi Nearby API với category "Tất cả" và tọa độ center bản đồ
And    Result Panel mở với kết quả tổng hợp
```

#### AC-B05 · Deep Link URL (T05)

```gherkin
Given  URL https://{domain}/search/[query]?near=lat,lng hợp lệ
Then   bản đồ load và center vào tọa độ near
And    Nearby Result Panel tự động mở với kết quả tìm kiếm

Given  URL thiếu near param hoặc tọa độ không hợp lệ
Then   hệ thống fallback về map center mặc định và thực hiện tìm kiếm
```

#### AC-B06 · Search-on-move (T06)

```gherkin
Given  Nearby Result Panel đang mở
When   người dùng pan hoặc zoom bản đồ ra ngoài ngưỡng (di chuyển > 20% viewport)
Then   banner "Tìm kiếm trong khu vực này" xuất hiện phía trên Result Panel

Given  người dùng click banner "Tìm kiếm trong khu vực này"
Then   banner biến mất, hệ thống gọi lại Nearby API với center mới
And    result list refresh với kết quả mới
And    POI markers cũ xóa, markers mới hiển thị

Given  người dùng không click banner và tiếp tục pan
Then   banner vẫn hiển thị (không tự động trigger search)
```

#### AC-B07 · Skeleton Loading state

```gherkin
Given  Nearby API call đang thực hiện
When   chờ > 150ms
Then   skeleton list 3–5 items hiển thị tại Result Panel / Bottom Sheet

Given  API resolve trong < 150ms
Then   skeleton không xuất hiện (render trực tiếp kết quả)
```

#### AC-B08 · Success state

```gherkin
Given  Nearby API trả về có kết quả
Then   danh sách items hiển thị, mỗi item gồm:
       - Thumbnail ảnh địa điểm (hoặc category icon placeholder)
       - Tên địa điểm
       - Category label
       - Khoảng cách từ điểm tham chiếu (ví dụ: "350m", "1.2km")
       - Rating ngắn gọn (nếu có)
And    POI markers tương ứng xuất hiện trên bản đồ
And    hover item trong list → highlight marker tương ứng trên bản đồ (web)
And    Sort chips: [Gần nhất] [Đánh giá cao] hiển thị đầu danh sách
```

#### AC-B09 · ZERO_RESULTS state

```gherkin
Given  Nearby API trả về không có kết quả trong bán kính
Then   hiển thị empty state illustration
And    text: "Không tìm thấy địa điểm nào gần đây"
And    subtext: "Hãy thử tìm kiếm ở khu vực khác hoặc mở rộng bán kính"
And    nút [Mở rộng tìm kiếm] để tăng bán kính lên 2x
```

#### AC-B10 · Error state

```gherkin
Given  API lỗi 4xx/5xx hoặc timeout sau 5 giây (theo COMMON §5)
Then   toast "Không thể tải kết quả, vui lòng thử lại"
And    toast tự đóng sau 4 giây
And    Result Panel giữ nguyên (không đóng) để người dùng có thể retry
```

#### AC-B11 · Đóng Nearby Result Panel

```gherkin
Given  Nearby Result Panel đang mở
When   người dùng nhấn nút [×] hoặc Escape (web)
Then   panel đóng, POI markers của nearby bị xóa khỏi bản đồ
And    Category chip (nếu có) bỏ active
And    URL trở về trạng thái trước

Given  người dùng click vào một item trong danh sách
Then   Place Detail Panel mở cho item đó (xem AC place-detail §B03)
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
