# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Hoang Dat<br>
**MSSV:** 2A202602267<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

---

## 1. Bài độc lập và nguồn dữ liệu

- **Mã SHA-256 của ZIP ảnh được cấp:**
  `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
  (file: `data/day2-cvat-input.zip`)

- **Bốn mã ảnh:**
  | # | Tên file |
  |---|----------|
  | 0 | drive_008.jpg |
  | 1 | drive_022.jpg |
  | 2 | drive_033.jpg |
  | 3 | drive_038.jpg |

  Tất cả 4 ảnh có kích thước **640 × 640 px**.

- **Số vật thể đã gán nhãn:**

  | Ảnh | car | truck | bus | van | Tổng |
  |---|---|---|---|---|---|
  | drive_008 | 23 | 1 | 3 | 3 | **30** |
  | drive_022 | 3 | 1 | 1 | 0 | **5** |
  | drive_033 | 25 | 1 | 2 | 1 | **29** |
  | drive_038 | 26 | 1 | 7 | 5 | **39** |
  | **Tổng** | **77** | **4** | **13** | **9** | **103** |

- **Mã SHA-256 của gói YOLO (day2-my-export.zip):**
  `c0eba0bd23977e52854378ccff8ef8ae8bd694f94973652a57e5d9642d27c745`

- **Mã SHA-256 của gói CVAT gốc (day2-native-export.zip):**
  `b37737a0c0d73122896f2b53b84799d1c658006827ac0d6518beda8ab9708e3d`

- **Nguồn đối chiếu:**
  Làm cá nhân (SOLO) — nhận bộ tham chiếu do người hướng dẫn thực hành cấp.

- **Mã SHA-256 của gói đối chiếu:**
  `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`

- **Mã lần phát và thời điểm nhận bộ tham chiếu:**
  Lần phát #1, nhận lúc 03:08 UTC ngày 2026-09-14 (trước khi gộp kết quả đối chiếu).

**Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:**

Toàn bộ công việc gán nhãn được hoàn thành và cam kết vào kho GitHub trước khi nhận bộ tham chiếu. Hai gói xuất (`day2-my-export.zip` và `day2-native-export.zip`) đã có mã SHA-256 cố định từ trước đó. Thao tác đối chiếu chỉ được thực hiện sau khi kho đã có lịch sử commit rõ ràng, đảm bảo không có sự điều chỉnh ngược chiều theo bộ tham chiếu.

---

## 2. Quyết định phân lớp

| Ảnh / vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
|---|---|---|---|
| drive_008 — vật thể lớn, dạng hộp dài, mặt kính đôi, chiếm ~37% chiều rộng ảnh | **bus** | Thân xe cao bất thường, cửa sổ liên tiếp dọc thân, kích thước lớn hơn hẳn xe xung quanh | Xe > 3 cửa sổ liên tiếp → bus; không phải truck vì không có cabin tách biệt |
| drive_008 — xe nhỏ nằm ở phần trên cảnh, thân vuông, không có thùng hàng | **van** | Thân cao, ngắn, cửa sổ bên hông, không có thùng hàng đằng sau | Cabin + khoang chở liền thân, không phân biệt → van; nếu có thùng riêng → truck |
| drive_008 — xe phía phải, góc cắt sát biên ảnh | **car** (truncated) | Chỉ nhìn thấy phần đuôi, rõ ràng là sedan/hatchback nhỏ | Dù bị cắt, hình dáng phần nhìn thấy đủ để xác định lớp; `boundary=truncated` |
| drive_022 — xe lớn màu trắng xanh, chiếm >30% ảnh | **bus** | Thân dài, cửa sổ liên tiếp, biển số xe buýt | Đặc trưng xe buýt đô thị rõ ràng |
| drive_033 — xe thùng dọc đường, cabin tách riêng | **truck** | Cabin phân biệt với thùng hàng phía sau | Cabin + thùng hàng tách biệt → truck |
| drive_033 — vật thể nhỏ xa, chỉ thấy hình hộp mờ | **car** (unclear) | Không đủ chi tiết nhưng kích thước và tỉ lệ phù hợp xe con | Kích thước phù hợp xe con, không đủ dấu hiệu truck/bus/van → car; `visibility=unclear` |
| drive_038 — xe nằm cạnh lề bên trái, thân cao, cửa sổ nhiều | **van** | Thân cao vuông, cửa hông, không thùng hàng | Tỉ lệ chiều cao/rộng > 0.8 và không có thùng → van |
| drive_038 — phương tiện góc trên bên phải, trục dài | **truck** | Cabin rõ, thùng hàng dài phía sau | Phân biệt rõ cabin và thùng → truck |

**Ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:**

Ảnh `drive_008`, box thứ 11 (xtl=621.70, ytl=550.89): lớp là **car** — đây là thông tin *phân loại*, xác định đây là loại phương tiện gì. Thuộc tính `visibility=unclear` và `boundary=truncated` — đây là thông tin *chất lượng quan sát*, mô tả điều kiện nhìn thấy và việc xe bị cắt ra khỏi khung ảnh. Hai thông tin này hoàn toàn độc lập: cùng một chiếc xe có thể là `car/clear/inside` hoặc `car/occluded/truncated` tùy góc nhìn, nhưng lớp vẫn là `car`.

---

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
|---|---|---|---|
| Xe lớn màu trắng (drive_008) ban đầu gán `truck` | lớp | Nhìn lại thấy có cửa sổ hành khách liên tiếp dọc thân, không thấy thùng hàng | Đổi sang `bus`; quy tắc: thân cao + cửa sổ liên tiếp → bus |
| Một số box bao quá rộng, bao cả khoảng trắng xung quanh xe | hình học | Dùng chế độ zoom CVAT kiểm tra từng box | Kéo sát mép xe; quy tắc: box phải khớp tới cạnh ngoài cùng nhìn thấy |
| Vật thể nhỏ ở xa (drive_033 top area) bỏ sót nhiều box | phạm vi | Xem lại ảnh ở 100% zoom, phát hiện thêm nhiều xe nhỏ | Thêm 8 box mới cho các xe ở phía trên xa; `visibility=unclear` |
| Box xe bị cắt ở biên ảnh thiếu thuộc tính `boundary=truncated` | thuộc tính | Lọc theo `boundary=inside` rồi kiểm lại các box chạm biên ảnh | Sửa sang `boundary=truncated` cho tất cả box có xtl=0 hoặc xbr=640 |
| Xe van nhỏ (drive_008) gán `car` vì thân ngắn | lớp | So sánh lại tỉ lệ chiều cao, nhận ra thân cao bất thường so với xe con | Đổi sang `van`; quy tắc: thân vuông, cao, liền khối → van |

- **Số hộp `needs_review` trước và sau khi kiểm:**
  - Trước khi kiểm: ~35 box cần xem lại (ước tính trong quá trình gán)
  - Sau khi kiểm lại và gộp vào gói xuất cuối: **28 box `needs_review`** / 103 box tổng (27.2%)
  - Trong đó: drive_008: 9, drive_022: 0, drive_033: 12, drive_038: 7

- **Một quyết định chưa đủ bằng chứng và cách xin hỗ trợ:**
  Ảnh `drive_038` góc trái trên có một phương tiện chỉ thấy phần mái và một phần thân — không rõ là `bus` hay `van` lớn (xtl=73.04, ytl=0.00). Tôi đánh dấu `review_state=needs_review` và ghi chú gửi Lab Coach kèm ảnh crop để xin xác nhận.

---

## 4. Một dòng nhãn YOLO

Ví dụ lấy từ `drive_008.txt`, dòng 9 (xe bus trung tâm):

```
2 0.756695 0.349188 0.371797 0.348406
```

- **Tên lớp và tọa độ điểm ảnh `xyxy`:**
  - Lớp: `2` = **bus**
  - x_center = 0.756695 × 640 = **484.3 px**
  - y_center = 0.349188 × 640 = **223.5 px**
  - width    = 0.371797 × 640 = **238.0 px**
  - height   = 0.348406 × 640 = **223.0 px**
  - Tọa độ pixel xyxy: **(365.3, 111.9) → (603.3, 334.9)**
  → Khớp với box trong annotations.xml: `xtl=365.31, ytl=111.99, xbr=603.26, ybr=334.97` ✓

- **Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?**

  Định dạng YOLO chỉ kiểm tra cú pháp (5 số, giá trị trong [0,1] và [0, nc-1]). Nó không kiểm tra nội dung. Ba loại lỗi ngữ nghĩa vẫn có thể tồn tại:
  1. **Sai lớp:** Ghi `1` (truck) thay vì `2` (bus) — định dạng vẫn hợp lệ nhưng mô hình học sai.
  2. **Sai phạm vi:** Box bao phủ vùng trống, hoặc bỏ sót một phần xe — tọa độ hợp lệ nhưng không đại diện đúng đối tượng.
  3. **Sai hình học:** Box bị lệch, không bám theo mép xe, hoặc bao luôn phần xe phía sau — tọa độ vẫn nằm trong ảnh nhưng hộp bao không chính xác.

---

## 5. Huấn luyện và dự đoán thử

- **Ba mã ảnh huấn luyện:** `drive_022`, `drive_033`, `drive_038`
- **Mã ảnh thẩm định:** `drive_008`
- **Mô hình:** `yolo11n.pt` (SHA-256: `0ebbc80d4a7680d14987a577cd21342b65ecfd94632bd9a8da63ae6417644ee1`)
- **Cấu hình:** 8 epochs, batch=4, freeze=10, seed=42, device=CPU, thời gian huấn luyện 31.16 giây

- **Mô tả một dự đoán trong `detect_result.jpg`:**
  Mô hình **không phát hiện được vật thể nào** trên ảnh thẩm định `drive_008` với ngưỡng `conf ≥ 0.25`. Ảnh trả về là ảnh gốc không có bounding box nào được vẽ. Đây là giao lộ có ~30 phương tiện nhưng mô hình bỏ sót toàn bộ.

- **Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?**
  Kết quả rỗng gợi ý hai khả năng: (1) 8 epochs trên 3 ảnh quá ít để mô hình học được đặc trưng phân biệt — backbone bị đóng băng 10 lớp đầu nên chỉ có phần đầu nhỏ được cập nhật, (2) phân bố lớp trong tập train lệch nặng về `car` (74/73 box) khiến mô hình không học được `bus`, `van`, `truck` đủ tốt. Cần kiểm lại xem data.yaml trỏ đúng đường dẫn không và split train/val có cân bằng không.

- **Minh chứng nào có thể bác bỏ nhận định của bạn?**
  Nếu tăng epochs lên 50 và bỏ freeze mà kết quả vẫn rỗng, thì lý do là chất lượng nhãn hoặc data pipeline, không phải số epochs. Nếu thử với ngưỡng `conf=0.01` mà vẫn không có box, thì mô hình hoàn toàn không học được — có thể do data.yaml sai đường dẫn ảnh.

- **Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?**
  Ba ảnh train và một ảnh val đều từ cùng một cảnh quay, cùng góc camera, cùng điều kiện ánh sáng — mô hình không được kiểm tra trên bất kỳ điều kiện nào khác. Tập val chỉ 1 ảnh nên mAP có phương sai cực cao, không thể ước lượng khả năng tổng quát hóa. Đây là công cụ chẩn đoán dữ liệu, không phải benchmark.

---

## 6. Đối chiếu nhãn

Nguồn đối chiếu: **bộ nhãn tham chiếu do Lab Coach cấp** (`teaching_reference`)
SHA-256 bộ đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`

- **Số hộp ghép được:** 48 / 103 (của tôi) — 48 / 50 (của bộ đối chiếu)
- **IoU trung bình:** 0.8715 | **IoU trung vị:** 0.8996
- **Mức đồng thuận lớp:** 72.9% (35/48 box ghép được đồng thuận lớp)
- **Số hộp phía tôi không ghép được:** 55 box (tôi gán thêm nhiều hơn bộ tham chiếu)
- **Số hộp phía đối chiếu không ghép được:** 2 box

- **Một điểm khác biệt cụ thể:**
  Sự không đồng thuận lớp tập trung ở các cặp `bus`↔`van`, `van`↔`truck`, và `truck`↔`bus`. Cụ thể:
  - `drive_008` box 9: tôi gán **bus**, bộ tham chiếu gán **van** (IoU=0.982, rất khớp hình học nhưng lớp khác nhau)
  - `drive_033` box 2: tôi gán **bus**, bộ tham chiếu gán **van** (IoU=0.976)
  - `drive_008` box 4 và 6: tôi gán **van**, bộ tham chiếu gán **truck** (IoU~0.81–0.92)
  - 55 box phía tôi không ghép được: chủ yếu là các xe nhỏ ở xa (`visibility=unclear`) mà bộ tham chiếu không gán — gợi ý tôi đã gán quá nhiều ở vùng xa.

- **Quy tắc hoặc hành động sửa phát sinh:**
  1. Cần làm rõ lại ranh giới `bus` vs `van`: bộ tham chiếu dùng tiêu chí nghiêm ngặt hơn (chỉ gán `bus` khi thấy rõ cửa sổ hành khách liên tiếp). Các xe tôi gán `bus` nhưng bị gán `van` đều là xe thân hộp lớn nhìn từ xa.
  2. Cần đặt ngưỡng kích thước tối thiểu cho box: 55 box không ghép được của tôi chủ yếu là box rất nhỏ (< 20×20 px). Bộ tham chiếu không gán các xe này → cần áp dụng quy tắc bỏ qua vật thể quá nhỏ.

- **Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?**
  IoU trung vị 0.90 và 72.9% đồng thuận lớp cho thấy hình học khá nhất quán, nhưng nếu cả hai nguồn đều mắc cùng loại lỗi hệ thống — ví dụ cả hai đều gán `van` thay vì `bus` cho cùng loại xe — thì đồng thuận vẫn cao dù cả hai đều sai. Đồng thuận đo tính tái lập của quy tắc, không đo tính đúng đắn tuyệt đối.

---

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất (`day2-my-export.zip` và `day2-native-export.zip` với SHA-256 đã ghi ở Mục 1).
- [x] Có thông tin lần huấn luyện và ảnh dự đoán (`training_run.json`, `detect_result.jpg`).
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu (`comparison_summary.json`, `comparison_iou.csv`, `comparison_overlay.png`).
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

---

**Minh chứng mạnh nhất trong bài:**
Bước đối chiếu tự động (`comparison_iou.csv`) cho thấy IoU trung vị 0.90 trên 48 box ghép được — hình học nhất quán cao giữa bài tôi và bộ tham chiếu. Đồng thời, hai gói xuất YOLO và CVAT XML được kiểm chứng chéo đạt `same_annotation_state=True` với IoU tối thiểu > 0.995, xác nhận quy trình xuất từ CVAT nhất quán.

**Câu hỏi còn lại cho Lab Coach:**
1. 55 box phía tôi không ghép được chủ yếu là xe nhỏ ở xa (`visibility=unclear`, box < 25×25 px). Có nên đặt ngưỡng kích thước tối thiểu (ví dụ 15×15 px) để bỏ qua không, hay bộ tham chiếu có quy tắc cụ thể cho trường hợp này?
2. Bất đồng `bus`↔`van` xuất hiện 4 lần với IoU rất cao (0.976–0.982) — hình học khớp nhưng lớp khác nhau. Tiêu chí nào trong bộ tham chiếu phân biệt `bus` và `van` cho xe thân hộp lớn nhìn từ góc cao?
3. Mô hình không phát hiện được vật thể nào trên ảnh val (`conf ≥ 0.25`) sau 8 epochs. Điều này có phải do số lượng ảnh train quá ít hay cần kiểm lại cấu hình data.yaml?
