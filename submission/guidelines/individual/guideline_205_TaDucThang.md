# Guideline gán nhãn — v0.1 (cá nhân) — 205 - Tạ Đức Thắng | Nhóm Genz

- Phiên bản: v0.1 (bản cá nhân, viết độc lập ở Pha 1, sửa từ v0 của giảng viên)
- Ngày: 2026-11-16
- Người viết: 205 - Tạ Đức Thắng
- Loại nhãn: bounding box trên ảnh đường phố · Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người tham gia giao thông và phương tiện thật** trên ảnh đường phố
(camera giao thông / hỗ trợ lái xe), chủ yếu bối cảnh châu Á: nhiều xe máy, xe ba bánh, đường đông.

Thứ tự mức độ nguy hiểm của lỗi, dùng để quyết định ca khó:

1. **Bỏ sót người hoặc xe thật** (nguy hiểm nhất: xe tự lái không thấy vật cản).
2. **Gán vật không thật thành người/xe** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi): mô hình sẽ phanh vô cớ và học sai khái niệm.
3. Sai lớp giữa các loại xe, sai thuộc tính: ít nguy hiểm hơn nhưng làm giảm đồng thuận.

Nguyên tắc chung: **chỉ gán vật thật, đang hiện diện vật lý trong cảnh**. Vật không thật để trống (làm nền),
không vẽ `ignore_region` lên chúng — để mô hình học được rằng đó *không phải* người/xe.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô chở người ≤ 9 chỗ, thân xe liền một khoang người: sedan, hatchback, SUV, wagon, taxi, xe thể thao. | Taxi img_040, SUV img_012, sedan img_023, xe trưng bày img_024 | SUV vs pickup: pickup có **thùng hở phía sau tách khỏi cabin** → `truck`. Xe đồ chơi img_038 → không gán. |
| `truck` | Xe chở hàng có **khu vực chở hàng tách khỏi cabin**: pickup (kể cả có nắp thùng/hộp thùng), xe tải nhỏ, xe tải thùng, đầu kéo/xe chở xe. | Xe tải nhỏ img_001, img_036; pickup img_012, img_028, img_037; xe tải ISUZU img_007; xe chở xe img_002 | Tượng pickup img_027 → không gán. Van (thân liền một khối) → `van`. |
| `van` | Xe **thân hộp liền một khối** (khoang hàng/người chung với cabin), mui cao, mũi xe ngắn; thường cửa lùa. | Toyota Hiace img_005, img_016; xe cứu thương dạng van img_011 | Minivan/MPV mũi dài, mui thấp → `car`. Xe buýt nhỏ (minibus ≥ 10 chỗ) img_009 → `other_vehicle`. |
| `motorcycle` | Xe **hai bánh có động cơ** (xăng hoặc điện): xe máy, xe tay ga, mô tô, xe điện; kể cả khi gắn **thùng/ghế bên hông** (sidecar). | img_006, img_021, xe điện img_029, sidecar img_022, xe máy kéo thùng hàng img_025 | Xe đạp → `other_vehicle`. Xe ba bánh có cabin (tuk-tuk) → `other_vehicle`. |
| `person` | **Người thật**: đi bộ, đứng, ngồi, hoặc ngồi trên xe *nhìn thấy trực tiếp* (không qua kính). | Người đi bộ img_014, img_034; người lái xe máy img_026; tài xế tuk-tuk img_004 | Tượng img_017, img_011; ma-nơ-canh img_033; người trên biển quảng cáo img_018, img_036; ảnh dán kính img_040; linh vật hình chuột img_019 → **không gán**. |
| `other_vehicle` | **(Đề xuất thêm lớp mới)** Phương tiện giao thông thật không thuộc 4 lớp xe trên: xe buýt, minibus, tuk-tuk/xe lam/auto-rickshaw, xích lô, xe đạp, xe đẩy hàng. | Minibus img_009; buýt img_031, img_037, img_040; tuk-tuk img_004, img_016, img_035; xích lô img_023, img_039; xe đạp img_003, img_006, img_029 | Xe đồ chơi/mô hình → không gán. Xe máy kéo thùng → `motorcycle`. |
| `ignore_region` | Vùng chữ nhật trên **cụm vật thật** không thể gán từng cái (quá nhỏ hoặc quá dày); bỏ sót trong vùng này không tính lỗi. | Bãi xe máy dày img_030; đám đông xa img_019, img_021 | Không dùng để "che" vật không thật. |

**Vì sao thêm `other_vehicle`** (phép thử "mô hình có phải đoán ra không?"): xe buýt, tuk-tuk, xe đạp là **vật cản thật**
mà mô hình phải phát hiện; nếu không gán, mô hình bị dạy rằng tuk-tuk là nền. Chúng xuất hiện ở ≥ 13/40 ảnh nên không thể bỏ qua.
Loại cụ thể (buýt / tuk-tuk / xe đạp) chưa tách thành lớp riêng vì số mẫu ít — *chưa chốt, cần họp* (xem mục 4).

**Vật không thuộc lớp nào và không phải phương tiện** (chó img_009, cọc tiêu img_004/img_032, ô dù, hàng hoá): không gán.

**Vật không phải vật thật** — tượng, tượng đài, ma-nơ-canh, linh vật, hình in/biển quảng cáo/tranh tường, ảnh phản chiếu
(gương, kính, mặt đường ướt), đồ chơi/mô hình: **không gán gì cả**, kể cả khi trông rất giống thật.

### Thuộc tính

`occluded_level`: tỉ lệ **diện tích** của vật bị **vật khác** che (ước lượng theo phần tư: ¼ ≈ 25%, ½ ≈ 50%…).
Không tính phần bị cắt bởi mép ảnh (đó là `truncated`).

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10% | Xe tải img_036, người đi bộ img_034 |
| `1` | che 10% – < 40% | Xe tải img_001 bị người phụ nữ che phần đuôi |
| `2` | che 40% – < 80% | Xe máy giữa đám đông img_005 |
| `3` | che ≥ 80% nhưng **vẫn nhận ra được lớp** | Xe máy hàng sau img_021 chỉ thấy đèn hậu + biển số |

- Không nhận ra được lớp → không gán (hoặc nằm trong `ignore_region` nếu thuộc cụm).
- **Phần người bị chính chiếc xe mình đang ngồi che, và phần xe bị chính người lái che, không tính** vào `occluded_level`
  (người lái xe máy bình thường = mức 0 nếu không có gì khác che).
- Hàng rào song thưa/kính trong suốt: tính phần diện tích thực sự bị che (song sắt img_018 ≈ 10–20% → mức 1). Kính trong không tính.

`truncated`:

| Giá trị | Ngưỡng |
|---|---|
| `yes` | Box chạm mép ảnh **và** ước lượng ≥ 10% vật nằm ngoài khung ảnh |
| `no` | Còn lại (kể cả chỉ chạm mép mà gần như đủ vật) |

`can_review`: **bắt buộc tick** khi (a) không chắc lớp (ví dụ SUV hay pickup khi không thấy thùng), (b) vật bị nhoè chuyển động
khiến ranh giới box sai > 10% chiều rộng, (c) ca được liệt kê ở mục "Chưa chốt". Không tick chỉ vì "hơi khó".

## 3. Quy tắc vẽ box

1. **Ôm sát phần nhìn thấy** (không đoán phần bị che). Khe hở giữa box và vật ≤ 2 px mỗi cạnh. Gương chiếu hậu thuộc xe nên nằm trong box.
2. **Vật bị che chia làm nhiều mảnh** (ví dụ cột che giữa xe): **một** box trùm từ mảnh trái nhất đến mảnh phải nhất.
3. **Vật bị cắt mép ảnh**: box đến sát mép ảnh, `truncated = yes` theo ngưỡng trên.
4. **Người trên xe**: mỗi người một box `person` (lái + người ngồi sau tách riêng, img_005, img_026), xe một box riêng.
   Box người gồm cả phần chân chồng lên xe; box xe gồm cả phần yên bị người che.
5. **Người trong xe**: nhìn thấy **trực tiếp** (tuk-tuk, xích lô, xe mui trần img_019, sidecar img_022) → gán `person`;
   chỉ thấy **qua kính** (tài xế ô tô, van, cabin xe tải) → không gán.
6. **Hàng hoá chở trên xe**: box xe **chỉ ôm thân xe** (khung, thùng, giá chở cố định, bánh), **không** ôm hàng nhô ra ngoài
   (bao hàng img_035, ống dài img_029, tượng ngựa img_028, thùng giao hàng sau xe máy img_008).
   Ngoại lệ: **xe thật được chở trên xe khác** (img_002) vẫn gán từng chiếc `car` riêng với `occluded_level` tương ứng; giá chở thuộc box `truck`.
7. **Phản chiếu, hình in, tượng, mô hình, đồ chơi**: không gán (img_015/img_031 mặt đường ướt, img_020 gương, img_033 kính tủ,
   img_017/img_027/img_028 tượng, img_038 xe mô hình, img_018 biển quảng cáo).
8. **Xe chủ (ego)** — xe mà camera đang ngồi trên/trong: không gán (gương + tay lái xe máy ở đáy img_040; thân xe trong gương img_020).
9. **Vật quá nhỏ**: cao **< 20 px** → không gán từng vật; nếu ≥ 3 vật nhỏ sát nhau thì vẽ một `ignore_region` trùm cụm.
10. **Đám đông / bãi xe dày**: khi ≥ 5 vật cùng loại chồng lên nhau mà phần lớn ở mức che 3 → vẽ `ignore_region` trùm **phần phía sau**;
    các vật phía trước còn tách được (cao ≥ 20 px, che ≤ mức 2) vẫn gán riêng, và `ignore_region` **không trùm** lên box đã gán.
11. **Ảnh tối / nhoè chuyển động**: gán nếu vẫn nhận ra lớp. Với nhoè, box ôm phần thân rõ nhất, **không** ôm vệt mờ kéo dài; tick `can_review` (img_014, img_015).
12. **Ảnh đen trắng / ảnh cũ** (img_022): gán bình thường.
13. **Ảnh không có vật hợp lệ** (img_017, img_020, img_027, img_033, img_038): để trống, vẫn tính là đã gán.

## 4. Ca khó

| Ảnh | Tình huống | Quyết định | Lý do |
|---|---|---|---|
| img_002 | Xe đầu kéo chở 9 ô tô | 1 box `truck` (đầu kéo + giá chở) + mỗi ô tô một box `car` (che theo giá chở); người đứng sau xe → `person` | Các ô tô là xe thật, mô hình phát hiện chúng là đúng; hàng hoá khác thì không gán |
| img_004, img_016, img_035, img_040 | Tuk-tuk / auto-rickshaw (không có trong v0) | `other_vehicle`; tài xế nhìn thấy trực tiếp → `person` | Vật cản thật, không thuộc 4 lớp xe |
| img_009, img_031, img_037, img_040 | Xe buýt / minibus | `other_vehicle` | Không phải van (> 9 chỗ), không phải truck |
| img_003, img_006, img_010, img_029 | Xe đạp (img_010 xe đạp chở hàng quả) | `other_vehicle`; người đạp → `person` | Không có động cơ nên không phải `motorcycle` |
| img_023, img_039 | Xích lô có người đạp | `other_vehicle` + `person` cho người đạp và khách (nếu thấy trực tiếp) | Như trên |
| img_001 | Xe tải nhỏ Suzuki bị cắt mép phải, người đứng che đuôi | `truck`, `truncated=yes`, `occluded_level=1` | Có thùng hàng tách cabin |
| img_007 | Pickup gắn thùng hộp kín phía sau, bãi tối | `truck` (box gồm cả thùng hộp vì là thân xe cố định) | Cabin và khoang hàng tách rời |
| img_037 | Pickup có nắp thùng (truck cap) cao | `truck`, không phải `van` | Nhìn bên hông thấy cabin và thùng tách rời |
| img_012 | Pickup chiếm nửa ảnh, chụp từ phía sau thùng | `truck`, `truncated=yes`; hàng trong thùng không gán | Xe đang đỗ, không phải xe chủ (camera không ở trên xe) |
| img_028 | Pickup chở tượng ngựa | `truck` ôm thân xe, không ôm tượng; tượng không gán; SUV đỏ phía sau → `car`, `occluded_level=2` | Tượng là vật không thật; hàng không thuộc box xe |
| img_027 | Tượng đài hình pickup bằng đá | Không gán | Không phải xe thật |
| img_017, img_011, img_013 | Tượng người cưỡi ngựa, tượng Predator, tượng rồng/sư tử | Không gán | Vật không thật |
| img_033 | Ma-nơ-canh + người/xe phản chiếu trên kính | Không gán gì | Không phải vật thật |
| img_018, img_026, img_030, img_031, img_036 | Người/vật trên biển quảng cáo, tranh tường | Không gán | Hình in |
| img_020 | Xe trong gương chiếu hậu | Không gán | Phản chiếu + xe chủ |
| img_015, img_031 | Xe phản chiếu trên mặt đường ướt | Không gán phần phản chiếu; xe thật gán bình thường | Phản chiếu |
| img_015 | Xe chạy nhoè mờ trong suốt (phơi sáng) | `car` + `can_review` | Xe thật nhưng ranh giới không rõ |
| img_014 | Người đi xe máy nhoè chuyển động hai bên | Gán `person` + `motorcycle` khi nhận ra, box ôm phần thân rõ nhất, `can_review` | Vật thật nhưng nhoè |
| img_019 | Xe Danger Mouse (xe trang trí diễu hành, có biển số) | `car` + `can_review`; người lái → `person`; linh vật chuột → không gán; đám đông sau rào → `ignore_region` | Xe thật chạy trên đường dù hình dạng giả tưởng |
| img_022 | Mô tô có sidecar (ảnh đen trắng) | Một box `motorcycle` gồm cả sidecar; người lái → `person`, `occluded_level=2` (bị kính chắn gió che) | Sidecar gắn liền xe |
| img_025 | Xe máy kéo thùng hàng bên hông | Một box `motorcycle` gồm cả thùng; người lái `person` | Thống nhất với sidecar |
| img_029 | Xe điện chở ống dài | `motorcycle` (xe điện), box không ôm ống | Ống là hàng do người giữ |
| img_021, img_030, img_003, img_008 | Đám xe máy dày đặc | Gán riêng các xe/người hàng đầu; phía sau dùng `ignore_region` | Quy tắc 10 |
| img_032 | Đám đông chờ xe buýt (người cao ~60–100 px) | Gán **từng người** (đều ≥ 20 px, tách được) | Không đủ điều kiện `ignore_region` |
| img_040 | Gương và tay lái ở đáy ảnh | Không gán (xe chủ); người đội mũ bảo hiểm → `person`, `truncated=yes` | Quy tắc 8 |
| img_005 | Van Hiace phía sau đám xe máy | `van`, `occluded_level=2` | Thân hộp liền khối |

**Chưa chốt, cần họp:**

- `other_vehicle` có nên tách thành `bus`, `bicycle`, `three_wheeler` không? (img_004, img_009, img_029) — tách thì chính xác hơn nhưng mỗi lớp ít mẫu.
- img_024: chiếc xe vàng trên nóc xe đen là **xe thật** hay **mô hình cỡ nhỏ**? Tạm thời: không gán. Các xe trưng bày trong bảo tàng → `car`.
- img_008: xe songthaew (pickup chở khách) ở xa → `truck` hay `other_vehicle`?
- img_011: các xe máy độ bằng kim loại có phải xe chạy được hay tác phẩm điêu khắc? Tạm: `motorcycle` + `can_review`.
- img_018: xe tối màu sau hàng rào chỉ thấy đầu xe — SUV (`car`) hay pickup (`truck`)? Tạm: `car` + `can_review`.
- img_019: xe giả tưởng có nên coi là `car` không (mô hình sẽ gặp xe trang trí ở lễ hội)?

## 5. QC và phiên bản

- Ngưỡng QC (theo `submission/taxonomy.json`, không đổi): IoU ≥ 0.85 với box cao ≥ 50 px, ≥ 0.7 với box cao 20–50 px;
  IAA: IoU trung bình ≥ 0.85 và cùng lớp ≥ 95%; tỉ lệ ảnh lỗi ≤ 2% mới Release.
- Ca mơ hồ khi gán: tick `can_review`, **không tự đoán**, ghi lại để đem ra họp calibration.
- Ai quyết ca mơ hồ: cả nhóm biểu quyết trong buổi calibration; hoà thì chọn phương án khiến mô hình **ít bỏ sót vật thật** hơn (mục 1).
- Mọi thay đổi làm đổi nhãn → tăng phiên bản và ghi vào bảng dưới.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1 | Thêm lớp `other_vehicle`; định nghĩa lại `car`/`truck`/`van` theo cấu trúc thân xe; ngưỡng % cho `occluded_level`, `truncated`; quy tắc vật không thật, xe chủ, hàng hoá, người trên/trong xe, đám đông, vật < 20 px, ảnh nhoè | img_002, 004, 009, 016, 017, 019, 020, 022, 027, 028, 033, 035, 037, 038, 040 |
