# Thông tin nhóm

- Tên nhóm: Genz
- Repo bài nộp (URL): https://github.com/Dthang-zz/K5-L23-DAY07-Genz-GuidelineQC
- Commit chốt bài (hash): lab07_Genz_update
- Số thành viên: 2 → số người gán mỗi ảnh (K): 2 (cả hai người cùng gán đủ 40/40 ảnh)

| STT | MSHV (mã dùng trong lab) | MSHV đầy đủ | Họ và tên       | Lớp (L2 / L3 / khác) | Tài khoản GitHub         |
| --- | ------------------------ | ----------- | --------------- | -------------------- | ------------------------ |
| 1   | 205                      | 2A202603163 | Tạ Đức Thắng    | L2                   | @Dthang-zz               |
| 2   | 153                      | 2A202603153 | Nguyễn Mạnh Đức | L2                   | @nguyenmanhduc17111-lgtm |

Theo hướng dẫn của coach, mã MSHV dùng trong tên file và `--members` là mã 3 số (`split_assign.py --members 153,205`).

Danh sách trên phải khớp dòng tiêu đề của `README.md` (đã đổi theo [SUBMISSION.md](SUBMISSION.md)) và `--members` đã dùng ở `scripts/split_assign.py`.

## Phân công (mỗi người làm gì ngoài phần gán nhãn)

| MSHV        | Phụ trách thêm                                                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 205         | Quản lý repo, viết `guideline_v1.0_agreed.md`, điền `calibration_decisions.csv` + `guideline_v1.1`, kiểm QC nửa mẫu                  |
| 153         | Sửa `taxonomy.json` + chạy `make_cvat_labels.py`, chạy `split_assign.py`, sửa nhãn calibration, viết `QC_REPORT.md`, kiểm QC nửa mẫu |

## Kết quả tóm tắt (điền cuối bài)

- Số thành viên / số lần gán mỗi ảnh (K): 2 / 2
- IAA trước calibration: IoU trung bình 0.882, đồng thuận lớp 98.4 % (mẫu số: 314 cặp box ghép; 175/877 box không ghép được)
- Số ảnh phải calibration: 35 / 40
- Tỉ lệ lỗi sau calibration: 10.0 % theo ảnh (4/40), 0.9 % theo đối tượng (4/460)
- **Quyết định:** Rework — lý do một dòng: 4/40 ảnh có lỗi (10% > ngưỡng 2%), lỗi lặp lại ở cảnh dày đặc (ranh giới `ignore_region`, box trùng).
