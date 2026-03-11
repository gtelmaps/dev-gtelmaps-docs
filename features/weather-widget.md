# Weather Widget

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

Weather Widget là thành phần UI hiển thị thông tin thời tiết hiện tại (nhiệt độ, tình trạng thời tiết, icon) tại khu vực trung tâm bản đồ đang hiển thị. Widget cập nhật tự động khi bản đồ di chuyển đến vùng địa lý mới và cho phép xem chi tiết dự báo khi tương tác.

### Why

- **User need:** Người dùng cần biết thời tiết tại khu vực đang xem trên bản đồ — đặc biệt khi lập kế hoạch di chuyển, logistics, hoặc khám phá địa điểm ngoài trời.
- **Business value:** Weather Widget tăng tính hữu ích của Maps Viewer trong lập lịch và vận tải; là điểm tích hợp dịch vụ thời tiết của GTEL và đối tác B2B (Fleet, Logistics).
- **Data sovereignty:** Dữ liệu thời tiết được lấy từ Weather API nội bộ hoặc đối tác được GTEL phê duyệt.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                     | So sánh                                              |
| -- | -------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Weather API nội bộ GTEL          | Dữ liệu thời tiết từ nguồn nội địa hoặc đối tác GTEL phê duyệt — data sovereignty        | Google Maps không có weather widget tích hợp sẵn     |
| 2  | Cập nhật theo vùng bản đồ        | Widget tự cập nhật khi pan bản đồ sang vùng địa lý mới                                    | Google Maps không hiển thị thời tiết trên bản đồ     |
| 3  | Tích hợp Fleet / Logistics       | Cung cấp dữ liệu thời tiết realtime cho lập kế hoạch vận chuyển — API nội bộ, không giới hạn call | Không có tương đương trên Google Maps               |

---

# Part 02 - Specifications

## Backend

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Sự kiện Bản đồ (Map Events):** Khởi động ứng dụng, tải bản đồ hoặc sau khi tương tác di chuyển.
- **Weather Widget:** Tương tác trực tiếp trên giao diện của widget (click, pull-to-refresh).
- **Hệ thống (System):** Hệ thống cập nhật ngầm khi dữ liệu hết hạn.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                                          | Nền tảng    | Input                                     | AC  |
| --- | ---------------------------------------------------------------- | ----------- | ----------------------------------------- | --- |
| T01 | Bản đồ load lần đầu (khởi động app)                             | Web, Mobile | `(lat, lng)` từ map center khởi tạo       | B01 |
| T02 | Bản đồ dừng di chuyển sau pan / zoom (idle event)               | Web, Mobile | `(lat, lng)` từ map center mới            | B02 |
| T03 | Click / tap vào Weather Widget (mở Weather Detail)              | Web, Mobile | `(lat, lng)` hiện tại                     | B03 |
| T04 | Pull-to-refresh trên mobile / nút refresh trong Weather Detail  | Mobile, Web | `(lat, lng)` hiện tại                     | B04 |
| T05 | TTL cache hết hạn — tự động refetch nền                         | Web, Mobile | `(lat, lng)` hiện tại                     | B05 |

### States Inventory

| State     | Mô tả                                           | Component                            |
| --------- | ----------------------------------------------- | ------------------------------------ |
| `hidden`  | Widget ẩn (feature flag off hoặc API lỗi liên tiếp) | Không hiển thị                    |
| `loading` | Đang fetch thời tiết lần đầu                    | Widget skeleton (icon + temp placeholder) |
| `success` | Có dữ liệu thời tiết                            | Widget đầy đủ: icon + nhiệt độ + mô tả |
| `stale`   | Đang refetch nền, đang dùng dữ liệu cũ          | Widget hiển thị bình thường + spinner nhỏ |
| `error`   | Fetch thất bại, không có cache                  | Widget ẩn hoặc icon "?" + tooltip lỗi |

### Components, Responsive & Typography

#### Component Inventory

| Component                                    | Dùng trong                  |
| -------------------------------------------- | --------------------------- |
| Weather Widget (compact)                     | T01–T05 (corner overlay)    |
| Weather icon (animated SVG / lottie)         | Success, stale state        |
| Temperature display                          | Success, stale state        |
| Stale refresh spinner                        | Stale state                 |
| Weather Detail Popover / Bottom Sheet        | T03                         |
| Hourly forecast row (24h)                    | Weather Detail              |
| Daily forecast list (7 ngày)                 | Weather Detail              |
| Weather condition description                | Weather Detail              |
| Wind, humidity, UV index summary             | Weather Detail              |
| Refresh button                               | T04 (Weather Detail header) |

#### Responsive Behavior

| Breakpoint                 | Widget position              | Weather Detail layout       |
| -------------------------- | ---------------------------- | --------------------------- |
| Mobile portrait (< 768px)  | Bottom-right, above zoom btn | Bottom Sheet (half-screen)  |
| Mobile landscape (< 768px) | Top-right corner             | Popover cạnh widget         |
| Tablet (768px–1024px)      | Top-right corner             | Popover cạnh widget         |
| Desktop (> 1024px)         | Top-right corner             | Popover cạnh widget         |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Khởi tạo widget khi bản đồ load (T01)

```gherkin
Given  ứng dụng Maps Viewer khởi động
When   bản đồ render xong lần đầu
Then   Weather Widget hiển thị ở vị trí cố định (corner overlay)
And    hệ thống gọi Weather API với tọa độ center bản đồ khởi tạo
And    widget hiển thị skeleton (icon placeholder + "–°") trong khi chờ

Given  đã có cache thời tiết hợp lệ cho vùng tọa độ này (TTL chưa hết)
Then   widget render ngay với dữ liệu cache (không gọi lại API)
And    không hiển thị skeleton
```

#### AC-B02 · Cập nhật khi bản đồ di chuyển (T02)

```gherkin
Given  Weather Widget đang hiển thị
When   người dùng pan / zoom bản đồ và bản đồ dừng lại (map idle event)
And    tọa độ center mới cách tọa độ cũ > 50km (ngưỡng trigger refetch)
Then   hệ thống fetch Weather API với center mới
And    widget chuyển sang stale state (hiển thị dữ liệu cũ + spinner nhỏ)
And    sau khi fetch xong, widget cập nhật với dữ liệu mới (transition mượt)

Given  pan bản đồ nhưng tọa độ center mới vẫn trong vùng ≤ 50km
Then   Weather Widget không refetch (dùng cache hiện tại)
```

> **Lý do ngưỡng 50km:** Thời tiết không thay đổi đáng kể trong bán kính nhỏ, tránh quá nhiều API call khi người dùng pan liên tục.

#### AC-B03 · Click vào Widget — Weather Detail (T03)

```gherkin
Given  Weather Widget đang hiển thị ở success state
When   người dùng click / tap vào widget
Then   Weather Detail Popover / Bottom Sheet mở
And    hiển thị:
       - Tên tỉnh / thành phố của vị trí hiện tại
       - Nhiệt độ hiện tại + cảm giác như
       - Mô tả tình trạng (ví dụ: "Có mây rải rác")
       - Dự báo theo giờ (24h tới): icon + nhiệt độ
       - Dự báo 7 ngày: icon + min/max temp
       - Chỉ số bổ sung: Gió (km/h), Độ ẩm (%), Chỉ số UV
And    có nút [×] để đóng Weather Detail

Given  người dùng click ra ngoài Popover (web)
Then   Weather Detail đóng
```

#### AC-B04 · Refresh thủ công (T04)

```gherkin
Given  Weather Detail đang mở
When   người dùng nhấn nút [Refresh] trong header
Then   Weather API được gọi lại với tọa độ hiện tại
And    nút Refresh hiển thị spinner trong khi fetch
And    dữ liệu Weather Detail cập nhật sau khi fetch xong
And    TTL cache được reset

Given  mobile — người dùng pull-to-refresh trong Weather Detail Bottom Sheet
Then   tương tự hành vi nhấn nút Refresh
```

#### AC-B05 · TTL cache và auto-refetch nền (T05)

```gherkin
Given  dữ liệu thời tiết đã được cache
When   TTL hết hạn (mặc định 10 phút)
Then   hệ thống tự động refetch nền mà không gián đoạn UI
And    widget chuyển sang stale state trong khi refetch
And    sau khi fetch xong, dữ liệu cập nhật ngầm

Given  người dùng đang mở Weather Detail khi auto-refetch hoàn tất
Then   nội dung Weather Detail cập nhật ngầm (không close / reopen)
```

#### AC-B06 · Loading state

```gherkin
Given  Weather API call đang thực hiện lần đầu (không có cache)
Then   widget hiển thị skeleton: icon placeholder hình tròn + text "–°"
And    skeleton không nhấp nháy quá nhanh (animation ≥ 1.5s cycle)
```

#### AC-B07 · Error state

```gherkin
Given  Weather API lỗi 4xx/5xx hoặc timeout sau 5 giây (theo COMMON §5)
And    không có cache dữ liệu nào
Then   widget ẩn (không chiếm không gian layout)

Given  đang ở stale state và refetch thất bại
Then   stale spinner dừng, widget giữ dữ liệu cũ
And    hiện tooltip nhỏ khi hover: "Dữ liệu thời tiết có thể chưa được cập nhật"

Given  Weather Detail đang mở và refresh thất bại
Then   toast "Không thể cập nhật thời tiết, vui lòng thử lại"
And    toast tự đóng sau 4 giây
```

#### AC-B08 · Feature flag và visibility

```gherkin
Given  feature flag weather_widget = false
Then   widget không render (không chiếm không gian layout, không gọi API)

Given  người dùng đang ở trạng thái Place Detail Panel full-width (mobile)
Then   weather widget ẩn để không chồng lên panel
And    widget tự hiện lại khi panel đóng
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
