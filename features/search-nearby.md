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

## Unique Selling Propositions (USP)

> Các lợi thế cạnh tranh cốt lõi của tính năng Search Nearby trên GTel Maps tập trung vào đột phá trải nghiệm người dùng và nhu cầu thị trường nội địa.

1. **Hover-to-route preview (Đột phá UX toàn cầu):**
   - **Tính năng:** Tự động hiển thị trước tuyến đường từ vị trí hiện tại đến POI tương ứng khi hover vào kết quả tìm kiếm (hỗ trợ đa phương tiện).
   - **Giá trị:** Trải nghiệm hoàn toàn mới chưa có trên bất kỳ nền tảng bản đồ nào, cho phép người dùng so sánh nhanh khoảng cách thực tế mà không cần tải trang hay chuyển màn hình.

2. **Isochrone & Isodistance Visualization (Độc quyền tại Việt Nam):**
   - **Tính năng:** Tích hợp bộ lọc vùng phủ theo thời gian đi lại (Isochrone) hoặc khoảng cách thực tế (Isodistance) hỗ trợ đa phương tiện.
   - **Giá trị:** Không đối thủ nội địa nào có; vượt trội hơn Google Maps nhờ thiết kế popup chọn chế độ rõ ràng, tường minh.

3. **Toàn quyền kiểm soát và Chia sẻ phiên tìm kiếm (Full Search State):**
   - **Tính năng:** URL Deep link lưu lại đầy đủ trạng thái tìm kiếm (query, tọa độ, bộ lọc). Người dùng được trao quyền tự quyết định có tự động cập nhật kết quả hay không thông qua checkbox "Auto-update khi di chuyển bản đồ".
   - **Giá trị:** Chia sẻ chính xác ngữ cảnh đang tìm kiếm (vượt xa các đối thủ nội địa vốn chỉ chia sẻ được 1 địa điểm tĩnh).

> Đối thủ: Tham khảo & Đánh giá

- [Google Maps](https://www.google.com/maps)
- [Bản đồ TP.HCM](https://bando.tphcm.gov.vn)
- Không đơn vị nào khác tại Việt Nam có tính năng tương tự, chỉ dừng ở mức Category chip.

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                                             | Nền tảng    | Input                                             | AC  |
| --- | ------------------------------------------------------------------- | ----------- | ------------------------------------------------- | --- |
| T01 | Click nút [Nearby] trong Action Bar của Place Detail Panel          | Web, Mobile | `place_id` hoặc `(lat, lng)` của địa điểm đang mở | B01 |
| T02 | Click vào Category chip trong Quick Filter bar (áp dụng nearby)     | Web, Mobile | `category` + `(lat, lng)` từ map center           | B02 |
| T03 | Nhập query vào Search Bar → chọn kết quả dạng "Tìm [query] gần đây" | Web, Mobile | `query` + `(lat, lng)` từ vị trí hiện tại         | B03 |
| T05 | Deep link URL `/search/[query]?near=lat,lng` hoặc `/@lat,lng,zoom`  | Web         | `query` + `(lat, lng)` từ URL params              | B05 |
| T06 | Pan / zoom bản đồ → nút "Search this area" / tự động cập nhật       | Web, Mobile | `(lat, lng)` từ map center mới                    | B06 |
| T07 | Cuộn danh sách kết quả xuống gần cuối (Infinity scroll)             | Web, Mobile | `page` / `offset` tiếp theo                       | B12 |
| T08 | Click nút "Back to top"                                             | Web, Mobile | —                                                 | B12 |
| T09 | Hover từng kết quả result trong danh sách                           | Web         | Kết quả tương ứng                                 | B08 |
| T10 | Tương tác với Filter chips (bộ lọc)                                 | Web, Mobile | Các tiêu chí lọc                                  | B13 |
| T11 | Click nút Chia sẻ (Share)                                           | Web, Mobile | —                                                 | B14 |

### States Inventory

| State          | Mô tả                            | Component                             |
| -------------- | -------------------------------- | ------------------------------------- |
| `idle`         | Chưa có tìm kiếm                 | Không có result panel / markers       |
| `loading`      | Chờ Nearby API response          | Result panel + skeleton list          |
| `success`      | Có kết quả                       | Result list + POI markers trên bản đồ |
| `zero_results` | Không có địa điểm trong bán kính | Empty state panel                     |
| `error`        | API lỗi / timeout                | Toast                                 |
| `expanded`     | Người dùng kéo xem thêm kết quả  | Bottom sheet full-screen (mobile)     |

### Components, Responsive & Typography

#### Component Inventory

| Component                                | Dùng trong              |
| ---------------------------------------- | ----------------------- |
| Nearby Result Panel (side panel)         | Web / tablet            |
| Nearby Result Bottom Sheet               | Mobile                  |
| POI Marker category trên bản đồ          | T01–T06 (success state) |
| Marker vị trí hiện tại                   | Success state           |
| Result List Item (thumbnail + details)   | Success state           |
| Skeleton List (3–5 items)                | Loading state           |
| Distance badge trên mỗi result item      | Success state           |
| Sort / Filter chips (Gần nhất, Rating)   | Success state           |
| Nút Chia sẻ (Share)                      | Success state           |
| Checkbox "Update results when map moves" | Success state           |
| Popup Isochrone / Isodistance            | T01 (khi click Nearby)  |
| Nút "Search this area"                   | T06 (search-on-move)    |
| Empty State illustration                 | zero_results state      |
| Toast notification                       | Error state             |
| Category chip (Quick Filters)            | T02                     |
| Nút "Back to top"                        | khi cuộn list kết quả   |
| Infinity scroll loader                   | khi load thêm trang     |
| Marker highlight (khi hover)             | Success state (hover)   |

#### Responsive Behavior

| Breakpoint                 | Layout hiển thị                    | Result list         |
| -------------------------- | ---------------------------------- | ------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet (peek 40%, full drag) | Scroll dọc trong BS |
| Mobile landscape (< 768px) | Side panel hẹp (35% width)         | Scroll dọc          |
| Tablet (768px–1024px)      | Side panel tiêu chuẩn (360px)      | Scroll dọc          |
| Desktop (> 1024px)         | Side panel tiêu chuẩn (400px)      | Scroll dọc          |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Nearby từ Place Detail (T01)

```gherkin
Given  Place Detail Panel đang mở
When   người dùng click nút [Nearby]
Then   hiển thị popup hỗ trợ Isochrone (thời gian) và Isodistance (khoảng cách) trên bản đồ
And    popup cho phép chọn khoảng thời gian/khoảng cách và phương tiện di chuyển (đi bộ, đi xe)
And    chuyển sang Nearby Result Panel với category mặc định "Tất cả"
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
And    checkbox "Update results when map moves" đang TẮT
And    nút "Search this area" mặc định đang ẨN
When   người dùng pan hoặc zoom bản đồ ra ngoài vị trí search cũ (di chuyển > 20% viewport)
Then   nút "Search this area" xuất hiện phía trên Result Panel

Given  người dùng click nút "Search this area"
Then   nút "Search this area" biến mất, hệ thống gọi lại Nearby API với center mới
And    result list refresh với kết quả mới
And    POI markers cũ xóa, markers mới hiển thị

Given  người dùng không click nút "Search this area" và tiếp tục pan
Then   nút "Search this area" vẫn hiển thị (không tự động trigger search)

Given  Nearby Result Panel đang mở
And    checkbox "Update results when map moves" đang BẬT
When   người dùng pan hoặc zoom bản đồ ra ngoài ngưỡng
Then   không hiển thị nút "Search this area"
And    hệ thống tự động gọi lại Nearby API với center mới
And    result list và POI markers tự động cập nhật
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
And    danh sách các marker phân biệt theo category mới xuất hiện trên bản đồ, cùng với marker vị trí hiện tại
And    khi hover từng kết quả result trong danh sách → thay thế marker đang chỉ định bằng marker mới và dẫn routing từ marker vị trí hiện tại tới marker đang chỉ định theo 2 phương tiện đi bộ hoặc đi xe (web)
And    khi bỏ hover → ẩn routing và marker trở về bình thường
And    Sort / Filter chips (Gần nhất, Đánh giá cao, Khu vực, Khoảng cách) hiển thị đầu danh sách
And    Nút Chia sẻ (Share) hiển thị cho phép chia sẻ kết quả
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

#### AC-B12 · Infinity scroll & Back to top (T07, T08)

```gherkin
Given  Nearby Result Panel đang hiển thị danh sách kết quả
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

#### AC-B13 · Lọc kết quả với Filter chips

```gherkin
Given  Nearby Result Panel đang hiển thị
When   người dùng tương tác với Filter chips (ví dụ: giá, khu vực, khoảng cách, v.v.)
Then   kết quả tìm kiếm trong danh sách được cập nhật lại theo điều kiện lọc
And    danh sách các marker trên bản đồ tương ứng cập nhật theo kết quả mới
```

#### AC-B14 · Tính năng chia sẻ

```gherkin
Given  Nearby Result Panel đang hiển thị kết quả
When   người dùng click nút Chia sẻ (Share)
Then   hệ thống lấy link theo thông tin tìm kiếm gần nhất bao gồm vị trí và bộ lọc
And    hiển thị công cụ chia sẻ hoặc sao chép vào clipboard để gửi cho người khác
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
