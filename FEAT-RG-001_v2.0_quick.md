# [FEAT-RG-001] Reverse Geocoding — Bản rút gọn

> COMMON inherited: **v1.4** → [COMMON_v1.4_quick.md](./COMMON_v1.4_quick.md)
> Chi tiết đầy đủ → [FEAT-RG-001_v2.0_supplemented.md](./FEAT-RG-001_v2.0_supplemented.md)
> Override: §5 (NFR), §6 (Error extend), §7 (Cache), §8 (Log extend), §9 (Alert)

---

## Quyết định V1

| Chủ đề | Quyết định |
|---|---|
| Result | **Single only** — `results[]` length `0..1` |
| Ngoài VN | `200 ZERO_RESULTS` |
| Cache precision | **3dp** (~111m) |
| Search Bar | Chỉ **decimal degrees** |
| Deep link zoom | Mặc định `15` |
| Fallback | **Không có** — Nominatim down → `504` |
| Quota | Platform defaults từ COMMON |

---

# Part 1 — Bối cảnh

## User Stories (tóm tắt)

| ID | Role | Goal |
|---|---|---|
| US-01 | Developer | Gửi tọa độ → nhận địa chỉ có cấu trúc |
| US-02 | Developer | Lọc theo cấp hành chính |
| US-03 | Developer | Nhận địa chỉ bằng `vi` hoặc `en` |
| US-04 | End User | Click bản đồ → thấy địa chỉ |
| US-05 | End User (Mobile) | Long press → xem địa chỉ |
| US-06 | End User | Mở link có tọa độ → địa chỉ tự động |
| US-07 | Platform Admin | Xem usage per API key |

---

# Part 2 — Đặc tả

## API Contract (§VII)

```http
GET /v1/geocode/reverse
Authorization: Bearer {api_key}
```

**Scope required:** `geocoding.reverse`

| Param | Type | Required | Default |
|---|---|---|---|
| `lat` | float | ✅ | — |
| `lng` | float | ✅ | — |
| `result_type` | string | ❌ | all |
| `language` | string | ❌ | `vi` |

`result_type` ∈ `{street_address, ward, district, province, country}`
`language` ∈ `{vi, en}`

### Response 200 OK

```json
{
  "status": "OK",
  "request_id": "uuid-v4",
  "results": [{
    "formatted_address": "136 Nguyễn Huệ, Phường Bến Nghé, Quận 1, Hồ Chí Minh, Việt Nam",
    "place_id": "gtel_place_abc123xyz",
    "address_components": [
      { "long_name": "136", "short_name": "136", "types": ["street_number"] },
      { "long_name": "Nguyễn Huệ", "short_name": "Nguyễn Huệ", "types": ["route"] },
      { "long_name": "Phường Bến Nghé", "short_name": "Bến Nghé", "types": ["sublocality_level_1"] },
      { "long_name": "Quận 1", "short_name": "Q.1", "types": ["administrative_area_level_2"] },
      { "long_name": "Hồ Chí Minh", "short_name": "TP.HCM", "types": ["administrative_area_level_1"] },
      { "long_name": "Việt Nam", "short_name": "VN", "types": ["country"] }
    ],
    "geometry": { "location": { "lat": 10.7769, "lng": 106.7009 }, "location_type": "ROOFTOP" },
    "types": ["street_address"]
  }]
}
```

`location_type` ∈ `{ROOFTOP, RANGE_INTERPOLATED, GEOMETRIC_CENTER, APPROXIMATE}`
Ưu tiên: `ROOFTOP > RANGE_INTERPOLATED > GEOMETRIC_CENTER > APPROXIMATE`

### address_components bắt buộc

| `types` | Ý nghĩa | Bắt buộc |
|---|---|---|
| `sublocality_level_1` | Phường/Xã | ✅ |
| `administrative_area_level_2` | Quận/Huyện | ✅ |
| `administrative_area_level_1` | Tỉnh/TP | ✅ |
| `country` | VN | ✅ |
| `street_number`, `route`, `postal_code` | | ❌ nếu có data |

### Error Codes (COMMON + Feature-specific)

| HTTP | `status` | Ghi chú |
|---|---|---|
| 400 | `MISSING_PARAMETER` | Thiếu lat/lng |
| 400 | `INVALID_REQUEST` | Sai kiểu/range |
| 400 | `INVALID_LANGUAGE` | Không phải vi/en |
| 400 | `INVALID_RESULT_TYPE` | ⭐ Feature-specific |
| 504 | `GATEWAY_TIMEOUT` | Nominatim down |

> Còn lại: xem COMMON (401, 403, 413, 429, 500)

### Mock Responses (§7.4)

Frontend dùng mock để build song song với Backend:

- **Mock 1 — Happy Path:** `status: "OK"`, 1 result đầy đủ address_components
- **Mock 2 — ZERO_RESULTS:** `status: "ZERO_RESULTS"`, `results: []`
- **Mock 3 — Error:** `status: "INVALID_REQUEST"`, không có field `results`

> Chi tiết JSON → xem §7.4 bản đầy đủ.

---

## UI/UX & Frontend (§VIII)

### 8 Triggers

| ID | Trigger | Platform | AC riêng |
|---|---|---|---|
| T01 | Click / tap bản đồ | Web + Mobile | B01 |
| T02 | Right-click → "Đây là đâu?" | Web | B08 |
| T03 | Long press ≥ 500ms | Mobile | B09 |
| T04 | Drag & drop marker | Web + Mobile | B10 |
| T05 | Nhập tọa độ Search Bar | Web + Mobile | B11 |
| T06 | Deep link URL | Web + Mobile | B12 |
| T07 | GPS "Vị trí của tôi" | Mobile + Web | B13 |
| T08 | API call trực tiếp | API | A01–A10 |

> T01–T07 đều gọi cùng API → AC nhóm A + B02–B07 áp dụng tất cả.

### UI States

| State | Component | Ghi chú |
|---|---|---|
| idle | Không có gì | |
| loading | Popup/Sheet + spinner | Sau 150ms |
| success | Popup/Sheet + actions | |
| zero_results | Empty message | Giữ marker |
| error | Toast 4s | |
| gps_denied | Toast | |
| gps_low_accuracy | Marker + badge | > 100m |
| dragging | Popup đóng | |
| disabled | Nút disabled + tooltip | GPS unavailable |

**Responsive:** < 768px portrait → Bottom Sheet. Còn lại → Popup. Font: Be Vietnam Pro.

### Design Asset Status (§8.3)

| Component / State | Status |
|---|---|
| Popup — success / loading / zero_results | ⬜ Chờ link |
| Bottom Sheet — all states | ⬜ Chờ link |
| Context Menu — "Đây là đâu?" | ⬜ Chờ link |
| GPS accuracy badge | ⬜ Chờ link |
| Toast — error / gps_denied | ⬜ Chờ link |
| Inline error — Search Bar | ⬜ Chờ link |

> DES cần fill Figma link + đổi status → ✅ Ready trước khi Frontend build.

---

## Acceptance Criteria — Tóm tắt

### Nhóm A — API

| AC | Tóm tắt | Input/Condition |
|---|---|---|
| A01 | Happy path | lat=10.7769, lng=106.7009 → 200 OK + address |
| A02 | address_components đầy đủ | Mỗi phần tử có long_name, short_name, types[] |
| A03 | Kết quả gần đúng | Không khớp chính xác → trả cấp gần nhất |
| A04 | result_type filter | result_type=district → types chứa district |
| A05 | language | en → English. Không truyền → vi. jp → 400 |
| A06 | ZERO_RESULTS | Ngoài VN hoặc không có data → 200 ZERO_RESULTS |
| A07 | Validation | Thiếu lat → 400. lat=91 → 400. lat=abc → 400. (0,0) → ZERO_RESULTS |
| A08 | Single-result | results.length ≤ 1 mọi lúc |
| A09 | Response nhất quán | Cùng input → cùng shape |
| A10 | Upstream failure | Nominatim down → 504, không trả partial |

### Nhóm B — UI

| AC | Tóm tắt |
|---|---|
| B01 | Click/tap → lấy lat/lng + gọi API |
| B02 | Marker xuất hiện ≤ 100ms, chỉ 1 marker |
| B03 | Loading sau 150ms: "Đang tìm địa chỉ..." |
| B04 | OK → popup hiện formatted_address + "Sao chép" + "Chỉ đường" |
| B05 | ZERO_RESULTS → "Không tìm thấy địa chỉ cho vị trí này", giữ marker |
| B06 | Error/timeout 5s → toast 4s: "Không thể tải địa chỉ, vui lòng thử lại" |
| B07 | Marker ≤ 100ms, popup ≤ 500ms, CLS < 0.1 |
| B08 | Right-click → "Đây là đâu?" → đóng menu + marker + API |
| B09 | Long press ≥ 500ms → haptic + marker + bottom sheet. < 500ms → không kích hoạt |
| B10 | Drag marker → đóng popup → thả → API mới → popup mới |
| B11 | "10.7769, 106.7009" + Enter → di chuyển + API. Sai format → inline error |
| B12 | `/map?lat=X&lng=Y[&zoom=Z]` → auto load. Sai param → toast lỗi |
| B13 | GPS denied → toast. OK → marker + API. Accuracy > 100m → badge. Không có GPS → nút disabled + tooltip |

---

## NFR Override (§IX)

| Chỉ số | COMMON | **RG Override** |
|---|---|---|
| P95 (cache MISS) | ≤ 500ms | **≤ 300ms** |
| Throughput | ≥ 100 RPS | **≥ 500 RPS** |
| Marker sau trigger | — | **≤ 100ms** |
| Popup render sau response | — | **≤ 500ms** |
| Loading indicator | — | Hiển thị sau **> 150ms** |

---

# Part 3 — Vận hành

## Technical Stack (§XI)

| Component | Tech | Namespace |
|---|---|---|
| Geocoding Engine | Nominatim (self-hosted) | `maps-geocoding` |
| Cache | Redis Cluster | `maps-cache` |
| Gateway | Kong | `maps-gateway` |
| Usage Tracking | Kafka → ClickHouse | `maps-analytics` |

**Nominatim mapping:**
```
GET /v1/geocode/reverse?lat=X&lng=Y&language=vi
  → GET /reverse?lat=X&lon=Y&format=jsonv2&addressdetails=1&accept-language=vi
```

**Cache:** key = `rgeo:{lat_3dp}:{lng_3dp}:{result_type}:{language}`, TTL = 24h

**K8s:** HPA min=2, max=10, CPU 70%. Baseline: 500m CPU, 2Gi RAM.

### Data Readiness Checklist (§XI)

| Data Source | Owner | Trạng thái | Deadline |
|---|---|---|---|
| OSM Vietnam import vào Nominatim | DevOps | ⬜ Pending | Trước staging |
| GTEL Admin DB sync + mapping layer | GIS + Backend | ⬜ Pending | Trước staging |
| Postcode DB loaded | GIS | ⬜ Pending | Trước staging |
| Data quality test 100 tọa độ mẫu | QA + GIS | ⬜ Pending | Trước UAT |

---

## Alert Override (§XII)

| Alert | Condition | Severity |
|---|---|---|
| Latency Warning | P95 > 350ms / 5min | 🟡 |
| Latency Critical | P95 > 500ms / 5min | 🔴 |
| ZERO_RESULTS | > 20% / 1h | 🟡 |
| Nominatim Down | Health fail > 30s | 🔴 |

---

## Analytics Events (§XIII)

| Event | Khi nào |
|---|---|
| `rgeo_triggered` | T01–T07 kích hoạt |
| `rgeo_api_called` | UI gọi API |
| `rgeo_success` | status=OK |
| `rgeo_zero_results` | ZERO_RESULTS |
| `rgeo_error` | 4xx/5xx/timeout |
| `rgeo_copy_address_clicked` | Nhấn Sao chép |
| `rgeo_directions_clicked` | Nhấn Chỉ đường |
| `rgeo_gps_permission_denied` | Từ chối GPS |
| `rgeo_searchbar_invalid_input` | Nhập sai format |

> Không gửi formatted_address. Tọa độ tối đa 3dp. Không block UI.

---

# Part 4 — Chất lượng & Phát hành

## Team Dependency & Handoff (§XIV)

| Phase | Output | Owner | Blocked by |
|---|---|---|---|
| P0 | API Contract finalized | BA + Backend Lead | — |
| P1 | Mock API ready | Backend Dev | P0 |
| P2 | Design assets ready | DES | P0 |
| P3 | Infra staging ready | DevOps | Data readiness |
| P4 | Backend API on staging | Backend Dev | P3 |
| P5 | Frontend UI on staging (mock) | Frontend Dev | P1, P2 |
| P6 | Integration FE ↔ real API | Frontend + Backend | P4, P5 |
| P7 | UAT | QA / QC | P6 |
| P8 | Perf test (500 RPS, P95 ≤ 300ms) | DevOps | P4 |

```text
Sprint 0:    [BA: Contract] [Backend: Mock API] [DES: Figma]
Sprint 1:    [Backend: Real API ────] [Frontend: UI + Mock ────] [DevOps: Infra]
Sprint 2:    [Integration ──] [Perf test] [UAT ────]
Sprint 2–3:  [Bug fix] [Sign-off] [Rollout prep]
```

---

## Business Rules tóm tắt

| BR | Rule |
|---|---|
| RG-01 | Ưu tiên: ROOFTOP > RANGE_INTERPOLATED > GEOMETRIC_CENTER > APPROXIMATE |
| RG-02 | V1: 0 hoặc 1 result |
| RG-03 | Ngoài VN → ZERO_RESULTS |
| RG-04 | Cache key: 3dp (~111m) |
| RG-05 | Search Bar: chỉ decimal degrees |
| RG-06 | Deep link: zoom mặc định 15. Param sai → default view + toast |
| RG-07 | Không có fallback. Nominatim down → 504 |
| RG-08 | GPS accuracy > 100m → badge. Không có GPS → nút disabled + tooltip |

---

## Rollout (§XVII)

**P0** Staging 100% → **P1** Internal 10% (24h no critical) → **P2** Internal 50% (stable) → **P3** Prod 100%

**Rollback:** P95 vượt 2 kỳ, error > 5%/5min, Nominatim down dài, data sai hàng loạt.

---

## Test Data (§XV)

| ID | lat | lng | Expected |
|---|---|---|---|
| TD-01 | 10.7769 | 106.7009 | OK, ROOFTOP (HCM) |
| TD-02 | 21.0285 | 105.8542 | OK, ROOFTOP (HN) |
| TD-03 | 16.0678 | 108.2208 | OK, ROOFTOP (ĐN) |
| TD-04 | 10.2500 | 106.3750 | OK, GEOMETRIC_CENTER (Long An) |
| TD-05 | 22.3964 | 103.8436 | OK, APPROXIMATE (Tây Bắc) |
| TD-06 | 8.6700 | 106.6200 | ZERO_RESULTS (biển Cà Mau) |
| TD-07 | 15.8800 | 108.3350 | ZERO_RESULTS (Cù Lao Chàm) |
| TD-08 | 48.8566 | 2.3522 | ZERO_RESULTS (Paris) |
| TD-09 | 0.0000 | 0.0000 | ZERO_RESULTS (Null island) |
| TD-10 | 10.7769 | 106.7009 | X-Cache: HIT (lần 2) |

---

# Part 5 — Tham khảo

## Open Items (không chặn build)

1. DMS cho Search Bar? → Sau UAT V1
2. Fleet tracking cache precision? → Khi có use case
3. Secondary geocoder fallback? → Trước gói Enterprise
4. Privacy 3dp legal review? → **Trước staging**
