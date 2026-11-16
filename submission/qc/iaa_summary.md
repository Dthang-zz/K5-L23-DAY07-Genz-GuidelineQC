# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 2
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 314 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 314 | 0.882 | 98.4% | 60.5% | 93.6% | 122/53 | 34 | ĐẠT |
| 153|205 | 40 | 314 | 0.882 | 98.4% | 60.5% | 93.6% | 122/53 | 34 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 422 ({'bo_sot_thua': 175, 'sai_thuoc_tinh': 136, 'co_can_review': 25, 'iou_thap': 81, 'sai_lop': 5}), trên 35 ảnh cần calibration.
