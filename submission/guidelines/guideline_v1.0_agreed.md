# Guideline gán nhãn — v1.0 (thống nhất) — Nhóm Genz

- Phiên bản: **v1.0** (khớp `submission/taxonomy.json` version `v1.0`)
- Ngày chốt: 2026-11-16
- Người viết / duyệt: Tạ Đức Thắng, Nguyễn Mạnh Đức
- Nguồn: hợp nhất hai bản cá nhân trong `submission/guidelines/individual/`; chỗ bất đồng chốt theo bảng ở mục 0.
- Loại nhãn: bounding box (Rectangle) · Công cụ: CVAT · Xuất: COCO 1.0 (không kèm ảnh)

> Từ khi chốt v1.0 đến hết Pha 3 **không đổi guideline**. Ca mơ hồ: tick `can_review`, ghi lại, đem ra calibration.

---

## 0. Các chỗ bất đồng giữa hai bản cá nhân và quyết định

| # | Vấn đề | Bản Thắng | Bản Đức | Quyết định v1.0 | Lý do |
|---|---|---|---|---|---|
| D1 | Xe ngoài taxonomy (bus, tuk-tuk, xích lô, xe đạp) | Thêm lớp `other_vehicle` | Không gán, không ignore | **Thêm `other_vehicle`** | Vật cản thật, có ở ≥ 13/40 ảnh (img_004, 009, 016, 023, 029, 035, 040…); không gán thì mô hình học rằng chúng là nền |
| D2 | Ngưỡng `occluded_level` | 10 / 40 / 80 % | 0 / 30 / 70 % | **< 10 / 10–40 / 40–80 / ≥ 80 %** | Mức 0 có dung sai 10% để hai người không lệch vì vài pixel bị che; mức 3 chỉ dành cho vật gần như khuất hẳn |
| D3 | Người trong ô tô nhìn qua kính | Không gán | Gán nếu nhận ra được | **Không gán** | Người sau kính thường mờ/phản sáng → hai người gán khác nhau; xe đã được gán, mô hình không cần người bên trong |
| D4 | Ngưỡng `truncated` | Chạm mép và ≥ 10% vật ngoài ảnh | Có bất kỳ phần nào ra ngoài mép | **Chạm mép và ≥ 10% ngoài ảnh** | Tránh lệch khi vật chỉ chạm mép vài pixel |
| D5 | Phần bị chính xe/người lái che | Không tính vào mức che | Không nói rõ | **Không tính** | Người lái xe máy bình thường luôn bị xe che chân → nếu tính, mọi người lái đều ≥ mức 1, dễ lệch |
| D6 | Vật không chắc lớp | `can_review` | Chọn lớp có bằng chứng mạnh nhất + `can_review` | **Chọn lớp có bằng chứng mạnh nhất + `can_review`** | Hai bản tương thích; ghép lại cho rõ |
| D7 | Vật cao < 20 px | `ignore_region` khi ≥ 3 vật nhỏ sát nhau | `ignore_region` nhỏ sát vật/cụm | **Một vật nhỏ lẻ: không gán gì; ≥ 3 vật nhỏ sát nhau: một `ignore_region` trùm cụm** | Tránh rải hàng chục vùng ignore tí hon, khó đồng thuận |

Các điểm hai bản **giống nhau** (giữ nguyên): box ôm phần nhìn thấy; một box cho vật bị chia mảnh; không gán phản chiếu/hình in/tượng/ma-nơ-canh/đồ chơi; xe chở xe gán riêng từng xe; box xe không ôm hàng nhô ra; người và xe gán box riêng; không gán xe chủ.

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người tham gia giao thông và phương tiện thật** trên ảnh đường phố
(camera giao thông / hỗ trợ lái xe), chủ yếu bối cảnh châu Á: nhiều xe máy, xe ba bánh, đường đông.

Thứ tự mức độ nguy hiểm của lỗi (dùng để quyết ca khó):

1. **Bỏ sót người hoặc xe thật** — xe tự lái không thấy vật cản.
2. **Gán vật không thật thành người/xe** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) — mô hình phanh vô cớ, học sai khái niệm.
3. Sai lớp giữa các loại xe, sai thuộc tính — ít nguy hiểm hơn nhưng làm giảm đồng thuận.

Nguyên tắc chung: **chỉ gán vật thật, hiện diện vật lý trong cảnh, theo bằng chứng nhìn thấy**. Vật không thật để trống
(làm nền), không vẽ `ignore_region` lên chúng, để mô hình học rằng đó *không phải* người/xe.

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người ≤ 9 chỗ, thân liền một khoang người, không có thùng hàng riêng: sedan, hatchback, SUV, wagon, MPV, taxi. | Taxi img_040, SUV img_012, sedan img_023, xe trưng bày img_024 | Pickup (thùng hở tách cabin) → `truck`. Xe đồ chơi img_038 → không gán. |
| `truck` | Xe chở hàng có **khu vực hàng tách khỏi cabin**: pickup (kể cả có nắp/hộp thùng), xe tải nhỏ, xe tải thùng, đầu kéo, xe chở xe. | Xe tải nhỏ img_001, img_036; pickup img_012, img_028, img_037; xe tải img_007; xe chở xe img_002 | Tượng pickup img_027 → không gán. Thân liền một khối → `van`. |
| `van` | Xe **thân hộp liền một khối** (khoang hàng/người chung với cabin), mui cao, mũi ngắn; ≤ 9 chỗ. | Toyota Hiace img_005, img_016; van cứu thương img_011 | MPV mũi dài, mui thấp → `car`. Minibus ≥ 10 chỗ img_009 → `other_vehicle`. |
| `motorcycle` | Xe **hai bánh có động cơ** (xăng hoặc điện): xe máy, tay ga, mô tô, xe điện; **gồm cả** sidecar/thùng gắn bên hông. | img_006, img_021, xe điện img_029, sidecar img_022, xe máy kéo thùng img_025 | Xe đạp → `other_vehicle`. Ba bánh có cabin (tuk-tuk) → `other_vehicle`. |
| `person` | **Người thật nhìn thấy trực tiếp** (không qua kính): đi bộ, đứng, ngồi, nằm, người lái và hành khách. | Người đi bộ img_014, img_034; người lái xe máy img_026; tài xế tuk-tuk img_004 | Tượng img_011, img_017; ma-nơ-canh img_033; người trên biển quảng cáo img_018, img_036; ảnh dán kính img_040; linh vật img_019 → **không gán**. |
| `other_vehicle` | Phương tiện giao thông thật **ngoài 4 lớp xe trên**: xe buýt/minibus, tuk-tuk/xe lam/auto-rickshaw, xích lô, xe đạp, xe đẩy hàng. | Minibus img_009; buýt img_031, img_037, img_040; tuk-tuk img_004, img_016, img_035; xích lô img_023, img_039; xe đạp img_003, img_006, img_029 | Xe đồ chơi/mô hình → không gán. Xe máy kéo thùng → `motorcycle`. |
| `ignore_region` | Vùng chữ nhật trên **cụm vật thật** không gán từng cái được (quá nhỏ hoặc quá dày). Bỏ sót trong vùng không tính lỗi. Không có thuộc tính. | Bãi xe máy dày img_030; đám đông xa img_019, img_021 | Không dùng để "che" vật không thật hay vật rõ, đủ điều kiện gán. |

**Vật không chắc lớp** (ví dụ chỉ thấy đầu xe, không biết SUV hay pickup): chọn lớp có bằng chứng mạnh nhất, tick `can_review`.

**Vật không phải phương tiện, không phải người** (chó img_009, cọc tiêu img_004/img_032, ô dù, hàng hoá): không gán.

**Vật không phải vật thật** — tượng, tượng đài, ma-nơ-canh, linh vật, hình in/biển quảng cáo/tranh tường/màn hình,
phản chiếu (gương, kính, mặt đường ướt), đồ chơi/mô hình: **không gán gì**, kể cả khi trông rất giống thật.

### 2.2. Thuộc tính (áp dụng cho mọi lớp trừ `ignore_region`)

**`occluded_level`** — tỉ lệ **diện tích** vật bị **vật khác** che, ước lượng bằng mắt theo phần tư (¼ ≈ 25%, ½ ≈ 50%…),
so với hình dạng đầy đủ dự kiến của vật trong khung ảnh.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10% | Xe tải img_036, người đi bộ img_034 |
| `1` | 10% ≤ che < 40% | Xe tải img_001 bị người phụ nữ che phần đuôi; ô tô sau hàng rào song thưa img_018 |
| `2` | 40% ≤ che < 80% | Xe máy giữa đám đông img_005; SUV đỏ sau pickup img_028 |
| `3` | che ≥ 80% nhưng **vẫn nhận ra lớp** | Xe máy hàng sau img_021 chỉ thấy đèn hậu + biển số |

- Không tính: phần nằm ngoài ảnh (đó là `truncated`); vật tự che do góc nhìn; phần người bị **chính xe mình đang ngồi** che
  và phần xe bị **chính người lái** che.
- Kính trong suốt không tính là che; khung/cột/song sắt có tính.
- Mờ, tối, phản sáng **không** phải là che.
- Phân vân ở ranh giới 10/40/80% → chọn mức gần nhất theo ước lượng và tick `can_review`.
- Không nhận ra lớp → không gán (hoặc nằm trong `ignore_region` nếu thuộc cụm).

**`truncated`**

| Giá trị | Ngưỡng |
|---|---|
| `yes` | Box chạm mép ảnh **và** ước lượng ≥ 10% vật nằm ngoài khung ảnh |
| `no` | Còn lại (kể cả chỉ chạm mép mà gần như đủ vật) |

**`can_review`** — **bắt buộc tick** khi: (a) không chắc lớp; (b) vật nhoè/tối khiến ranh giới box có thể sai > 10% chiều rộng;
(c) phân vân mức che ở ranh giới; (d) ca nằm trong mục 4.2 "Ca còn mở". Không tick chỉ vì "hơi khó".

## 3. Quy tắc vẽ box

1. **Ôm sát phần nhìn thấy**, song song trục ảnh, không đệm thêm (khe hở ≤ 2 px mỗi cạnh). Không đoán phần bị che.
   Đo theo pixel ảnh gốc, không theo mức zoom.
2. **Vật bị che chia nhiều mảnh** (cột che giữa xe): **một** box trùm từ mảnh trái nhất đến mảnh phải nhất.
3. **Vật bị cắt mép ảnh**: box dừng tại mép ảnh, `truncated` theo ngưỡng 2.2.
4. **Box xe** gồm thân, bánh, gương và phần gắn cố định (giá chở, nắp thùng, hộp thùng). **Không** gồm bóng đổ, hàng hoá nhô ra
   (bao hàng img_035, ống dài img_029, tượng ngựa img_028, thùng giao hàng sau xe máy img_008).
5. **Xe thật được chở trên xe khác** (img_002): gán từng chiếc riêng (`car`…), mức che theo giá chở; giá chở thuộc box `truck`.
6. **Người trên xe**: mỗi người một box `person` (người lái và người ngồi sau tách riêng: img_005, img_026); xe một box riêng.
   Box người gồm cả phần chân chồng lên xe; box xe gồm cả phần yên bị người che.
7. **Người trong xe**: thấy **trực tiếp** (tuk-tuk, xích lô, xe mui trần img_019, sidecar img_022) → gán `person`;
   chỉ thấy **qua kính** (tài xế ô tô, van, cabin xe tải) → không gán.
8. **Phản chiếu, hình in, tượng, mô hình, đồ chơi**: không gán (mặt đường ướt img_015/img_031, gương img_020, kính tủ img_033,
   tượng img_017/img_027/img_028, xe mô hình img_038, biển quảng cáo img_018).
9. **Xe chủ (ego)** — xe camera đang ngồi trên/trong: không gán (gương + tay lái ở đáy img_040; thân xe trong gương img_020).
   Xe khác dù sát camera vẫn gán (img_012).
10. **Vật nhỏ**: cao **< 20 px** → không gán từng vật. Một vật nhỏ lẻ: bỏ qua. **≥ 3 vật nhỏ sát nhau**: một `ignore_region` trùm cụm.
11. **Đám đông / bãi xe dày**: khi ≥ 5 vật cùng loại chồng nhau mà phần lớn ở mức che 3 → `ignore_region` trùm **phần phía sau**;
    vật phía trước còn tách được (cao ≥ 20 px, che ≤ mức 2) vẫn gán riêng. `ignore_region` nhỏ nhất có thể và **không trùm** box đã gán.
    Đám đông mà từng người tách được (img_032) → gán từng người.
12. **Ảnh tối / nhoè chuyển động**: gán nếu vẫn nhận ra lớp. Vật nhoè: box ôm phần thân rõ nhất, **không** ôm vệt mờ kéo dài; tick `can_review` (img_014, img_015).
13. **Ảnh đen trắng / ảnh cũ** (img_022): gán bình thường.
14. **Ảnh không có vật hợp lệ** (img_017, img_020, img_027, img_033, img_038): để trống, vẫn giữ ảnh trong Task/export.

## 4. Ca khó

### 4.1. Ca đã chốt

| Ảnh | Tình huống | Quyết định | Quy tắc |
|---|---|---|---|
| img_002 | Đầu kéo chở 9 ô tô | 1 `truck` (đầu kéo + giá chở) + mỗi ô tô một `car` (che theo giá chở); người đứng sau xe → `person` | 3.5 |
| img_004, img_016, img_035, img_040 | Tuk-tuk / auto-rickshaw | `other_vehicle`; tài xế thấy trực tiếp → `person` | 2.1, 3.7 |
| img_009, img_031, img_037, img_040 | Xe buýt / minibus | `other_vehicle` | 2.1 |
| img_003, img_006, img_010, img_029 | Xe đạp (img_010 xe đạp bày hàng) | `other_vehicle`; người đạp → `person` | 2.1 |
| img_023, img_039 | Xích lô | `other_vehicle`; người đạp và khách thấy trực tiếp → `person` | 2.1, 3.7 |
| img_001 | Xe tải nhỏ cắt mép phải, người che đuôi | `truck`, `truncated=yes`, `occluded_level=1` | 2.2 |
| img_007 | Pickup gắn hộp thùng kín, bãi tối | `truck`, box gồm cả hộp thùng | 3.4 |
| img_037 | Pickup có nắp thùng cao | `truck` (không phải `van`) | 2.1 |
| img_012 | Pickup chiếm nửa ảnh, chụp từ sau thùng | `truck`, `truncated=yes`; hàng trong thùng không gán | 3.9 |
| img_028 | Pickup chở tượng ngựa | `truck` chỉ ôm thân xe; tượng không gán; SUV đỏ phía sau → `car`, `occluded_level=2` | 3.4, 3.8 |
| img_027 | Tượng đài hình pickup | Không gán | 3.8 |
| img_011, img_013, img_017 | Tượng Predator, tượng rồng/sư tử, tượng người cưỡi ngựa | Không gán | 3.8 |
| img_033 | Ma-nơ-canh + phản chiếu trên kính | Không gán gì | 3.8 |
| img_018, img_026, img_030, img_031, img_036 | Người/vật trên biển quảng cáo, tranh tường | Không gán | 3.8 |
| img_020 | Xe trong gương chiếu hậu | Không gán | 3.8, 3.9 |
| img_015, img_031 | Xe phản chiếu trên mặt đường ướt | Không gán phản chiếu; xe thật gán bình thường | 3.8 |
| img_015 | Xe nhoè mờ trong suốt (phơi sáng) | `car` + `can_review` | 3.12 |
| img_014 | Người đi xe máy nhoè hai bên | `person` + `motorcycle` khi nhận ra; box ôm phần rõ nhất; `can_review` | 3.12 |
| img_022 | Mô tô có sidecar (ảnh đen trắng) | Một `motorcycle` gồm sidecar; người lái `person`, `occluded_level=2` | 2.1, 3.13 |
| img_025 | Xe máy kéo thùng hàng bên hông | Một `motorcycle` gồm thùng; người lái `person` | 2.1 |
| img_029 | Xe điện chở ống dài | `motorcycle`, box không ôm ống | 3.4 |
| img_003, img_008, img_021, img_030 | Đám xe máy dày đặc | Gán riêng hàng đầu; phía sau `ignore_region` | 3.11 |
| img_032 | Đám đông chờ xe buýt (~60–100 px) | Gán từng người | 3.11 |
| img_040 | Gương + tay lái ở đáy ảnh | Không gán (xe chủ); người đội mũ → `person`, `truncated=yes` | 3.9 |
| img_005 | Van Hiace sau đám xe máy | `van`, `occluded_level=2` | 2.1 |

### 4.2. Ca còn mở (gán theo lựa chọn tạm thời + `can_review`, đem ra calibration)

| Ảnh | Vấn đề | Lựa chọn tạm thời v1.0 |
|---|---|---|
| img_024 | Xe vàng trên nóc xe đen: xe thật hay mô hình cỡ nhỏ? | Không gán; các xe trưng bày khác → `car` |
| img_008 | Xe songthaew (pickup chở khách) ở xa | `truck` + `can_review` |
| img_011 | Xe máy độ bằng kim loại: xe chạy được hay tác phẩm điêu khắc? | `motorcycle` + `can_review` |
| img_018 | Xe tối màu sau hàng rào, chỉ thấy đầu xe | `car` + `can_review` |
| img_019 | Xe trang trí giả tưởng (có biển số, có người lái) | `car` + `can_review` |
| — | Có nên tách `other_vehicle` thành `bus` / `bicycle` / `three_wheeler`? | Giữ một lớp gộp ở v1.0 |

## 5. QC và phiên bản

### 5.1. Kiểm tra trước khi nộp nhãn

- Đúng danh sách ảnh trong `submission/qc/assignments/<MSHV>.txt`; export COCO 1.0, không kèm ảnh.
- Tên lớp và thuộc tính khớp `taxonomy.json` v1.0 (Project CVAT tạo từ `cvat/labels_generated.json` mới).
- Kiểm lại mức che, cắt mép, `ignore_region`, các box `can_review`.
- Chạy `python scripts/validate_labels.py --all`, sửa hết `LỖI` trước khi push.

### 5.2. Ngưỡng QC (giữ nguyên `rules` trong taxonomy)

| Chỉ số | Ngưỡng |
|---|---|
| Chiều cao tối thiểu | 20 px |
| Ghép cặp box giữa hai người | IoU ≥ 0.5 |
| IoU từng cặp — box cao ≥ 50 px | ≥ 0.85 |
| IoU từng cặp — box cao 20–50 px | ≥ 0.70 |
| IAA: IoU trung bình | ≥ 0.85 |
| IAA: đồng thuận lớp | ≥ 95% |
| Tỉ lệ ảnh lỗi để Release | ≤ 2% |

Báo cáo luôn kèm **mẫu số** (số cặp box, số ảnh). Báo cả tỉ lệ lỗi theo ảnh và theo đối tượng.

Cây quyết định: **Hold** (guideline còn mơ hồ / dữ liệu hỏng) → **Rework** (> 2% ảnh lỗi, lỗi lặp lại cùng kiểu, hoặc IAA dưới ngưỡng) → **Release**.

### 5.3. Ca mơ hồ và calibration

- Khi gán: tick `can_review`, **không tự đoán**, không hỏi nhau về ảnh đang gán.
- Người chốt: cả nhóm (2 người) thống nhất trong buổi calibration; không thống nhất được thì chọn phương án khiến mô hình
  **ít bỏ sót vật thật** hơn (mục 1).
- Mỗi quyết định một dòng trong `submission/qc/calibration_decisions.csv`; quy tắc mới vào `guideline_v1.1_calibrated.md`;
  đổi lớp/thuộc tính thì sửa `taxonomy.json` và tăng version.

### 5.4. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | Thêm lớp `other_vehicle`; định nghĩa lại `car`/`truck`/`van` theo cấu trúc thân xe; `motorcycle` gồm xe điện và sidecar; `person` chỉ người thật thấy trực tiếp | img_002, 004, 009, 016, 022, 023, 029, 035, 037, 040 |
| v1.0 | Ngưỡng đo được: `occluded_level` < 10 / 10–40 / 40–80 / ≥ 80 %; `truncated` ≥ 10% ngoài ảnh; điều kiện bắt buộc `can_review` | D2, D4, D5; img_001, 005, 021, 028 |
| v1.0 | Quy tắc vật không thật, xe chủ, hàng hoá, xe chở xe, người trên/trong xe, vật < 20 px, đám đông, ảnh nhoè | img_011, 013, 014, 015, 017, 019, 020, 027, 028, 030, 032, 033, 038 |
