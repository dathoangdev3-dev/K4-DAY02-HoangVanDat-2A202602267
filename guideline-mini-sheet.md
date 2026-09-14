# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Hoàng Văn Đạt<br>
**MSSV:** 2A202602267<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_008.jpg` — box tọa độ xtl=35.96, ytl=41.00, xbr=150.40, ybr=124.01 (góc trái, phần trước xe lớn màu trắng)
- Dấu hiệu nhìn thấy: Thân xe cao, rộng, nhìn thấy phần đầu và hông. Có ít nhất 2 cửa sổ hành khách liên tiếp dọc thân. Chiều cao thân vượt rõ so với xe con xung quanh. Không thấy khoang hàng tách biệt.
- Quy tắc áp dụng: Thân xe khách dài + nhiều cửa sổ liên tiếp → `bus`. Nếu thân ngắn, hộp, kín, không cửa sổ hành khách nhiều → `van`. Kích thước box (~115×83 px) lớn hơn hẳn các van trong cùng ảnh.
- Quyết định: **bus** (`visibility=occluded`, `boundary=inside`, `review_state=confident`)
- Nếu vẫn thiếu bằng chứng: Đánh dấu `needs_review`, zoom 200% kiểm tra thêm chi tiết cửa sổ và hình dáng đuôi xe, sau đó hỏi Lab Coach kèm ảnh crop.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038.jpg` — box tọa độ xtl=421.21, ytl=136.65, xbr=522.21, ybr=213.60 (xe lớn phía trên, giữa ảnh)
- Dấu hiệu nhìn thấy: Nhìn thấy cabin tách biệt với phần thùng/khoang hàng phía sau. Trục cơ sở dài, chiều cao cabin thấp hơn chiều cao khoang sau. Không có cửa sổ hành khách dọc thân.
- Quy tắc áp dụng: Cabin rõ ràng tách biệt với khoang hàng → `truck`. Nếu thân liền một khối không tách biệt cabin → `van`. Nếu khoang sau không rõ là hàng hay hành khách, ưu tiên xem tỉ lệ chiều dài cabin / tổng chiều dài.
- Quyết định: **truck** (`visibility=clear`, `boundary=inside`, `review_state=confident`)
- Nếu vẫn thiếu bằng chứng: Đánh dấu `needs_review`, xem góc nhìn khác trong video (nếu có), hoặc so sánh với xe truck khác đã xác định trong cùng tập ảnh.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_033.jpg` — box tọa độ xtl=606.50, ytl=289.53, xbr=640.00, ybr=440.39 (phần phải ảnh, chỉ thấy một phần hông xe lớn)
- Dấu hiệu nhìn thấy khi phóng 100%: Chỉ thấy phần hông bên trái của một xe rất lớn, thân cao, mép phải bị cắt hoàn toàn bởi biên ảnh. Có thể nhìn thấy một phần cửa sổ hành khách và thân xe màu trắng. Ước tính chiều cao thân ~150 px — lớn hơn xe con/van thông thường.
- Giá trị `visibility`: `occluded` — phần lớn xe bị mép ảnh và các xe khác che khuất, chỉ nhìn thấy một phần nhỏ.
- Giá trị `boundary`: `truncated` — xe chạm và vượt quá biên phải của ảnh (xbr=640.00).
- Trạng thái `review_state`: `needs_review` — chỉ nhìn thấy ~20% diện tích ước tính của xe, không đủ để xác định chắc chắn là `bus` hay loại xe lớn khác.
- Lý do: Dù có dấu hiệu của xe buýt (thân cao, cửa sổ), phần nhìn thấy quá nhỏ để loại trừ khả năng là van lớn hoặc truck thùng kín. Đánh dấu `needs_review` để xin xác nhận từ Lab Coach.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review` (28 hộp được đánh dấu, đã ghi lý do).
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: **103** (drive_008: 30, drive_022: 5, drive_033: 29, drive_038: 39) — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
