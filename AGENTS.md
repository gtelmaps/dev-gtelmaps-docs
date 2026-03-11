# AGENTS.md — GTEL Maps Documentation

## Dự án này là gì

Đây là **repo tài liệu đặc tả** (feature specs) cho **GTEL Maps** — ứng dụng bản đồ web/mobile của GTEL, phục vụ thị trường Việt Nam. Repo KHÔNG chứa source code, chỉ chứa tài liệu để dev/QA/PO dùng khi xây dựng features.

---

## Cấu trúc thư mục

```
├── AGENTS.md              ← File này
├── README.md              ← Danh sách features và trạng thái ([x] done, [ ] todo, [-] in progress)
├── competitors.md         ← Danh sách đối thủ cạnh tranh (VN + quốc tế)
├── features/
│   ├── common.md          ← ⚠️ QUAN TRỌNG: Shared specs, NFR, error codes, glossary
│   ├── template.md        ← Template chuẩn cho feature spec mới
│   ├── reverse-geocoding.md
│   ├── unified-search-bar.md
│   └── ...                ← Mỗi feature một file .md
└── templates/
    ├── COMMON_v1.4_quick.md          ← Bản rút gọn COMMON cho dev/QA
    └── COMMON_v1.4_supplemented.md   ← Bản đầy đủ COMMON (backend specs)
```

---

## Quy tắc khi viết Feature Spec

### 1. Luôn đọc trước khi viết

Trước khi tạo hoặc chỉnh sửa bất kỳ feature spec nào, **BẮT BUỘC** đọc:

1. `features/common.md` — Shared specs (breakpoints, debounce, animation, z-index, a11y, error codes, glossary)
2. `features/template.md` — Cấu trúc 5 Parts chuẩn
3. `templates/COMMON_v1.4_quick.md` — Backend contract chung (auth, quota, response format, NFR, cache, logging, security, DoD)
4. Các feature liên quan (nếu có dependency)

### 2. Cấu trúc 5 Parts

Mỗi feature spec tuân theo cấu trúc:

| Part | Nội dung | Ai viết |
|------|----------|---------|
| **Part 01 — Background** | What, Why, Out of Scope, User Stories | PO, BA |
| **Part 02 — Specifications** | Backend API, UI/UX, Triggers, States, AC (Gherkin), Flow | BA, Dev, DES |
| **Part 03 — Operations** | Runbook, monitoring, alerting | DevOps, Backend |
| **Part 04 — Quality & Release** | Test plan, release checklist, rollout | QA, QC |
| **Part 05 — References** | Figma links, ADRs, API docs, changelog | All |

### 3. Phần "What" — viết đơn giản, dễ hiểu

- 1–2 câu mô tả feature làm gì
- Tập trung vào OUTPUT / hành vi người dùng thấy
- KHÔNG gom nhiều tính năng vào một mô tả
- Viết cho người không có context kỹ thuật cũng hiểu được

### 4. Kế thừa từ COMMON

- Mọi feature mặc định kế thừa toàn bộ COMMON v1.4
- Dùng ký hiệu `(theo COMMON §N)` để tham chiếu shared spec
- Nếu cần thay đổi giá trị COMMON, khai báo rõ trong header: `COMMON sections overridden: §5 (NFR extend)`
- Xem bảng Override Pattern trong `COMMON_v1.4_quick.md` để biết section nào được override/extend

### 5. Acceptance Criteria

- Viết bằng **Gherkin** (Given/When/Then)
- Mỗi AC gắn với trigger cụ thể qua ID: `AC-B01 · Tên AC (T01)`
- Luôn có AC cho **error state** và **edge cases**
- Error state mặc định: timeout 5 giây (theo COMMON §5), toast tự đóng sau 4 giây (theo COMMON §6)

### 6. Triggers & Entry Points

Mỗi feature phải khai báo bảng triggers:

```markdown
| ID  | Trigger              | Nền tảng    | Input          | AC  |
| --- | -------------------- | ----------- | -------------- | --- |
| T01 | [Hành động trigger]  | Web, Mobile | [Input data]   | B01 |
```

### 7. States Inventory

Mỗi feature phải liệt kê đầy đủ states:

- `idle` — Trạng thái mặc định
- `loading` — Đang xử lý (áp dụng skeleton threshold 150ms theo COMMON §4)
- `success` — Thành công
- `zero_results` — Không có kết quả (áp dụng empty state theo COMMON §7)
- `error` — Lỗi (áp dụng toast theo COMMON §6)

### 8. Responsive

Luôn khai báo behavior cho 3 breakpoints theo COMMON §1:

- Mobile (< 768px) — Bottom Sheet pattern
- Tablet (768px–1024px)
- Desktop (> 1024px) — Popup/Panel pattern

---

## Quy tắc đặt tên file

- Feature spec: `features/{feature-name}.md` (kebab-case)
- Tên file = tên feature, không viết tắt
- Ví dụ: `reverse-geocoding.md`, `unified-search-bar.md`, `weather-widget.md`

---

## Khi tạo feature mới

1. Copy `features/template.md` → `features/{tên-feature}.md`
2. Điền Part 01 trước (What, Why)
3. Khai báo header kế thừa COMMON nếu feature có backend
4. Điền Triggers → States → Components → AC → Flow
5. Cập nhật `README.md` thêm link đến feature mới với trạng thái `[ ]`

---

## Khi chỉnh sửa feature đã có

1. Đọc lại feature spec hiện tại **toàn bộ**
2. Đọc `common.md` để đảm bảo không vi phạm shared specs
3. Giữ nguyên cấu trúc 5 Parts, chỉ sửa phần cần thay đổi
4. Cập nhật Changelog ở Part 05

---

## Conventions quan trọng

| Quy tắc | Chi tiết |
|----------|----------|
| Ngôn ngữ | Tiếng Việt cho nội dung spec. Tiếng Anh cho technical terms, code, Gherkin keywords |
| Tham chiếu COMMON | Dùng `(theo COMMON §N)` — KHÔNG copy-paste nội dung COMMON vào feature |
| Error messages | Tiếng Việt, thân thiện, không lộ kỹ thuật. Ví dụ: "Không thể tải địa chỉ, vui lòng thử lại" |
| ID Format | Trigger: `T01, T02...` · AC: `AC-B01, AC-B02...` |
| Dependency | Khai báo rõ trong header: `Depends on: [Feature](./feature.md)` |
| Skeleton loading | Threshold 150ms trước khi show (theo COMMON §4) |
| Toast | Auto-close 4 giây, max 3 stacked (theo COMMON §6) |
| Animation | Xem bảng COMMON §3 — map fly-to 600ms, panel 250ms |

---

## Tham khảo đối thủ

Khi viết feature spec, có thể tham khảo UX/flow từ các đối thủ trong `competitors.md`. Ưu tiên tham khảo:

- **Google Maps** — benchmark toàn cầu
- **Map4D, Vietmap** — context Việt Nam
- **Apple Maps, HERE WeGo** — UX mobile tốt

---

## Checklist trước khi submit

- [ ] Đã đọc `common.md` và `template.md`
- [ ] Part 01 có What + Why rõ ràng
- [ ] Triggers table đầy đủ
- [ ] States inventory đầy đủ (idle, loading, success, zero_results, error)
- [ ] AC viết bằng Gherkin, gắn trigger ID
- [ ] Có AC cho error state
- [ ] Responsive behavior cho 3 breakpoints
- [ ] Tham chiếu COMMON đúng ký hiệu `(theo COMMON §N)`
- [ ] KHÔNG duplicate nội dung từ COMMON
- [ ] `README.md` đã cập nhật
