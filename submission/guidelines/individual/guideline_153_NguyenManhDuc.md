# Guideline gán nhãn cá nhân — 2202603153 - Nguyễn Mạnh Đức

- Ngày: 2026-11-16
- Người viết: Nguyễn Mạnh Đức — MSHV: 2202603153
- Phiên bản: bản cá nhân dự thảo, đề xuất cho v1.0
- Công cụ: CVAT; nhãn Rectangle; xuất COCO 1.0.
- Phạm vi: 40 ảnh gốc trong `data/images/`.

## 1. Mục tiêu

Tạo nhãn huấn luyện mô hình phát hiện người và phương tiện trên ảnh đường phố. Mỗi đối tượng thật, thuộc lớp mục tiêu và đủ điều kiện kích thước được gán một box. Nhãn phải nhất quán giữa các thành viên để đo IAA và kiểm soát lỗi bỏ sót, thừa, sai lớp, sai box và sai thuộc tính.

Chỉ gán theo bằng chứng nhìn thấy trong ảnh. Không suy đoán vật bị che hoàn toàn. Không dùng chuyển động, tình trạng đỗ xe hay có người điều khiển làm điều kiện loại bỏ đối tượng.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Giữ các tên lớp hiện có trong `submission/taxonomy.json`. Các ví dụ dưới đây minh họa quy tắc, chưa phải kết quả kiểm tra ảnh cụ thể.

| Lớp             | Định nghĩa và phạm vi                                                                                                       | Ví dụ                               | Loại trừ / dễ nhầm                                                       |
| --------------- | --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- | ------------------------------------------------------------------------ |
| `car`           | Ô tô chở người dạng sedan, hatchback, SUV hoặc MPV; thân xe không có thùng hàng riêng                                       | Sedan, SUV, taxi dạng ô tô con      | Pickup thuộc `truck`; xe buýt ngoài taxonomy                             |
| `truck`         | Xe vận chuyển hàng có thùng/sàn hàng riêng hoặc đầu kéo nhận diện được                                                      | Pickup, xe tải thùng, đầu kéo       | Xe van có khoang kín liền thân thuộc `van`                               |
| `van`           | Xe thân dạng hộp, khoang kín liền cabin, dùng chở hàng hoặc người; chỉ chọn khi hình dáng đủ rõ để phân biệt với MPV và bus | Van giao hàng, van chở người        | MPV dạng ô tô con thuộc `car`; không suy lớp chỉ từ kích thước biểu kiến |
| `motorcycle`    | Phương tiện hai bánh có động cơ, gồm xe máy và mô tô                                                                        | Scooter, mô tô                      | Xe đạp và tuk-tuk ba bánh ngoài taxonomy                                 |
| `person`        | Người thật, dù đang đi, đứng, ngồi hay nằm; gồm người điều khiển và hành khách nhìn thấy                                    | Người đi bộ, người ngồi trên xe máy | Tượng, ma-nơ-canh, người in trên biển quảng cáo                          |
| `ignore_region` | Vùng không đánh giá từng đối tượng vì quá nhỏ hoặc không thể tách/nhận diện đáng tin cậy                                    | Cụm người quá nhỏ ở xa              | Không dùng để che lỗi hay bao trùm vật rõ, đủ điều kiện gán              |

Vật nhận diện rõ nhưng ngoài taxonomy, như xe đạp, bus, tuk-tuk hoặc rơ-moóc rời: không gán box và không tạo vùng ignore chỉ vì ngoài taxonomy. Nếu nhóm muốn thêm lớp, phải chốt trước khi gán và cập nhật taxonomy tương ứng.

Vật có khả năng thuộc lớp mục tiêu nhưng chưa phân biệt được lớp: chọn lớp có bằng chứng mạnh nhất, bật `can_review`, ghi lý do. Nếu không đủ bằng chứng để chọn bất kỳ lớp mục tiêu nào, đánh dấu vùng khó bằng `ignore_region` và ghi vào danh sách cần review.

### 2.2. Thuộc tính

Áp dụng đủ ba thuộc tính cho mọi box đối tượng; `ignore_region` không cần thuộc tính.

| Thuộc tính       | Giá trị | Quy tắc                                                  |
| ---------------- | ------- | -------------------------------------------------------- |
| `occluded_level` | `0`     | Không thấy phần đối tượng bị vật khác che                |
| `occluded_level` | `1`     | Tỉ lệ bị che lớn hơn 0% và không quá 30%                 |
| `occluded_level` | `2`     | Tỉ lệ bị che lớn hơn 30% và không quá 70%                |
| `occluded_level` | `3`     | Tỉ lệ bị che lớn hơn 70%                                 |
| `truncated`      | `yes`   | Đối tượng tiếp tục ra ngoài ít nhất một mép ảnh          |
| `truncated`      | `no`    | Không có bằng chứng đối tượng bị cắt bởi mép ảnh         |
| `can_review`     | `true`  | Chưa chắc về lớp, box, mức che hoặc cách áp dụng quy tắc |
| `can_review`     | `false` | Áp dụng quy tắc rõ ràng, không có điểm cần xác nhận      |

Tỉ lệ che là ước lượng diện tích hình chiếu của đối tượng bị vật khác che, chia cho diện tích hình chiếu dự kiến của chính đối tượng trong khung ảnh nếu bỏ vật che. Không dùng diện tích nền trong box làm mẫu số; không tính phần nằm ngoài ảnh vào mức che. Đây là ước lượng thị giác, không phải số đo diện tích chính xác. Nếu không ước lượng được hoặc phân vân ở ranh giới 30%/70%, chọn mức có bằng chứng phù hợp nhất và bật `can_review`.

Một vật có thể đồng thời bị che và bị cắt mép. Vật tự che do góc nhìn, ví dụ không thấy mặt sau xe, không được tính là bị vật khác che. Mờ, tối và phản sáng không tự động đồng nghĩa với occlusion.

Các ngưỡng 30%/70% là đề xuất của bản dự thảo, cần nhóm duyệt. Mức `3` được làm rõ thành che trên 70%, thay cho mô tả mơ hồ “không rõ” của v0; sự không chắc chắn dùng `can_review`.

## 3. Quy tắc vẽ box

### 3.1. Ranh giới và kích thước

1. Vẽ box chữ nhật song song trục ảnh, ôm sát toàn bộ phần nhìn thấy của một đối tượng. Không thêm khoảng đệm có chủ ý.
2. Nếu các phần nhìn thấy bị ngắt bởi vật che, dùng một box bao tất cả các phần nhìn thấy thuộc cùng đối tượng. Box có thể chứa vật che ở giữa; không mở rộng để đoán biên bị khuất.
3. Giữ tọa độ trong ảnh. Vật cắt mép: dừng box tại mép ảnh, không kéo ra ngoài.
4. Đánh giá chiều cao box theo pixel ảnh gốc, không theo mức zoom màn hình. Cao từ 20 px trở lên thì gán nếu nhận diện được; dưới 20 px thì dùng vùng ignore nhỏ, sát vật hoặc cụm vật quá nhỏ.
5. Một đối tượng chỉ có một box. Các đối tượng khác nhau có thể có box chồng lấn.
6. Box xe gồm thân, bánh, gương và các phần gắn cố định nhìn thấy được. Không gồm bóng đổ, khói hoặc hàng hóa nhô ra. Box người gồm cơ thể và quần áo/đồ bảo hộ đang mặc; không mở rộng chỉ để chứa hành lý cầm hoặc kéo riêng.

### 3.2. Các tình huống thường gặp

| Tình huống                                    | Quy tắc                                                                                                        |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Người ngồi trên xe                            | Gán riêng người và xe nếu từng đối tượng đủ điều kiện; không gộp thành một box                                 |
| Xe chở xe                                     | Gán riêng xe vận chuyển và từng xe được chở nhận diện được; dùng cùng điều kiện kích thước                     |
| Người trong ô tô                              | Chỉ gán người thật nhìn thấy, đủ 20 px và nhận diện được; không suy ra người từ ghế hoặc kính tối              |
| Vật sau kính                                  | Gán nếu nhìn thấy trực tiếp qua kính và nhận diện được; phần bị khung/cột che vẫn tính occlusion               |
| Phản chiếu trong gương, kính, mặt nước        | Không gán bản phản chiếu; nếu vật thật cũng xuất hiện thì chỉ gán vật thật                                     |
| Hình in, màn hình, tượng, đồ chơi, ma-nơ-canh | Không gán như người/xe thật; nếu không phân biệt được thì ghi ca review                                        |
| Xe chủ gắn camera                             | Loại phần xe chủ nhận diện chắc chắn ở mép ảnh, như nắp ca-pô; không loại xe khác chỉ vì nằm sát camera        |
| Đám đông                                      | Gán từng người đủ điều kiện; chỉ dùng ignore cho phần không thể tách cá thể đáng tin cậy                       |
| Vật bị che trên 70%                           | Vẫn gán nếu phần nhìn thấy đủ kích thước và đủ bằng chứng xác định lớp; chọn mức `3`                           |
| Vật bị che hoàn toàn                          | Không tạo box suy đoán                                                                                         |
| Ảnh mờ/tối                                    | Gán nếu còn xác định được lớp và biên; bật review nếu còn bất định; dùng ignore cục bộ khi không thể nhận diện |
| Không có đối tượng đủ điều kiện               | Giữ ảnh trong Task/export, không tạo box giả                                                                   |

Vùng ignore phải nhỏ nhất có thể mà vẫn bao được vùng cần bỏ qua; tránh bao các đối tượng đã gán rõ ràng. Script có thể miễn lỗi cho box chưa ghép nếu ít nhất 50% diện tích box nằm trong một vùng ignore, nên cần kiểm vùng này bằng mắt.

### 3.3. Làm việc độc lập và xử lý mơ hồ

Sau khi chốt v1.0, không đổi quy tắc trong lúc gán độc lập. Không xem nhãn thành viên khác. Với ca mơ hồ, ghi tên ảnh, đối tượng/vị trí, vấn đề và lựa chọn tạm thời; bật `can_review` nếu có box đối tượng. Vùng ignore không có thuộc tính nên ghi riêng trong danh sách review. Đưa các ca đó ra calibration sau khi nộp nhãn độc lập.

## 4. Ca khó

Nhóm cần tự quan sát bộ ảnh và điền bảng dưới trước khi chốt. Bảng hiện chưa ghi nhận quan sát hoặc quyết định thực tế nào.

| Ảnh                                | Đối tượng / vị trí                          | Vấn đề v0 chưa giải quyết                                             | Quyết định của nhóm                | Lý do / quy tắc áp dụng |
| ---------------------------------- | ------------------------------------------- | --------------------------------------------------------------------- | ---------------------------------- | ----------------------- |
| img_029                            | Xe điện chở ống dài                         | `motorcycle` (xe điện), box không ôm ống                              | Ống là hàng do người giữ           |
| img_021, img_030, img_003, img_008 | Đám xe máy dày đặc                          | Gán riêng các xe/người hàng đầu; phía sau dùng `ignore_region`        | Quy tắc 10                         |
| img_032                            | Đám đông chờ xe buýt (người cao ~60–100 px) | Gán **từng người** (đều ≥ 20 px, tách được)                           | Không đủ điều kiện `ignore_region` |
| img_040                            | Gương và tay lái ở đáy ảnh                  | Không gán (xe chủ); người đội mũ bảo hiểm → `person`, `truncated=yes` | Quy tắc 8                          |
| img_005                            | Van Hiace phía sau đám xe máy               | `van`, `occluded_level=2`                                             | Thân hộp liền khối                 |

Các điểm cần xác nhận khi duyệt: ranh giới car/van/truck; vật ngoài taxonomy; mức che; box phần nhìn thấy; phản chiếu/vật mô phỏng; xe chở xe; vật nhỏ và vùng ignore. Chỉ ghi mã ảnh khi đã kiểm tra đúng ảnh đó.

## 5. QC và phiên bản

### 5.1. Kiểm tra trước khi nộp nhãn

- Đúng danh sách ảnh được giao; xuất COCO 1.0, không kèm ảnh.
- Tên lớp và giá trị thuộc tính khớp taxonomy.
- Box ôm sát phần nhìn thấy, trong biên ảnh, không trùng đối tượng.
- Kiểm lại mức che, cắt mép, các vùng ignore và ca review.
- Chạy `python scripts/validate_labels.py --all`, sửa lỗi trước khi push.

### 5.2. Ngưỡng và cách đọc số liệu

Giữ nguyên `rules` trong taxonomy:

| Chỉ số                   | Ngưỡng / cách tính                                       |
| ------------------------ | -------------------------------------------------------- |
| Chiều cao tối thiểu      | 20 px                                                    |
| Ghép box giữa hai người  | IoU ≥ 0.5; ghép 1-1 không phụ thuộc lớp                  |
| IoU từng cặp box lớn     | ≥ 0.85 khi trung bình chiều cao hai box ≥ 50 px          |
| IoU từng cặp box nhỏ     | ≥ 0.70 khi trung bình chiều cao hai box < 50 px          |
| IAA IoU trung bình       | ≥ 0.85 trên các cặp box ghép được                        |
| Đồng thuận lớp           | ≥ 95% trên các cặp box ghép được                         |
| Tỉ lệ ảnh lỗi để Release | ≤ 2% ảnh trong mẫu, đồng thời đáp ứng các điều kiện dưới |

Báo cáo riêng box không ghép được; không suy ra không bỏ sót chỉ từ IAA cao. Mẫu QC theo lab gồm mọi ảnh bị cờ cộng 20 ảnh chọn ngẫu nhiên theo script, seed 42; báo số ảnh thực tế và mẫu số. Điền mỗi lỗi một dòng trong `qc_review.csv`, ảnh sạch ghi `khong`. Báo cả tỉ lệ theo ảnh và theo đối tượng, nêu rõ cách đếm của script.

Thứ tự quyết định:

1. **Hold:** guideline còn khoảng trống chưa xử lý hoặc dữ liệu có vấn đề.
2. **Rework:** trên 2% ảnh mẫu có lỗi, IAA dưới ngưỡng, hoặc một loại lỗi chiếm ít nhất 50% tổng lỗi và có ít nhất 3 lỗi theo tiêu chí script.
3. **Release:** không thuộc các trường hợp trên, mẫu đã được kiểm đầy đủ và người kiểm xác nhận.

Script kiểm cấu trúc và sự khác nhau giữa các bản nhãn; người kiểm vẫn phải đối chiếu ảnh gốc để phát hiện cả nhóm cùng bỏ sót, cùng sai lớp hoặc box cùng lệch.

### 5.3. Calibration và quản lý phiên bản

- Người điều phối / người chốt: [Nhóm bổ sung].
- Ghi mỗi quyết định vào `calibration_decisions.csv`, gồm ca, ảnh, quyết định, quy tắc, phiên bản và người chốt.
- Quy tắc mới phải vào v1.1 và lịch sử thay đổi; cập nhật taxonomy nếu đổi lớp/thuộc tính.
- Sửa nhãn các ảnh bị cờ theo quyết định, gộp bằng `build_final.py`, rồi QC.
- Giữ bản cá nhân và v1.0 để truy vết; không sửa số liệu IAA bằng tay.
- Sau khi nhóm duyệt dự thảo và điền ca khó, lưu bản chốt thành `guideline_v1.0_agreed.md`, cập nhật version và định nghĩa trong taxonomy rồi chạy `make_cvat_labels.py`.
- Ghi việc dùng AI hỗ trợ soạn và cách nhóm kiểm chứng vào ghi chú cá nhân.

| Phiên bản         | Thay đổi                                                                                    | Lý do / bằng chứng                                     |
| ----------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| v0 → v1.0 dự thảo | Làm rõ lớp, mức che 30%/70%, box phần nhìn thấy, vật ngoài taxonomy, ignore và quy trình QC | Đề xuất để nhóm duyệt; chưa gắn với ca ảnh đã quan sát |
| v1.0 đã duyệt     | [Nhóm bổ sung ngày, thay đổi và người duyệt]                                                | [Các ca ảnh thực tế / quyết định họp]                  |
