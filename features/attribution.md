# Attribution

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

Attribution là thành phần hiển thị thông tin bản quyền dữ liệu bản đồ (data providers, tile sources, imagery credits) ở góc dưới bản đồ. Nội dung attribution cập nhật động theo layer đang hiển thị và vùng địa lý đang xem.

### Why

- **Legal need:** Hợp đồng với các nhà cung cấp dữ liệu (OSM, imagery providers, traffic data) bắt buộc phải hiển thị credit đúng cách. Vi phạm có thể dẫn đến rủi ro pháp lý.
- **User need:** Người dùng có thể xem nguồn gốc dữ liệu bản đồ để đánh giá độ tin cậy.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                     | So sánh                                              |
| -- | -------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Dynamic theo layer + vùng        | Attribution cập nhật tự động khi chuyển layer hoặc pan sang vùng có data provider khác     | Google Maps hiển thị attribution cố định             |
| 2  | Transparent data sourcing        | Người dùng biết chính xác nguồn dữ liệu đang xem — tăng trust                            | Google Maps chỉ hiển thị "Google" + "Map data ©"     |
| 3  | White-label ready                | Enterprise có thể customize attribution theo hợp đồng                                     | Google Maps không cho phép thay đổi attribution      |

---

# Part 02 - Specifications

## Backend

_Attribution text được nhúng trong tile metadata hoặc layer config từ backend._

## UI/UX & Frontend

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Sự kiện Bản đồ (Map Events):** Quá trình tải lần đầu, người dùng di chuyển sang khu vực mới hoặc thay đổi loại bản đồ.
- **Tương tác UI:** Click vào thông tin bản quyền trên Attribution text.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                      | Nền tảng    | Input                        | AC  |
| --- | -------------------------------------------- | ----------- | ---------------------------- | --- |
| T01 | Bản đồ load lần đầu                          | Web, Mobile | Layer config + tile region   | B01 |
| T02 | Layer thay đổi (Standard → Satellite...)     | Web, Mobile | Layer mới                    | B02 |
| T03 | Bản đồ pan sang vùng địa lý có provider khác | Web, Mobile | Tile region mới              | B02 |
| T04 | Click [©] hoặc link trong attribution        | Web         | —                            | B03 |

### States Inventory

| State     | Mô tả                               | Component                       |
| --------- | ----------------------------------- | ------------------------------- |
| `visible` | Bản đồ đang hiển thị                | Attribution text ở bottom-right |
| `collapsed` | Mobile — thu gọn, chỉ hiện [©]   | Icon [©] expandable             |

### Components, Responsive & Typography

#### Component Inventory

| Component                        | Dùng trong        |
| -------------------------------- | ----------------- |
| Attribution text (inline HTML)   | Desktop, tablet   |
| Collapsed [©] icon               | Mobile            |
| Expandable attribution tooltip   | Mobile (on tap)   |

#### Responsive Behavior

| Breakpoint                 | Layout                              |
| -------------------------- | ----------------------------------- |
| Mobile portrait (< 768px)  | Thu gọn thành [©], tap để mở rộng  |
| Mobile landscape (< 768px) | Text rút gọn (max 1 dòng)          |
| Tablet (768px–1024px)      | Text đầy đủ                        |
| Desktop (> 1024px)         | Text đầy đủ                        |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Hiển thị attribution khi load (T01)

```gherkin
Given  bản đồ load xong
Then   attribution text hiển thị đúng vị trí bottom-right
And    text bao gồm tất cả providers của layers đang active
And    text không che các widget khác (z-index thấp nhất trong overlay group)
```

#### AC-B02 · Cập nhật khi layer / vùng thay đổi (T02, T03)

```gherkin
Given  người dùng bật layer Satellite
Then   attribution cập nhật thêm credit của imagery provider

Given  người dùng pan sang vùng có data provider khác
Then   attribution cập nhật đúng provider cho vùng đó
And    transition mượt (fade in/out 150ms)
```

#### AC-B03 · Click link trong attribution (T04)

```gherkin
Given  attribution chứa link (ví dụ: "© OpenStreetMap contributors")
When   người dùng click link
Then   mở URL đích trong tab mới
```

#### AC-B04 · Mobile collapsed state

```gherkin
Given  viewport là mobile portrait
Then   attribution thu gọn thành icon [©]
And    tap vào [©] mở tooltip với full attribution text
And    tap ra ngoài tooltip để đóng
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
