# Ghi chú cá nhân — 153 - Nguyễn Mạnh Đức

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

> Bản guideline Pha 1 của tôi chưa ghi ca khó theo ảnh; cột cuối là cách tôi xử lý khi gán ở Pha 3.

| Ảnh     | v0 không trả lời được gì                                         | Quyết định của tôi (Pha 3)                                 |
| ------- | ---------------------------------------------------------------- | ---------------------------------------------------------- |
| img_037 | Pickup có nắp thùng cao: `truck` hay `van`? Xe buýt không có lớp | 1 `other_vehicle`, 1 `ignore_region`; không có box `truck` |
| img_011 | Tượng người, xe máy độ kim loại: vật thật hay không?             | 16 `person`, 1 `motorcycle`, 1 `ignore_region`             |
| img_012 | SUV hay pickup: v0 không định nghĩa ranh giới car / truck        | Gán `car` cho xe mà v1.0 xếp là pickup → `truck`           |

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- D1: tôi không gán xe ngoài taxonomy → nhóm thêm `other_vehicle`.
- D3: tôi gán người sau kính → nhóm chốt không gán.
- D4: tôi tính `truncated` khi có bất kỳ phần nào ra ngoài → nhóm chốt ≥ 10 %.
- D6: nhóm theo bản tôi (chọn lớp mạnh nhất + `can_review`).

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: 40. Số box đã vẽ: 487. Số box tôi tick `can_review`: 1.
- Lưỡng lự nhất: img_007, pickup gắn thùng hộp kín trong bãi tối (box duy nhất tôi tick `can_review`).

## 4. Pha 4–5: tôi lệch ở đâu

- **Chưa nắm quy tắc:** gán pickup là `car` ở img_012, img_028 (và img_016), dù v1.0 ghi rõ pickup → `truck`.
- Vẽ nhiều box hơn Thắng: 122 box chỉ tôi có (person 72, motorcycle 32), nhiều nhất ở img_021, img_003, img_011, img_006. Cần calibration để xác định tôi gán thừa (tượng, vật trong vùng ignore) hay Thắng bỏ sót.
- Tôi gần như không dùng `can_review` (1 so với 24 của Thắng), nên các ca mơ hồ không được đánh dấu.
- Sự cố: tôi đã sửa file nhãn sau khi xem IAA (508 box). Nhóm khôi phục bản gán độc lập (487 box) để tính IAA; các sửa đổi chuyển sang calibration.

## 5. Dùng AI

<<Ghi công cụ AI đã dùng, việc gì, kiểm tra thế nào; hoặc "không dùng".>>

## 6. Bài học

- Ghi ca khó kèm mã ảnh ngay từ Pha 1, và tick `can_review` khi phân vân thay vì tự đoán.
