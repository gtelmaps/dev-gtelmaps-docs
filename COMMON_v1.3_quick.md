# COMMON v1.3 — Bản rút gọn

> Đây là bản tóm tắt để dev/QA nắm nhanh. Chi tiết đầy đủ → [COMMON_v1.3_supplemented.md](./COMMON_v1.3_supplemented.md)

---

## Auth

```http
Authorization: Bearer {api_key}
```

| Lỗi | HTTP | `status` |
|---|---|---|
| Thiếu key | 401 | `MISSING_API_KEY` |
| Key sai/hết hạn | 401 | `INVALID_API_KEY` |
| Không có quyền | 403 | `PERMISSION_DENIED` |
| Key bị disable | 403 | `API_KEY_DISABLED` |

---

## Quota

| Tier | QPS | Daily | Monthly |
|---|---|---|---|
| Free | 10 | 1.000 | 25.000 |
| Standard | 50 | 50.000 | 1.500.000 |
| Enterprise | Hợp đồng | Hợp đồng | Hợp đồng |

**Tính quota:** `200 OK` + `200 ZERO_RESULTS` + Cache HIT → tính. `4xx/5xx/401/403/429` → không tính.

| Lỗi | HTTP | `status` |
|---|---|---|
| Vượt QPS | 429 | `OVER_QUERY_LIMIT` + `Retry-After` |
| Hết quota | 429 | `QUOTA_EXCEEDED` + `X-Quota-Reset` |

---

## Response Contract

**Thành công / Không có kết quả:**
```json
{ "status": "OK|ZERO_RESULTS", "request_id": "<uuid-v4>", "results": [...] }
```

**Lỗi** (không có field `results`):
```json
{ "status": "ERROR_CODE", "error_message": "English description", "request_id": "<uuid-v4>" }
```

**Headers luôn có:** `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `X-Request-Id`, `X-Cache`, `X-Cache-TTL`

### Error Codes chung

| HTTP | `status` |
|---|---|
| 400 | `MISSING_PARAMETER` · `INVALID_REQUEST` · `INVALID_LANGUAGE` |
| 401 | `MISSING_API_KEY` · `INVALID_API_KEY` |
| 403 | `PERMISSION_DENIED` · `API_KEY_DISABLED` |
| 413 | `REQUEST_TOO_LARGE` (> 8 KB) |
| 429 | `OVER_QUERY_LIMIT` · `QUOTA_EXCEEDED` |
| 500 | `INTERNAL_ERROR` |
| 504 | `GATEWAY_TIMEOUT` |

---

## NFR

| Chỉ số | Giá trị |
|---|---|
| P95 (cache MISS) | ≤ 500ms (feature có thể override) |
| P95 (cache HIT) | ≤ 20ms |
| Timeout | **5 giây** (cả gateway + client-side) |
| Throughput | ≥ 100 RPS/feature |
| Uptime | ≥ 99.5% monthly |
| RTO | ≤ 15 phút |

---

## Cache

| Thông số | Giá trị |
|---|---|
| Layer | Redis Cluster (`maps-cache`) |
| TTL mặc định | 24h |
| Redis down | Bypass → `X-Cache: BYPASS` |

---

## Logging

**Fields bắt buộc:** `timestamp`, `api_key_id`, `endpoint`, `http_method`, `http_status`, `status`, `latency_ms`, `cache_hit`, `request_id`, `language`

**Privacy:** Tọa độ log ở 3dp. Không bao giờ log API key value.

---

## Security

- HTTPS only, không hỗ trợ HTTP
- API key qua header, không qua query string
- Input validation trước khi truyền backend
- Response không lộ stack trace / IP / service name
- CORS: chỉ origin đã đăng ký, cấm `*`
- OWASP ZAP bắt buộc trước production

---

## Alert chung

| Alert | Condition | Severity |
|---|---|---|
| Latency Warning | P95 > 700ms / 5min | 🟡 |
| Latency Critical | P95 > 1000ms / 5min | 🔴 |
| Error Rate | > 5% / 5min | 🔴 |
| Engine Down | Health fail > 30s | 🔴 |
| Cache Low | Hit ratio < 20% / 1h | 🟡 |
| Quota Abuse | Key > 10x avg / 1h | 🟡 |

> Feature có SLA chặt hơn **phải** override alert threshold.

---

## DoD chung

- [ ] ConfigMap + Secret deployed staging
- [ ] HPA configured + tested
- [ ] Dashboard + alerts active
- [ ] Runbook written
- [ ] Unit test ≥ 80%
- [ ] Integration test pass (auth, rate limit, quota, errors)
- [ ] Security test: no critical/high
- [ ] API docs on Developer Portal
- [ ] Contract test pass (headers + JSON schema)
- [ ] Sign-off: QC + PO + DevOps

---

## Versioning & Deprecation

- Patch: typo/format. Minor: additive. Major: breaking.
- Deprecation: min 90 ngày trước sunset. Flow: `active` → `deprecated` → `sunset`.
- Breaking change: migration note + impact assessment + test/SDK/docs update.
