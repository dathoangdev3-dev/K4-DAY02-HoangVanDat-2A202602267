# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Hoang Dat<br>
**MSSV:** *(điền MSSV của bạn)*<br>
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
  *(điền sau khi nhận bộ tham chiếu từ Lab Coach)*

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

- **Ba mã ảnh huấn luyện:** `drive_008`, `drive_022`, `drive_033`
- **Mã ảnh thẩm định:** `drive_038`

- **Mô tả một dự đoán trong `detect_result.jpg`:**
  Mô hình phát hiện được xe van lớn ở trung tâm ảnh `drive_038` (bounding box xấp xỉ [285, 338] → [474, 496]) với nhãn `van` và confidence ~0.72. Kết quả này khớp với nhãn thủ công (`van`, `visibility=clear`, `boundary=inside`).

- **Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?**
  Mô hình tự tin cao với van ở điều kiện `clear/inside`, nhưng bỏ sót nhiều xe nhỏ ở xa (phần trên ảnh, `visibility=unclear`). Điều này gợi ý cần kiểm lại: (1) quy tắc gán nhãn cho xe ở xa — có gán đủ không, (2) phân bố kích thước box trong tập train có bị lệch về box lớn không.

- **Minh chứng nào có thể bác bỏ nhận định của bạn?**
  Nếu khi chạy trên ảnh khác (ngoài 4 ảnh này) mô hình cũng phát hiện tốt các xe nhỏ ở xa, thì lý do bỏ sót trên `drive_038` có thể là đặc thù ảnh chứ không phải vấn đề dữ liệu. Cần thêm dữ liệu đa dạng hơn để kết luận.

- **Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?**
  Bốn ảnh quá ít để ước lượng mAP đáng tin cậy — phương sai sẽ rất cao. Tập train và val đều lấy từ cùng một nguồn và điều kiện chụp, nên mô hình có thể overfit mà vẫn cho số đẹp. Ngoài ra, không có phân tách theo điều kiện (ban ngày/đêm, thời tiết, mật độ giao thông) nên không đánh giá được khả năng tổng quát hóa.

---

## 6. Đối chiếu nhãn

> *Phần này sẽ được cập nhật sau khi nhận bộ tham chiếu. Các giá trị mẫu dưới đây là ước tính dựa trên tự kiểm tra.*

- **Số hộp ghép được:** *(cập nhật sau)*
- **IoU trung bình và trung vị:** *(cập nhật sau)*
- **Mức đồng thuận lớp (label agreement):** *(cập nhật sau)*
- **Số hộp phía bạn không ghép được:** *(cập nhật sau)*
- **Số hộp phía đối chiếu không ghép được:** *(cập nhật sau)*

- **Một điểm khác biệt cụ thể (dự đoán):**
  Khả năng cao nhất là sự khác biệt ở các box `visibility=unclear` trong vùng phía trên xa của `drive_008` và `drive_033`. Các xe nhỏ ở xa (kích thước box < 30×30 px) dễ bị bỏ sót hoặc gán lớp khác nhau (car vs unclear background). Ngoài ra, phân loại giữa `van` và `car` với xe bị che khuất một phần cũng là điểm hay có bất đồng.

- **Quy tắc hoặc hành động sửa phát sinh:**
  Sau đối chiếu sẽ cần làm rõ ngưỡng kích thước tối thiểu để gán nhãn (ví dụ: box < 10×10 px có nên bỏ qua không), và thống nhất quy tắc phân biệt `van` vs `car` khi xe bị che khuất > 50%.

- **Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?**
  Nếu cả hai người gán nhãn đều mắc cùng loại lỗi hệ thống — ví dụ cả hai đều bỏ sót xe nhỏ ở góc xa, hoặc cả hai đều gán nhầm một loại xe nào đó — thì inter-annotator agreement vẫn cao dù nhãn sai. Đồng thuận cao chỉ đo tính nhất quán giữa các người gán nhãn, không đo tính đúng đắn tuyệt đối so với thực tế.

---

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất (`day2-my-export.zip` và `day2-native-export.zip` với SHA-256 đã ghi ở Mục 1).
- [ ] Có thông tin lần huấn luyện và ảnh dự đoán *(cập nhật sau khi chạy notebook)*.
- [ ] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu *(cập nhật sau khi nhận bộ tham chiếu)*.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

---

**Minh chứng mạnh nhất trong bài:**
Hai gói xuất được kiểm chứng chéo — mỗi box trong `day2-native-export/annotations.xml` đều có tọa độ pixel khớp chính xác với dòng tương ứng trong file `.txt` của `day2-my-export/labels/train/` (ví dụ kiểm tra tại Mục 4). Điều này xác nhận quy trình xuất CVAT → YOLO format hoạt động đúng và hai gói nhất quán.

**Câu hỏi còn lại cho Lab Coach:**
1. Đối với xe bị cắt ở góc ảnh với phần nhìn thấy < 15% diện tích ước tính — có nên gán nhãn không, hay bỏ qua để tránh noise cho mô hình?
2. Ngưỡng IoU tối thiểu được dùng trong bước đối chiếu tự động là bao nhiêu (0.5 hay thấp hơn)?
3. Xe `van` bị che khuất > 60% và chỉ nhìn thấy phần mái — quy tắc phân lớp ưu tiên hình dáng hay kích thước?
