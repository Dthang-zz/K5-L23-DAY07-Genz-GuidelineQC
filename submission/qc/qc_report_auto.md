# Báo cáo QC tự động (nháp)

## Mẫu

- Cỡ mẫu: 40 ảnh (35 ca khó + 5 ngẫu nhiên), 460 đối tượng; tổng lô: 40 ảnh.
- Phiên bản taxonomy: v1.0

## Số đo

| Chỉ số                                  | Giá trị             | Mẫu số                                                        |
| --------------------------------------- | ------------------- | ------------------------------------------------------------- |
| Tỉ lệ lỗi **theo đối tượng**            | 4/460 = 0.9%        | 460 đối tượng trong mẫu (lỗi bỏ sót tính như 1 đối tượng lỗi) |
| Tỉ lệ **ảnh có lỗi** (dùng cho ngưỡng)  | 4/40 = 10.0%        | 40 ảnh trong mẫu                                              |
| IoU trung bình (IAA, trước calibration) | 0.882 (ngưỡng 0.85) | 314 cặp box ghép                                              |
| Đồng thuận lớp (IAA)                    | 98.4% (ngưỡng 95%)  | 314 cặp box ghép                                              |

Ngưỡng Release: ảnh có lỗi <= 2% (đếm theo ảnh).

## Top 3 loại lỗi

| Loại lỗi              | Số lỗi | Ảnh ví dụ                |
| --------------------- | ------ | ------------------------ |
| bo_sot                | 2      | img_005.jpg, img_019.jpg |
| sai_vi_tri_kich_thuoc | 1      | img_030.jpg              |
| thua                  | 1      | img_036.jpg              |

- tỉ lệ ảnh có lỗi 10.0% > 2%

Cây quyết định: Hold (guideline chưa rõ/dữ liệu hỏng) -> Rework (>ngưỡng, lỗi lặp, IAA thấp) -> Release.
