# Voice Search

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

Voice Search cho phép người dùng tìm kiếm địa điểm bằng giọng nói thay vì gõ phím. Người dùng tap nút [Mic] trên Search Bar, nói query, và kết quả được xử lý tương tự như Forward Geocoding sau khi transcript hoàn tất.

### Why

- **User need:** Khi lái xe hoặc di chuyển, người dùng không thể gõ tìm kiếm — voice search là giải pháp hands-free an toàn và nhanh hơn.
- **Business value:** Voice search tăng engagement trên mobile; là tính năng differentiation quan trọng so với maps tĩnh.

## Unique Selling Propositions (USP)

| #  | USP                              | Mô tả                                                                                     | So sánh                                              |
| -- | -------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1  | Vietnamese speech recognition    | Tối ưu nhận dạng tiếng Việt (vi-VN) — xử lý tốt giọng vùng miền                          | Google Maps dùng Google STT, tốt nhưng xử lý ở nước ngoài |
| 2  | Graceful degradation             | Browser không hỗ trợ → ẩn mic icon thay vì lỗi — UX sạch                                  | Google Maps tương đương                              |
| 3  | Privacy — Web Speech API local   | Dùng Web Speech API browser built-in, không gửi audio lên server GTEL                     | Google Maps gửi audio lên Google server              |

---

# Part 02 - Specifications

## Backend

_Web Speech API (browser built-in) hoặc GTEL Speech-to-Text API (fallback/enterprise)._

## UI/UX & Frontend

### Triggers & Entry Points

| ID  | Trigger                                            | Nền tảng    | Input                           | AC  |
| --- | -------------------------------------------------- | ----------- | ------------------------------- | --- |
| T01 | Click / tap nút [Mic] trên Search Bar              | Web, Mobile | —                               | B01 |
| T02 | Giọng nói được nhận diện (speech recognition)      | Web, Mobile | Audio stream từ microphone      | B02 |
| T03 | Người dùng dừng nói (silence detection)            | Web, Mobile | —                               | B03 |
| T04 | Click [×] hoặc tap ra ngoài để hủy                | Web, Mobile | —                               | B04 |
| T05 | Microphone permission chưa được cấp               | Web, Mobile | Browser Permissions API         | B05 |

### States Inventory

| State          | Mô tả                                      | Component                           |
| -------------- | ------------------------------------------ | ----------------------------------- |
| `idle`         | Chưa kích hoạt                             | Mic button bình thường              |
| `requesting`   | Đang xin quyền microphone                  | Mic button spinner                  |
| `listening`    | Đang nghe giọng nói                        | Overlay với animation sóng âm       |
| `processing`   | Đang xử lý transcript                      | "Đang xử lý..." text                |
| `done`         | Transcript hoàn tất → auto-submit search   | Search flow kích hoạt               |
| `error`        | Lỗi mic / không nhận ra giọng              | Error message + retry button        |
| `denied`       | Quyền mic bị từ chối                       | Mic button disabled + tooltip       |

### Components, Responsive & Typography

#### Component Inventory

| Component                                   | Dùng trong       |
| ------------------------------------------- | ---------------- |
| Mic button [🎤]                              | Tất cả states    |
| Voice listening overlay (full-screen dim)   | listening state  |
| Sound wave animation                        | listening state  |
| Interim transcript text                     | listening state  |
| "Đang xử lý..." indicator                   | processing state |
| Error message + retry                       | error state      |
| Permission tooltip                          | denied state     |

#### Responsive Behavior

_Voice listening overlay full-screen trên cả web và mobile._

#### Typography

### Acceptance Criteria — UI

`Implement: Frontend Dev | Design: DES | Review: QA | Sign-off: QC`

#### AC-B01 · Kích hoạt Voice Search (T01)

```gherkin
Given  người dùng tap nút [Mic]
And    quyền microphone đã được cấp
Then   Voice listening overlay mở (dim background)
And    animation sóng âm bắt đầu (visual feedback đang nghe)
And    speech recognition bắt đầu capture

Given  quyền microphone chưa được xin
Then   browser permission prompt xuất hiện trước
```

#### AC-B02 · Realtime transcript (T02)

```gherkin
Given  người dùng đang nói
Then   interim transcript hiển thị realtime trong overlay (chữ xám / italic)
And    khi nhận ra từ → text cập nhật liên tục
```

#### AC-B03 · Kết thúc nói — auto-submit (T03)

```gherkin
Given  silence detection sau 1.5 giây hoặc người dùng dừng nói
Then   speech recognition kết thúc
And    final transcript được điền vào Search Bar
And    Forward Geocoding / Search auto-submit với transcript (xem forward-geocoding.md §B01)
And    overlay đóng
```

#### AC-B04 · Hủy voice search (T04)

```gherkin
Given  Voice listening overlay đang mở
When   người dùng click [×] hoặc tap ra ngoài overlay
Then   speech recognition dừng
And    overlay đóng
And    Search Bar giữ nguyên text trước đó (không điền transcript)
```

#### AC-B05 · Permission bị từ chối (T05)

```gherkin
Given  người dùng từ chối quyền microphone
Then   overlay đóng
And    Mic button hiển thị disabled
And    tooltip: "Bật quyền microphone trong cài đặt trình duyệt để dùng tính năng này"

Given  permission đã denied trước đó
Then   click Mic button → hiển thị tooltip hướng dẫn ngay (không show prompt)
```

#### AC-B06 · Error states

```gherkin
Given  không nhận ra giọng nói sau 8 giây
Then   overlay hiển thị: "Không nhận ra giọng nói. Vui lòng thử lại."
And    nút [Thử lại] để restart recognition

Given  lỗi microphone hardware
Then   overlay đóng, toast: "Không thể truy cập microphone"
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
