# Forward Geocoding

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

Forward Geocoding chuyển đổi text query (địa chỉ, tên đường, tên POI, tên doanh nghiệp, danh mục) thành tập hợp kết quả địa lý có tọa độ, xếp hạng theo độ liên quan. Kết quả được hiển thị trong Search Results Panel và markers trên bản đồ.

### Why

- **User need:** Người dùng cần tìm kiếm bằng ngôn ngữ tự nhiên ("Hội An", "bệnh viện Bạch Mai", "cà phê gần đây") và nhận kết quả chính xác, có thể click để xem chi tiết.
- **Business value:** Là tính năng cốt lõi của Maps; search volume là KPI hàng đầu; là bề mặt chính cho Promoted Places và search ads.

---

# Part 02 - Specifications

## Backend

_Search API (Forward Geocoding + POI Search endpoint)._

## UI/UX & Frontend

### Search Scope

Forward Geocoding hỗ trợ các loại query sau:
- **Address search:** Địa chỉ đường phố đầy đủ hoặc một phần ("123 Lê Lợi, Q.1")
- **Street name search:** Tên đường ("đường Nguyễn Huệ")
- **POI search:** Tên địa điểm cụ thể ("Nhà thờ Đức Bà", "Vincom Center")
- **Business name search:** Tên doanh nghiệp ("Pizza 4P's", "Grab Vietnam")
- **Category search:** Loại địa điểm ("nhà hàng", "bệnh viện", "ATM")

### Triggers & Entry Points

| ID  | Trigger                                              | Nền tảng    | Input                           | AC  |
| --- | ---------------------------------------------------- | ----------- | ------------------------------- | --- |
| T01 | Submit Search Bar (Enter / tap Search button)        | Web, Mobile | Text query từ Search Bar        | B01 |
| T02 | Chọn suggestion từ Autocomplete dropdown             | Web, Mobile | `query` hoặc `place_id`         | B02 |
| T03 | Deep link URL `/search/[query]/@lat,lng,zoom`        | Web         | `query` + coordinates từ URL    | B03 |
| T04 | Click "Xem tất cả kết quả" trong Autocomplete        | Web, Mobile | Current query text              | B01 |

### States Inventory

| State          | Mô tả                                 | Component                          |
| -------------- | ------------------------------------- | ---------------------------------- |
| `loading`      | Đang gọi Search API                   | Search Results Panel + skeleton    |
| `success`      | Có kết quả                            | Result list + markers trên bản đồ  |
| `zero_results` | Không tìm thấy kết quả                | Empty state panel                  |
| `error`        | API lỗi / timeout                     | Toast                              |

### Components, Responsive & Typography

#### Component Inventory

| Component                                      | Dùng trong     |
| ---------------------------------------------- | -------------- |
| Search Results Panel (side panel)              | Web / tablet   |
| Search Results Bottom Sheet                    | Mobile         |
| Result List Item (icon + name + address + dist)| success state  |
| Result markers trên bản đồ                     | success state  |
| Skeleton list (3–5 items)                      | loading state  |
| Empty State illustration                       | zero_results   |
| Toast notification                             | error state    |
| "Tìm gần đây" suggestion chip                  | B01            |

#### Responsive Behavior

| Breakpoint                 | Layout                            |
| -------------------------- | --------------------------------- |
| Mobile portrait (< 768px)  | Bottom Sheet (peek 50%, drag up)  |
| Tablet (768px–1024px)      | Side panel (360px)                |
| Desktop (> 1024px)         | Side panel (400px)                |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Submit search query (T01, T04)

```gherkin
Given  người dùng nhập query và nhấn Enter / tap Search
Then   Search API được gọi với query + location bias (map center)
And    URL cập nhật thành /search/[encoded_query]/@lat,lng,zoom
And    Search Results Panel mở với skeleton loader
And    query được lưu vào Recent Searches

Given  query là tọa độ decimal ("10.77, 106.70")
Then   chuyển sang Reverse Geocoding flow (xem reverse-geocoding.md §B11)
```

#### AC-B02 · Chọn từ Autocomplete (T02)

```gherkin
Given  người dùng chọn suggestion có place_id
Then   Skip search results, mở trực tiếp Place Detail (xem place-detail.md §B03)
And    bản đồ fly-to địa điểm

Given  người dùng chọn suggestion chỉ có address (không có place_id)
Then   Search API được gọi với address query
And    Search Results Panel hiển thị kết quả
```

#### AC-B03 · Deep link URL (T03)

```gherkin
Given  URL /search/nh%C3%A0+h%C3%A0ng/@10.77,106.70,14z
Then   Search API gọi với query "nhà hàng" và location bias (10.77, 106.70)
And    Search Results Panel tự động mở với kết quả
```

#### AC-B04 · Success state

```gherkin
Given  Search API trả về kết quả
Then   danh sách hiển thị, mỗi item gồm:
       - Category icon
       - Tên địa điểm / địa chỉ (primary)
       - Địa chỉ đầy đủ hoặc loại địa điểm (secondary)
       - Khoảng cách từ map center (nếu áp dụng)
And    markers xuất hiện trên bản đồ cho từng kết quả
And    bản đồ fit bounds để hiển thị tất cả markers
And    hover item → highlight marker tương ứng (web)
And    click item → mở Place Detail (xem place-detail.md §B03)
```

#### AC-B05 · ZERO_RESULTS state

```gherkin
Given  Search API trả về không có kết quả
Then   empty state hiển thị:
       text: "Không tìm thấy kết quả cho '[query]'"
       suggestions: "Thử tìm với từ khóa khác" + example suggestions
```

#### AC-B06 · Error state

```gherkin
Given  Search API lỗi 4xx/5xx hoặc timeout (theo COMMON §5)
Then   toast "Không thể thực hiện tìm kiếm, vui lòng thử lại"
And    toast tự đóng sau 4 giây
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
