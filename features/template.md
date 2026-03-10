# [Feature Name]

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

_[Mô tả ngắn gọn feature làm gì — 1–2 câu. Tập trung vào OUTPUT / hành vi người dùng thấy.]_

### Why

- **User need:** _[Vấn đề người dùng cần giải quyết.]_
- **Business value:** _[Giá trị kinh doanh, KPI liên quan.]_
- **Data sovereignty:** _[Nếu có dữ liệu nhạy cảm — nêu rõ nơi lưu trữ/xử lý.]_

---

# Part 02 - Specifications

## Backend

_[Các API / service backend liên quan. Ghi "Không có backend riêng" nếu pure client-side.]_

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                  | Nền tảng    | Input                  | AC  |
| --- | ------------------------ | ----------- | ---------------------- | --- |
| T01 | _[Trigger 1]_            | Web, Mobile | _[Input data]_         | B01 |
| T02 | _[Trigger 2]_            | Web         | _[Input data]_         | B02 |

### States Inventory

| State     | Mô tả                     | Component                   |
| --------- | ------------------------- | --------------------------- |
| `idle`    | Chưa có tương tác         | _[Component mô tả]_         |
| `loading` | Đang xử lý                | _[Component + loading UI]_  |
| `success` | Thành công                | _[Component thành công]_    |
| `error`   | Lỗi                       | Toast / Error UI            |

### Components, Responsive & Typography

#### Component Inventory

| Component        | Dùng trong |
| ---------------- | ---------- |
| _[Component 1]_  | _[states]_ |
| _[Component 2]_  | _[states]_ |

#### Responsive Behavior

| Breakpoint                 | Mô tả layout       |
| -------------------------- | ------------------- |
| Mobile portrait (< 768px)  | _[layout mobile]_   |
| Tablet (768px–1024px)      | _[layout tablet]_   |
| Desktop (> 1024px)         | _[layout desktop]_  |

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · _[Tên AC]_ (T01)

```gherkin
Given  _[precondition]_
When   _[action]_
Then   _[expected result]_
And    _[additional expectations]_

Given  _[edge case precondition]_
Then   _[edge case result]_
```

#### AC-B02 · _[Tên AC]_ (T02)

```gherkin
Given  _[precondition]_
When   _[action]_
Then   _[expected result]_
```

#### AC-BXX · Error state

```gherkin
Given  API lỗi 4xx/5xx hoặc timeout sau 5 giây (theo COMMON §5)
Then   toast "_[Error message cụ thể]_"
And    toast tự đóng sau 4 giây
```

### Flow — UI

_[Diagram hoặc mô tả luồng UI sẽ được thêm vào đây.]_

## NFR & Performance

_[Non-functional requirements cụ thể của feature này. Xem mặc định tại COMMON Part 02.]_

## Risks & Assumptions

_[Các rủi ro kỹ thuật, UX, hoặc business cần lưu ý. Các assumption đang được dùng.]_

---

# Part 03 - Operations

_[Runbook, monitoring, alerting liên quan đến feature này.]_

---

# Part 04 - Quality & Release

_[Test plan, release checklist, rollout strategy.]_

---

# Part 05 - References

## Document References

_[Links đến designs (Figma), ADRs, API docs, related features.]_

## Changelog

| Date       | Author | Change                |
| ---------- | ------ | --------------------- |
| YYYY-MM-DD | —      | Initial draft         |
