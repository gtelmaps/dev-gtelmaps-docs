# App State Store

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

App State Store là tầng quản lý trạng thái toàn cục của Maps Viewer, bao gồm: trạng thái bản đồ (center, zoom, tilt, heading, layers), trạng thái UI (panel đang mở, selected place), và dữ liệu ephemeral (search results, nearby results). Store là source of truth duy nhất, đồng bộ hai chiều với URL và LocalStorage.

### Why

- **User need:** Trạng thái ứng dụng phải nhất quán khi người dùng navigate, back/forward, hoặc mở nhiều tab — không có inconsistency.
- **Business value:** Centralized state management giảm bugs, giúp feature development nhanh hơn, và là prerequisite cho Time Travel debugging và A/B testing.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                     | So sánh                                              |
| -- | -------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Single source of truth           | Centralized store đồng bộ 2 chiều với URL + LocalStorage — không inconsistency             | Google Maps internal, không rõ kiến trúc             |
| 2  | Time Travel debugging            | Kiến trúc store hỗ trợ replay state changes — debug và A/B testing dễ dàng                 | Không có tương đương public trên Google Maps         |
| 3  | Multi-tab safe                   | State nhất quán khi mở nhiều tab — không conflict giữa các instance                       | Google Maps tương đương                              |

---

# Part 02 - Specifications

## Backend

_Không có backend — state hoàn toàn client-side. Sync lên server khi user đăng nhập (Saved Places, Preferences)._

## UI/UX & Frontend

### State Shape (Top Level)

```typescript
interface AppState {
  map: MapState;         // center, zoom, tilt, heading, mapType, activeLayers
  ui: UIState;           // activePanelId, isSearchOpen, isSidebarOpen
  search: SearchState;   // query, results, selectedPlaceId, nearbyResults
  user: UserState;       // preferences, savedPlaces, recentSearches
  session: SessionState; // locationPermission, cookieConsent
}
```

### State Domains

| Domain    | Owned by      | Persisted to    | Synced to URL |
| --------- | ------------- | --------------- | ------------- |
| `map`     | Map engine    | LocalStorage    | Yes (/@lat,lng,zoom) |
| `ui`      | UI layer      | Session only    | Partial (/place/, /search/) |
| `search`  | Search feature| Session only    | Yes (/search/query) |
| `user`    | User service  | LocalStorage + Server | No   |
| `session` | App lifecycle | LocalStorage    | No            |

### Triggers & Entry Points

**Entry Points (Điểm bắt đầu):**
- **Component Action:** Bất kỳ dispatch event / cập nhật trạng thái nào từ giao diện.
- **Browser/URL Navigation:** Sự kiện thay đổi lịch sử trình duyệt.
- **Map Engine:** Các sự kiện từ thư viện hiển thị bản đồ (moveend, zoomend...).
- **App Initialization:** Quá trình khôi phục trạng thái (Hydration) ở bước khởi tạo ứng dụng.

**Triggers (Hành động kích hoạt cụ thể):**

| ID  | Trigger                                    | AC  |
| --- | ------------------------------------------ | --- |
| T01 | Action dispatch từ bất kỳ component        | B01 |
| T02 | URL change (browser navigation)            | B02 |
| T03 | Map engine event (moveend, zoomend...)     | B03 |
| T04 | App hydration từ LocalStorage khi load     | B04 |

### Acceptance Criteria

`Implement: Frontend Dev | Review: QA | Sign-off: QC`

#### AC-B01 · Unidirectional data flow (T01)

```gherkin
Given  một component dispatch action "SET_ACTIVE_PLACE"
Then   store cập nhật state.ui.selectedPlaceId
And    tất cả components subscribe vào state đó re-render với giá trị mới
And    không có side effect ngoài selector subscription
And    action được log trong dev tools (Redux DevTools hoặc tương đương)
```

#### AC-B02 · URL ↔ State sync (T02)

```gherkin
Given  URL thay đổi (browser back/forward hoặc navigate)
Then   store parse URL và dispatch actions để restore đúng state
And    bản đồ, panel, và search cập nhật theo

Given  state thay đổi (pan/zoom/open panel)
Then   URL cập nhật via History.replaceState / pushState (debounce 300ms)
```

#### AC-B03 · Map engine → Store sync (T03)

```gherkin
Given  người dùng pan/zoom bản đồ (map engine event)
Then   store.map.center và store.map.zoom cập nhật
And    các component depend on map state re-render (Weather Widget, Scalebar...)
```

#### AC-B04 · Hydration từ LocalStorage (T04)

```gherkin
Given  app load lần đầu
Then   store hydrate từ LocalStorage (user preferences, recent searches, saved places)
And    conflicts giữa URL params và LocalStorage → URL params ưu tiên
And    corrupted LocalStorage data → reset về default state (không crash)
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
