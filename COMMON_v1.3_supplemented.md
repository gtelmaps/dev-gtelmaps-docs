# GTEL Maps Platform — Common Specification

> **Scope:** Áp dụng cho tất cả API features của GTEL Maps Platform  
> **Status:** `Normative Draft`  
> **Version:** 1.3
> **Last updated:** 2026-03-09 · Claude (supplemented from v1.2)  
> **Owner:** Platform Team  
> **Kế thừa bởi:**
> - [FEAT-RG-001] Reverse Geocoding
> - [FEAT-RG-002] Geocoding Batch
> - [FEAT-FG-001] Forward Geocoding
> - [FEAT-PS-001] Places Search / Nearby
> - [FEAT-DR-001] Directions / Routing API
> - [FEAT-SM-001] Static Maps / Tile API

> ⚠️ **Nguyên tắc sử dụng**
> - Feature **không** copy lại nội dung đã có trong file này.
> - Feature chỉ ghi **override**, **extension**, hoặc **decision đặc thù**.
> - Mọi feature phải khai báo: `COMMON version inherited`, `override list`, `owner`, `status`.
> - Khi COMMON thay đổi, feature kế thừa phải được review impact trước khi release.

---

## Mục lục

| # | Section | Người tham gia |
|---|---|---|
| 0 | [Governance & Inheritance Contract](#0-governance--inheritance-contract) | PO · BA · Architect |
| 1 | [Glossary chung](#1-glossary-chung) | BA · DES · QC |
| 2 | [Authentication & Authorization](#2-authentication--authorization) | Backend Dev · QA |
| 3 | [Rate Limiting & Quota](#3-rate-limiting--quota) | PO · Backend Dev · QA |
| 4 | [Business Rules — Quota & Coverage](#4-business-rules--quota--coverage) | PO · BA |
| 5 | [Non-Functional Requirements](#5-non-functional-requirements) | PO · BA · DevOps · QA |
| 6 | [Response Contract & Error Codes](#6-response-contract--error-codes) | Backend Dev · QA |
| 7 | [Caching Strategy](#7-caching-strategy) | Backend Dev · DevOps |
| 8 | [Logging & Usage Tracking](#8-logging--usage-tracking) | DevOps · BA |
| 9 | [Monitoring & Alerting chung](#9-monitoring--alerting-chung) | DevOps |
| 10 | [Security Standards](#10-security-standards) | Backend Dev · DevOps |
| 11 | [Definition of Done chung](#11-definition-of-done-chung) | PO · QC |
| 12 | [Changelog](#12-changelog) | All |

---

## 0. Governance & Inheritance Contract
`👤 PO · BA · Architect`

### Metadata bắt buộc trong mọi feature spec

| Field | Bắt buộc | Ví dụ |
|---|---|---|
| Feature ID | ✅ | `FEAT-RG-001` |
| COMMON version inherited | ✅ | `1.3` |
| Override sections | ✅ | `§5, §7, §8` |
| Owner | ✅ | `Platform Team / Geocoding Team` |
| Status | ✅ | `Draft`, `Ready for Build`, `Released` |
| Last reviewed date | ✅ | `2026-03-09` |

### Quy tắc thay đổi phiên bản

| Thay đổi | Tăng version | Ví dụ |
|---|---|---|
| Chỉnh câu chữ, format, typo | Patch (`1.0 → 1.0.1`) | Sửa chính tả, anchor, wording |
| Thêm rule không phá vỡ contract cũ | Minor (`1.0 → 1.1`) | Thêm guideline log, governance |
| Đổi response contract / error semantics / breaking policy | Major (`1.x → 2.0`) | Đổi JSON shape, đổi quota semantics |

### Quy tắc tương thích

- Thay đổi **additive** được phép trong minor version.
- Thay đổi làm feature cũ phải sửa code/test/docs được xem là **breaking change**.
- Feature chỉ được ghi `Ready for Build` khi:
  - đã pin `COMMON version inherited`
  - không còn mâu thuẫn với COMMON
  - mọi override đều được liệt kê tường minh

### Impact Review Checklist khi COMMON thay đổi

- [ ] Có thay đổi auth / permission scope?
- [ ] Có thay đổi response body / header?
- [ ] Có thay đổi error code hoặc HTTP mapping?
- [ ] Có thay đổi quota / billing semantics?
- [ ] Có thay đổi logging / privacy policy?
- [ ] Có cần update test cases / SDK / docs portal?


### Deprecation & Compatibility Policy

| Hạng mục | Chính sách |
|---|---|
| Backward-compatible change | Có thể phát hành trong minor version |
| Breaking change | Phải tăng major version + có migration note |
| Thời gian thông báo deprecate | Tối thiểu 90 ngày trước khi remove ở production public API |
| Kênh thông báo | Changelog, Developer Portal, release note, email cho khách hàng enterprise |
| Trạng thái endpoint / field | `active` → `deprecated` → `sunset` |
| Sunset date bắt buộc | Có với mọi field / endpoint bị deprecate |

**Quy tắc áp dụng**
- Không được xóa field / endpoint public mà không có giai đoạn `deprecated`.
- Feature spec phải ghi rõ field nào đang deprecated nếu còn dùng tạm.
- Mọi breaking change phải kèm:
  - migration note
  - impact assessment
  - test / SDK / docs update plan
- Nếu API chỉ dùng nội bộ nhưng ảnh hưởng nhiều team, vẫn phải có changelog và owner sign-off.

**Mẫu deprecation note**
```yaml
deprecated_item: results[*].legacy_name
deprecated_in: COMMON 1.2
sunset_target: 2026-09-01
replacement: results[*].formatted_address
impact: SDK + docs portal + contract test
owner: Platform Team
```


---

## 1. Glossary chung
`👤 BA · DES · QC`

> Các thuật ngữ dùng xuyên suốt toàn bộ GTEL Maps Platform. Feature-specific terms định nghĩa trong file feature tương ứng.

| Thuật ngữ | Định nghĩa |
|---|---|
| **API Key** | Mã xác thực được cấp cho developer để truy cập GTEL Maps API |
| **QPS** | Queries Per Second — giới hạn số request mỗi giây của một API key |
| **Quota** | Giới hạn tổng số request trong một khoảng thời gian (ngày / tháng) |
| **Cache HIT** | Request được phục vụ từ cache, không gọi tới backend engine |
| **Cache MISS** | Request không có trong cache, phải gọi backend engine |
| **Cache BYPASS** | Cache bị bỏ qua do Redis không khả dụng |
| **`request_id`** | UUID v4 duy nhất cho mỗi request, có mặt trong mọi response |
| **`status`** | Trạng thái kết quả của request: `OK`, `ZERO_RESULTS`, hoặc error code |
| **`place_id`** | Mã định danh duy nhất của một địa điểm trong hệ thống GTEL Maps |
| **Nominatim** | Geocoding engine mã nguồn mở, self-hosted trên hạ tầng GTEL |
| **Kong Gateway** | API Gateway xử lý auth, rate limiting, routing cho toàn bộ Maps API |
| **ĐVHC** | Đơn vị hành chính — hệ thống phân cấp địa lý hành chính của Việt Nam |
| **Tier** | Gói dịch vụ API: `Free`, `Standard`, `Enterprise` |
| **SLA** | Service Level Agreement — cam kết uptime và chất lượng dịch vụ |
| **P95 Latency** | 95th percentile latency — 95% request được phục vụ trong thời gian này |
| **RPS** | Requests Per Second — throughput của hệ thống |

---

## 2. Authentication & Authorization
`👤 Backend Dev · QA`

> **Áp dụng cho:** Tất cả API endpoints của GTEL Maps Platform.  
> Feature chỉ cần ghi `→ Kế thừa từ COMMON §2`, không viết lại.

### AC-CMN-AUTH · Xác thực API Key

| Trường hợp | HTTP | `status` |
|---|---|---|
| Không có API key trong request | 401 | `MISSING_API_KEY` |
| API key sai định dạng | 401 | `INVALID_API_KEY` |
| API key hết hạn | 401 | `INVALID_API_KEY` |
| API key không tồn tại trong hệ thống | 401 | `INVALID_API_KEY` |
| Key không có quyền truy cập endpoint này | 403 | `PERMISSION_DENIED` |
| Key bị vô hiệu hóa thủ công bởi admin | 403 | `API_KEY_DISABLED` |
| Key hợp lệ, đúng quyền | — | Tiếp tục xử lý request |

### Cách truyền API Key

```http
Authorization: Bearer {api_key}
```

> Không hỗ trợ truyền API key qua query param (`?key=...`) vì lý do bảo mật.

### Permission Scope

Mỗi feature có permission scope riêng. Feature phải khai báo scope cần thiết trong phần API Contract của mình.

| Feature | Required Scope |
|---|---|
| Reverse Geocoding | `geocoding.reverse` |
| Forward Geocoding | `geocoding.forward` |
| Geocoding Batch | `geocoding.batch` |
| Places Search | `places.search` |
| Directions | `directions.route` |
| Static Maps | `maps.static` |

---

## 3. Rate Limiting & Quota
`👤 PO · Backend Dev · QA`

> **Áp dụng cho:** Tất cả API endpoints.  
> Feature chỉ cần ghi `→ Kế thừa từ COMMON §3`, không viết lại.

### AC-CMN-RATELIMIT · Rate Limiting

| Trường hợp | HTTP | `status` | Header bổ sung |
|---|---|---|---|
| Vượt QPS limit của API key | 429 | `OVER_QUERY_LIMIT` | `Retry-After: {seconds}` |
| Vượt daily quota | 429 | `QUOTA_EXCEEDED` | `X-Quota-Reset: {unix_timestamp}` |
| Vượt monthly quota | 429 | `QUOTA_EXCEEDED` | `X-Quota-Reset: {unix_timestamp}` |

### Response Headers — luôn có mặt

Mọi response (kể cả lỗi) đều phải trả về:

| Header | Mô tả |
|---|---|
| `X-RateLimit-Limit` | QPS limit của API key |
| `X-RateLimit-Remaining` | Số request còn lại trong giây hiện tại |
| `X-RateLimit-Reset` | Unix timestamp giây tiếp theo reset |
| `X-Request-Id` | Giống `request_id` trong response body |

### Quy tắc billing / counting

- Quota được tính **per API key**, không phải per user hay per application.
- Tất cả quy tắc tính quota phải nhất quán giữa gateway, analytics pipeline, billing dashboard và docs portal.
- Nếu commercial policy khác giá trị mặc định bên dưới thì commercial policy thắng, nhưng **semantics tính quota không đổi**.

---

## 4. Business Rules — Quota & Coverage
`👤 PO · BA`

> Các con số quota dưới đây là **platform defaults cho V1**, có thể bị commercial policy override khi phát hành gói dịch vụ thực tế.

### BR-CMN-01 · Quota Tier mặc định

| Tier | QPS limit | Daily quota | Monthly quota |
|---|---|---|---|
| `Free` | 10 req/s | 1.000 req | 25.000 req |
| `Standard` | 50 req/s | 50.000 req | 1.500.000 req |
| `Enterprise` | Theo hợp đồng | Theo hợp đồng | Theo hợp đồng |

### BR-CMN-02 · Tính quota

| Trường hợp | Tính quota? |
|---|---|
| Request trả về `200 OK` với `status=OK` | ✅ Tính 1 unit |
| Request trả về `200 OK` với `status=ZERO_RESULTS` | ✅ Tính 1 unit |
| Request trả về lỗi `4xx` do lỗi client | ❌ Không tính |
| Request trả về lỗi `5xx` do lỗi server | ❌ Không tính |
| Cache HIT | ✅ Tính 1 unit |
| Request bị từ chối do auth (`401`, `403`) | ❌ Không tính |
| Request bị từ chối do rate limit (`429`) | ❌ Không tính |

### BR-CMN-03 · Vùng phục vụ

- Nền tảng GTEL Maps chỉ phục vụ dữ liệu trong lãnh thổ Việt Nam (bao gồm hải đảo có dữ liệu).
- Tọa độ / địa điểm ngoài Việt Nam **mặc định** trả về `ZERO_RESULTS`, không trả lỗi.
- Không hỗ trợ dữ liệu quốc tế trong V1.
- Nếu một feature cần khác chuẩn này thì phải khai báo override rõ ràng trong feature spec.

### BR-CMN-04 · Ngôn ngữ

- Mặc định `language=vi` nếu không truyền tham số.
- Hỗ trợ chuẩn platform: `vi`, `en`.
- Tên riêng không có bản dịch tiếng Anh → giữ nguyên tiếng Việt không dấu khi `language=en`.

---

## 5. Non-Functional Requirements
`👤 PO · BA · DevOps · QA`

> **Áp dụng cho:** Toàn bộ Maps API.  
> Feature có SLA khác → khai báo override trong section NFR của feature đó.

### Performance

| Chỉ số | Yêu cầu chung | Ghi chú |
|---|---|---|
| P95 Latency (cache MISS) | ≤ 500ms | Feature có thể override xuống thấp hơn |
| P95 Latency (cache HIT) | ≤ 20ms | Áp dụng toàn bộ |
| Timeout | 5 giây | API Gateway tự trả `504` sau 5s. **Client-side cũng nên dùng giá trị này làm chuẩn timeout mặc định.** |
| Throughput | ≥ 100 RPS per feature | Feature có thể override cao hơn |

### Availability

| Chỉ số | Yêu cầu |
|---|---|
| Uptime | ≥ 99.5% monthly |
| RTO | ≤ 15 phút |
| RPO | N/A — stateless API |

### Security

- [ ] API key không được lưu trong log dạng plain text — chỉ lưu `api_key_id`
- [ ] Input được sanitize trước khi truyền vào backend engine
- [ ] Response không được lộ thông tin nội bộ: stack trace, IP, service name, namespace
- [ ] HTTPS bắt buộc — không chấp nhận HTTP request

### Readiness rule cho mọi SLA override

- [ ] Có benchmark hoặc perf test chứng minh mục tiêu override là khả thi
- [ ] Có owner chịu trách nhiệm metric
- [ ] Có dashboard và alert cho metric override

---

## 6. Response Contract & Error Codes
`👤 Backend Dev · QA`

> **Áp dụng cho:** Tất cả API endpoints.  
> Feature chỉ cần ghi `→ Kế thừa từ COMMON §6`, không viết lại phần error codes chung.

### Cấu trúc Response chuẩn

**Thành công:**
```json
{
  "status": "OK",
  "request_id": "<uuid-v4>",
  "results": [ ... ]
}
```

**Không có kết quả:**
```json
{
  "status": "ZERO_RESULTS",
  "request_id": "<uuid-v4>",
  "results": []
}
```

**Lỗi:**
```json
{
  "status": "ERROR_CODE",
  "error_message": "Human-readable description for developers (English).",
  "request_id": "<uuid-v4>"
}
```

> **Lưu ý:** Response lỗi **không** chứa field `results`. Chỉ response `200 OK` và `200 ZERO_RESULTS` mới có `results`.

### Quy tắc Response

- [ ] `request_id` là UUID v4 duy nhất — có mặt trong **mọi** response kể cả lỗi
- [ ] `error_message` bằng tiếng Anh, đủ thông tin để developer tự debug
- [ ] Không lộ thông tin nội bộ trong response lỗi
- [ ] `HTTP 200` chỉ dùng cho `OK` và `ZERO_RESULTS`
- [ ] `4xx/5xx` phải đi kèm `status` là error code tương ứng
- [ ] Không trộn nhiều ngữ nghĩa trong cùng một `status`

### Error Codes chung

| HTTP | `status` | Mô tả | Trigger |
|---|---|---|---|
| `400` | `MISSING_PARAMETER` | Thiếu tham số bắt buộc | Client |
| `400` | `INVALID_REQUEST` | Tham số sai kiểu hoặc ngoài range | Client |
| `400` | `INVALID_LANGUAGE` | `language` không được hỗ trợ | Client |
| `401` | `MISSING_API_KEY` | Không có API key trong request | Client |
| `401` | `INVALID_API_KEY` | API key sai hoặc hết hạn | Client |
| `403` | `PERMISSION_DENIED` | Key không có quyền truy cập endpoint | Client |
| `403` | `API_KEY_DISABLED` | Key bị vô hiệu hóa | Client |
| `413` | `REQUEST_TOO_LARGE` | Request vượt quá giới hạn kích thước (8 KB) | Client |
| `429` | `OVER_QUERY_LIMIT` | Vượt QPS + `Retry-After` header | Client |
| `429` | `QUOTA_EXCEEDED` | Hết quota ngày/tháng | Client |
| `500` | `INTERNAL_ERROR` | Lỗi hệ thống | Server |
| `504` | `GATEWAY_TIMEOUT` | Timeout backend engine | Server |

### AC-CMN-RESPONSE · Acceptance Criteria — Response Contract

```gherkin
Given  bất kỳ request nào đến Maps API
When   response được trả về
Then   "request_id" có mặt và là UUID v4 hợp lệ
And    "status" khớp với HTTP status code semantics
And    không có field nào chứa stack trace, IP nội bộ, hoặc tên service

Given  request gây ra lỗi bất kỳ
When   response lỗi được trả về
Then   "error_message" có mặt và là chuỗi tiếng Anh không rỗng
And    "request_id" vẫn có mặt
```

---

## 7. Caching Strategy
`👤 Backend Dev · DevOps`

> **Áp dụng cho:** Tất cả API endpoints có cache.  
> Feature phải khai báo **cache key pattern** và **TTL** riêng trong Technical Notes.

### Nguyên tắc chung

| Thông số | Giá trị chung | Override |
|---|---|---|
| Cache layer | Redis Cluster (`maps-cache`) | Không override |
| TTL mặc định | 24h | Feature có thể rút ngắn |
| Invalidation | TTL-based + manual flush | Feature có thể thêm event-based |
| Fallback khi Redis down | Bypass cache, gọi thẳng backend | Không override |

### Cache Headers — luôn có mặt trong response

| Header | Giá trị | Ý nghĩa |
|---|---|---|
| `X-Cache` | `HIT` | Kết quả từ cache |
| `X-Cache` | `MISS` | Kết quả từ backend engine, đã lưu cache |
| `X-Cache` | `BYPASS` | Redis không khả dụng, bypass cache |
| `X-Cache-TTL` | `{seconds}` | Số giây còn lại của cache entry |

### AC-CMN-CACHE · Acceptance Criteria — Caching

```gherkin
Given  Redis down hoặc không khả dụng
When   request được gửi
Then   API vẫn trả về kết quả (gọi thẳng backend)
And    response header "X-Cache: BYPASS"
And    latency có thể cao hơn SLA cache HIT

Given  hai request giống hệt nhau trong TTL window
When   request thứ hai được gửi
Then   response header "X-Cache: HIT"
And    latency ≤ 20ms
```

---

## 8. Logging & Usage Tracking
`👤 DevOps · BA`

> **Áp dụng cho:** Tất cả API endpoints.  
> Feature có thể thêm fields log riêng, nhưng **không được bỏ** các fields bắt buộc dưới đây.

### Log Schema bắt buộc

| Field | Type | Mô tả | Privacy rule |
|---|---|---|---|
| `timestamp` | ISO 8601 | Thời điểm request | — |
| `api_key_id` | string | ID của API key (không phải key value) | Không log key value |
| `endpoint` | string | Tên endpoint | — |
| `http_method` | string | `GET`, `POST`... | — |
| `http_status` | int | HTTP status code trả về | — |
| `status` | string | `OK`, `ZERO_RESULTS`, error code | — |
| `latency_ms` | int | End-to-end latency tính từ gateway | — |
| `cache_hit` | boolean | `true` nếu `X-Cache: HIT` | — |
| `request_id` | uuid | UUID v4 của request | — |
| `language` | string | `vi` / `en` | — |

### BR-CMN-05 · Privacy trong Log

- Tọa độ địa lý: làm tròn đến **3 chữ số thập phân** (~111m) trước khi ghi log.
- API key value: **không bao giờ** được ghi vào log — chỉ ghi `api_key_id`.
- Thông tin cá nhân (tên, địa chỉ nhà riêng): không ghi vào log hệ thống.
- Nếu feature cần log dữ liệu nhạy cảm hơn chuẩn này thì phải có sign-off của PO + Legal.

### AC-CMN-LOG · Acceptance Criteria — Logging

- [ ] Mọi request đều tạo ra ít nhất 1 log entry với đầy đủ fields bắt buộc
- [ ] Log được đẩy vào analytics pipeline trong vòng **60 giây**
- [ ] API key value không xuất hiện trong log (chỉ `api_key_id`)
- [ ] Request `4xx` do lỗi client → log nhưng **không tính quota**
- [ ] Request `5xx` do lỗi server → log nhưng **không tính quota**

---

## 9. Monitoring & Alerting chung
`👤 DevOps`

> **Áp dụng cho:** Tất cả API endpoints.  
> Feature có thể bổ sung thêm alert/panel riêng, không được xóa phần chung.

### Dashboard chung (Grafana)

| Panel | Metric key | Mô tả |
|---|---|---|
| Request Rate | `maps_api_rps{endpoint}` | RPS per endpoint |
| P95 Latency | `maps_api_latency_p95{endpoint}` | P95 latency per endpoint |
| Error Rate | `maps_api_error_rate{endpoint}` | % request lỗi 4xx/5xx |
| Cache Hit Ratio | `maps_api_cache_hit_ratio{endpoint}` | % cache HIT |
| Quota Usage | `maps_api_quota_used{api_key_id}` | Usage per API key |

### Alert Rules chung

| Alert | Condition | Severity | Notify |
|---|---|---|---|
| High Latency (Warning) | P95 > 700ms trong 5 phút | 🟡 Warning | DevOps |
| High Latency (Critical) | P95 > 1000ms trong 5 phút | 🔴 Critical | DevOps on-call |
| High Error Rate | Error rate > 5% trong 5 phút | 🔴 Critical | DevOps on-call |
| Backend Engine Down | Health check fail > 30s | 🔴 Critical | DevOps on-call |
| Cache Hit Ratio Low | < 20% trong 1 giờ | 🟡 Warning | DevOps |
| Quota Abuse | Single key > 10x daily avg trong 1 giờ | 🟡 Warning | Platform Admin |

> **Lưu ý cho feature overrides:** Nếu feature có SLA chặt hơn (ví dụ P95 ≤ 300ms), feature **phải** override alert threshold tương ứng trong §XIX của mình. Alert COMMON là ngưỡng **trần nền tảng**, không phải ngưỡng SLA của từng feature.

---

## 10. Security Standards
`👤 Backend Dev · DevOps`

> Áp dụng toàn bộ, không override trừ khi có quyết định kiến trúc riêng.

- [ ] Tất cả API endpoints chỉ chấp nhận HTTPS
- [ ] API key truyền qua header `Authorization: Bearer`, không qua query string
- [ ] Input validation bắt buộc trước khi truyền vào bất kỳ backend engine nào
- [ ] Response không lộ internal info: stack trace, service name, K8s namespace, IP
- [ ] Rate limiting enforce tại API Gateway (Kong), không tại application layer
- [ ] Security test (OWASP ZAP) bắt buộc trước mọi release lên production
- [ ] CORS policy: chỉ cho phép origin đã đăng ký trong API key config; không dùng `Access-Control-Allow-Origin: *` cho production
- [ ] Request body / URL tổng cộng không vượt quá **8 KB**; request vượt quá → `413 Request Entity Too Large`

---

## 11. Definition of Done chung
`👤 PO · QC`

> Checklist này áp dụng cho **mọi** feature. Feature có thể bổ sung thêm DoD riêng, nhưng không được bỏ items dưới đây.

### Infrastructure & Operations

- [ ] ConfigMap và Secret đã deploy lên staging
- [ ] HPA đã cấu hình và test scale up/down
- [ ] Monitoring dashboard live với đầy đủ panels chung (§9)
- [ ] Alert rules chung đã active (§9)
- [ ] Runbook cho incident response đã viết

### Quality

- [ ] Unit test coverage ≥ 80% cho module mới
- [ ] Integration test pass: auth, rate limit, quota, error codes
- [ ] Security test: không có critical/high vulnerability
- [ ] API docs publish lên Developer Portal
- [ ] Contract test pass cho response headers và JSON schema

### Sign-off

- [ ] QC sign-off: UAT pass
- [ ] PO sign-off: business requirements verified
- [ ] DevOps sign-off: infra và monitoring sẵn sàng production
- [ ] Architect / Platform Owner sign-off nếu feature có override breaking hoặc override SLA lớn

---

## 12. Changelog

| Date | Version | Author | Changes |
|---|---|---|---|
| 2026-03-08 | 1.0 | @tung | Created — tách từ FEAT-RG-001 v1.2; áp dụng cho 6 features |
| 2026-03-09 | 1.1 | ChatGPT | Thêm governance contract, versioning policy, impact review checklist, readiness rule cho SLA override, siết semantics response/quota/privacy |
| 2026-03-09 | 1.2 | ChatGPT | Bổ sung Deprecation & Compatibility Policy để scale public/internal API an toàn hơn |
| 2026-03-09 | 1.3 | Claude | Thêm High Latency Warning alert, note client-side timeout, clarify error response không có `results`, thêm CORS policy, request size limit 8KB + `413`, note feature phải override alert khi SLA chặt hơn |
