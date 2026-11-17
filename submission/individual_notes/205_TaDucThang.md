# Ghi chú cá nhân — 205 - Tạ Đức Thắng

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh     | v0 không trả lời được gì                                                             | Quyết định ban đầu của tôi                                                                                  |
| ------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| img_007 | Pickup gắn thùng hộp kín, bãi tối: v0 không phân biệt car / truck / van              | `truck`, box gồm cả thùng hộp (cabin và khoang hàng tách rời)                                               |
| img_008 | Xe songthaew (pickup chở khách) không thuộc lớp nào; đám xe máy dày không có quy tắc | Songthaew: chưa chốt (`truck` hay `other_vehicle`), cần họp. Đám xe: gán hàng đầu, phía sau `ignore_region` |
| img_021 | Không có ngưỡng che; không nói cách xử lý đám đông                                   | Xe hàng sau chỉ thấy đèn hậu + biển số → `motorcycle` mức che 3; cụm phía sau → `ignore_region`             |

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- Nhóm chốt theo bản tôi ở D1 (thêm `other_vehicle`), D2 (ngưỡng che 10/40/80 %), D3 (không gán người sau kính), D4 (`truncated` ≥ 10 %), D5 (không tính phần bị chính xe mình che).
- Theo bản Đức ở D6 (vật không chắc lớp: chọn lớp có bằng chứng mạnh nhất + `can_review`), rõ hơn bản tôi chỉ ghi `can_review`.

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: 40. Số box đã vẽ: 390. Số box tôi tick `can_review`: 24.
- Lưỡng lự nhất: img_008 (songthaew là `truck` hay `other_vehicle`) và img_011 (xe máy độ kim loại: xe thật hay tác phẩm điêu khắc). Cả hai đều tick `can_review`.

## 4. Pha 4–5: tôi lệch ở đâu

- Vẽ ít box hơn Đức: 122 box chỉ 153 có. Ví dụ img_021: tôi vẽ 12 `person` + 5 `ignore_region`, Đức vẽ 29 `person` + 2 `ignore_region`. Lệch chủ yếu do **ranh giới `ignore_region`** (quy tắc 3.11 chưa đo được) chứ không chỉ là bỏ sót.
- Mức che: tôi thường ghi cao hơn Đức một mức (51 ca 0 vs 1). Guideline có ngưỡng % nhưng thiếu ảnh mẫu cho từng mức.

## 5. Dùng AI

- Công cụ: Claude Code (Claude), dùng sau khi đã gán nhãn xong, KHÔNG dùng để gán box hay xem ảnh quyết định nhãn.

## 6. Bài học

- Mỗi mức `occluded_level` và ranh giới `ignore_region` cần kèm **ảnh mẫu**. Chỉ ghi % thì hai người vẫn lệch: đồng thuận mức che chỉ đạt 60.5 % (314 cặp box).
