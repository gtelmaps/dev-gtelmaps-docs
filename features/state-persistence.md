# State Persistence

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

State Persistence định nghĩa cơ chế lưu và khôi phục trạng thái của Maps Viewer giữa các session (reload trang, đóng/mở trình duyệt). Gồm hai kênh: URL (trạng thái bản đồ và feature đang active) và LocalStorage (preferences, lịch sử tìm kiếm, saved places cho guest users).

### Why

- **User need:** Khi quay lại Maps sau khi đóng tab, người dùng kỳ vọng thấy lại vị trí và trạng thái họ đã dùng lần trước — không phải bắt đầu lại từ đầu.
- **Business value:** Good session restore giảm time-to-value, tăng return session rate và stickiness.

---

# Part 02 - Specifications

## Backend

_Server-side persistence cho logged-in users (Saved Places, Preferences API). LocalStorage cho guest._

## UI/UX & Frontend

### Persistence Channels

| Channel        | Scope                     | TTL             | Encryption |
| -------------- | ------------------------- | --------------- | ---------- |
| URL params     | Bản đồ + feature state    | Session (URL)   | No         |
| LocalStorage   | User prefs, history       | 90 ngày         | No         |
| SessionStorage | Ephemeral UI state        | Tab session     | No         |
| Cookie         | Locale, consent           | 1 năm           | No         |
| Server (API)   | Saved Places (logged-in)  | Permanent       | Yes        |

### Persisted State Inventory

| State                   | Channel        | Key                      |
| ----------------------- | -------------- | ------------------------ |
| Map center + zoom       | URL            | `@lat,lng,zoom`          |
| Map tilt + heading      | URL            | `,tilt_a,heading_y`      |
| Active map layer        | URL + LS       | `?layer=`, `maps_layer`  |
| Theme preference        | LocalStorage   | `maps_theme`             |
| Locale preference       | Cookie         | `maps_locale`            |
| Recent searches         | LocalStorage   | `maps_recent_searches`   |
| Saved places (guest)    | LocalStorage   | `maps_saved_places`      |
| Cookie consent          | Cookie         | `maps_consent`           |
| Last map position       | LocalStorage   | `maps_last_position`     |

### Triggers & Entry Points

| ID  | Trigger                                    | AC  |
| --- | ------------------------------------------ | --- |
| T01 | State thay đổi → persist xuống storage     | B01 |
| T02 | App load → hydrate state từ storage        | B02 |
| T03 | User đăng nhập → merge guest state với server | B03 |
| T04 | Storage gần đầy hoặc corrupted             | B04 |

### Acceptance Criteria

`Implement: Frontend Dev | Review: QA | Sign-off: QC`

#### AC-B01 · Write to storage (T01)

```gherkin
Given  người dùng thay đổi theme sang dark
Then   LocalStorage.setItem("maps_theme", "dark") được gọi
And    write debounce 500ms để tránh write quá nhiều lần khi user pan bản đồ liên tục

Given  người dùng thực hiện search
Then   query được prepend vào maps_recent_searches array
And    array giới hạn 20 items, xóa item cũ khi vượt quá
```

#### AC-B02 · Read từ storage khi load (T02)

```gherkin
Given  LocalStorage có maps_last_position = {lat:10.77, lng:106.70, zoom:14}
And    URL không có coordinate params
Then   bản đồ load tại tọa độ saved

Given  URL có coordinate params
Then   URL params được ưu tiên hơn LocalStorage
And    LocalStorage cập nhật với vị trí mới từ URL

Given  LocalStorage data bị corrupt (JSON parse error)
Then   corrupt key bị xóa và reset về default
And    không crash app
```

#### AC-B03 · Guest → Logged-in merge (T03)

```gherkin
Given  user là guest, có saved places trong LocalStorage
When   user đăng nhập
Then   guest saved places được merge vào server account
And    LocalStorage saved places được xóa sau khi sync thành công
And    duplicates được loại bỏ trong quá trình merge
```

#### AC-B04 · Storage quota exceeded (T04)

```gherkin
Given  LocalStorage đầy (5MB limit)
When   app cố gắng write mới
Then   app bắt QuotaExceededError
And    xóa oldest recent searches trước để giải phóng space
And    retry write
And    nếu vẫn thất bại → log error về monitoring, tiếp tục hoạt động không persist
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
