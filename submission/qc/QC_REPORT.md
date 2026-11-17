# Báo cáo QC — Nhóm Genz

## 1. Lô

| Mục                          | Giá trị                                   |
| ---------------------------- | ----------------------------------------- |
| Tên lô                       | day07-street-40                           |
| Số ảnh                       | 40                                        |
| Thành viên (MSHV)            | 153 - Nguyễn Mạnh Đức, 205 - Tạ Đức Thắng |
| Số lần gán mỗi ảnh (K)       | 2 (cả hai người gán đủ 40/40 ảnh)         |
| Phiên bản guideline lúc gán  | v1.0                                      |
| Phiên bản guideline hiện tại | v1.1 (taxonomy giữ v1.0)                  |

## 2. Mẫu QC

- Cỡ mẫu: **40 ảnh** (35 ảnh ca khó + 5 ảnh ngẫu nhiên, seed 42), 460 đối tượng.
- Cách chọn: `qc_sample.py` lấy mọi ảnh bị cờ + 20 ảnh ngẫu nhiên; lô chỉ còn 5 ảnh không bị cờ nên mẫu = **100% lô**. Không nới lỏng so với quy tắc "lô < 500 ảnh kiểm 100%".
- Người kiểm: 205 - Tạ Đức Thắng (AI hỗ trợ xem ảnh); ngày: 2026-11-17.

## 3. Số đo

### 3.1 IAA (trước calibration)

| Chỉ số                                    | Giá trị                 | Mẫu số                                                                   | Ngưỡng | Đạt?                    |
| ----------------------------------------- | ----------------------- | ------------------------------------------------------------------------ | ------ | ----------------------- |
| IoU trung bình (cặp box ghép được)        | 0.882                   | 314 cặp box                                                              | ≥ 0.85 | Đạt                     |
| Đồng thuận lớp                            | 98.4 %                  | 314 cặp box                                                              | ≥ 95%  | Đạt                     |
| Box không ghép được (bỏ sót/thừa)         | 175 (153: 122, 205: 53) | 877 box (153: 487, 205: 390); 34 box bỏ qua do nằm trong `ignore_region` | —      | —                       |
| Đồng thuận `occluded_level` / `truncated` | 60.5 % / 93.6 %         | 314 cặp box                                                              | —      | Thấp ở `occluded_level` |

IAA đạt ngưỡng nhưng **chỉ tính trên box đã ghép**: 175/877 box không ghép được và mức che chỉ đồng thuận 60.5% → vẫn cần calibration (35/40 ảnh bị cờ).

### 3.2 Tỉ lệ lỗi (mẫu QC, sau calibration)

| Cách đếm                           | Lỗi | Mẫu số        | Tỉ lệ      |
| ---------------------------------- | --- | ------------- | ---------- |
| Theo **đối tượng**                 | 4   | 460 đối tượng | 0.9 %      |
| Theo **ảnh** (ngưỡng Release ≤ 2%) | 4   | 40 ảnh        | **10.0 %** |

Theo đối tượng thì thấp, nhưng 4 lỗi rơi vào 4 ảnh khác nhau nên tỉ lệ theo ảnh vượt ngưỡng 5 lần.

## 4. Lỗi

| Hạng | Loại lỗi              | Số lỗi | Ảnh ví dụ                                                                                  | Nguyên nhân gốc                                                                 |
| ---- | --------------------- | ------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| 1    | Bỏ sót                | 2      | img_005 (xe mép trái bị trùm `ignore_region`), img_019 (hàng người sát rào chưa gán riêng) | Guideline mơ hồ: ranh giới `ignore_region` (3.11) chưa đo được → đã làm rõ ở R6 |
| 2    | Sai vị trí/kích thước | 1      | img_030 (#331 box xe máy quá rộng, trùm 2 xe khác)                                         | Lấy box trung bình khi gộp nhãn trong bãi xe dày                                |
| 3    | Thừa                  | 1      | img_036 (#457 mảnh xe 19 px ở mép phải)                                                    | Vật nhỏ ở mép, không nhận ra lớp (R5)                                           |

Có **lỗi lặp lại cùng một kiểu**: có. 3/4 lỗi đều nằm ở **cảnh dày đặc** (đám đông, dãy xe đỗ) và liên quan ranh giới `ignore_region` / vật nhỏ. Ngoài ra script phát hiện 30 cặp box cùng lớp lồng nhau trong nhãn cuối (nhiều nhất img_003, img_006) cần kiểm lại khi Rework.

## 5. Calibration

- Số ảnh bị cờ: 35 / 40 (422 ca: bỏ sót/thừa 175, sai thuộc tính 136, IoU thấp 81, can_review 25, sai lớp 5). Số quyết định đã ghi: **112 dòng** (mỗi ảnh × chủ đề), phủ đủ 35 ảnh (`calibration_decisions.csv`).
- Ba quyết định quan trọng nhất:
  1. img_012, img_016, img_028: pickup gán `car` → `truck` → v1.0 (đã rõ, chỉ là gán nhầm).
  2. img_003, img_019, img_021: hàng trước tách được gán riêng, hàng sau che ≥ 80% gom một `ignore_region`; không trùm `ignore_region` lên vật lớn nhận rõ lớp → v1.1 R6.
  3. Lệch mức che giữa hai mức liền kề → chọn mức thấp hơn; không tính phần bị chính xe mình che → v1.1 R1.
- Quy tắc mới đã vào guideline: v1.1 mục 6 thêm R1–R8 (mức che, `truncated`, vật qua kính, một vật một box, vật nền nhỏ, ranh giới `ignore_region`, vật nhoè, box lệch) và chốt 6 ca còn mở.
- Nguyên nhân bất đồng chính: **guideline mơ hồ** ở mức che và ranh giới `ignore_region` (cả hai cùng lệch, nhiều ảnh); **chưa nắm quy tắc** ở lớp pickup (chỉ 153 lệch) và vẽ box trùng cho cùng một người (chỉ 153).

## 6. Script kiểm được gì, không kiểm được gì

- Script đã kiểm: `validate_labels.py` kiểm tên lớp và giá trị thuộc tính khớp `taxonomy.json`, box nằm trong ảnh, đủ đúng ảnh được giao; `qc_compare.py` ghép box giữa hai người (IoU ≥ 0.5) và liệt kê bỏ sót/thừa, sai lớp, sai thuộc tính, IoU thấp, can_review.
- Sự cố quy trình: file nhãn của 153 từng bị thay bằng bản đã sửa **sau** khi xem IAA (508 box, IoU 0.893 / 99.4% lớp). Script không phát hiện được việc này; nhóm phát hiện khi so số box giữa các phiên bản, đã khôi phục bản gán độc lập (487 box) để IAA có nghĩa và chuyển các sửa đổi sang calibration.
- Script **không** kiểm được: box có ôm sát không, lớp có đúng thực tế không, vật bị **cả hai** cùng bỏ sót, vật không thật (tượng, ma-nơ-canh, xe mô hình). Nhóm xem từng ảnh bị cờ khi calibration và duyệt toàn bộ 40 ảnh nhãn cuối bằng mắt (ảnh `work/review/`).

## 7. Quyết định

**Rework** — người kiểm: 205 - Tạ Đức Thắng, ngày 2026-11-17.

Lý do (đi theo cây Hold → Rework → Release):

1. Guideline chưa rõ / dữ liệu hỏng? Không: các ca mơ hồ đã chốt trong v1.1, không có ảnh hỏng → không Hold.
2. > 2% ảnh lỗi, lỗi lặp lại, hoặc IAA dưới ngưỡng? Có: 4/40 = 10% ảnh lỗi > 2%, và lỗi lặp lại cùng kiểu ở cảnh dày đặc.
3. Kết luận: **Rework**, chưa Release.

Việc cần làm tiếp:

- 153: sửa 4 ảnh lỗi (img_005, img_019, img_030, img_036) trong CVAT theo v1.1 R4–R6, export lại `labels_calibrated_flagged.json`, chạy `build_final.py`.
- 205: kiểm lại 30 cặp box lồng nhau (img_003, img_006, img_008, img_021, img_030, img_035, img_036), rồi chạy lại `qc_sample.py --force` và `qc_report.py`, ghi vòng 2 vào báo cáo này.
