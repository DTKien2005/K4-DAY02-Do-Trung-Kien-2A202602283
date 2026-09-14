# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Đỗ Trung Kiên<br>
**MSSV:** 2A202602283<br>
**Hình thức:** cá nhân<br>
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

- Ảnh và mã vật thể: `drive_038`, phương tiện thân hộp dạng trung bình ở làn giữa.
- Dấu hiệu nhìn thấy: Thân xe dạng khối hộp kín, trần cao, có các hàng cửa sổ bên hông nhưng chiều dài tổng thể ngắn, chỉ có khoảng 3–4 hàng ghế (dạng xe 16 chỗ), không có chiều dài vượt trội hay cửa lên xuống lớn như xe khách/xe buýt tuyến.
- Quy tắc áp dụng: Áp dụng mục 2 — `bus` yêu cầu thân xe khách dài, nhiều cửa sổ/hàng ghế lớn; `van` áp dụng cho thân hộp nhỏ/trung bình, kín, dùng chở người hoặc hàng hóa.
- Quyết định: Gán nhãn `van`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to ảnh lên 100% để đếm số khung cửa sổ và tỷ lệ chiều dài/chiều cao thân xe. Nếu góc chụp bị khuất không rõ chiều dài, đặt `review_state=needs_review` và ghi rõ lý do chưa thể xác định ranh giới giữa van lớn và bus nhỏ.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_033`, xe chở hàng kích thước nhỏ di chuyển ở làn trong.
- Dấu hiệu nhìn thấy: Phần cabin phía trước tách biệt rõ rệt với khoang chở hàng phía sau bằng một khe hở kết cấu; phía sau là thùng kín chuyên dụng chở hàng chứ không phải thân xe liền khối một thể.
- Quy tắc áp dụng: Áp dụng mục 2 — `truck` áp dụng khi có thùng hàng, sàn chở hàng hoặc khoang hàng tách biệt rõ ràng với cabin lái; `van` chỉ áp dụng khi thân xe là một khối hộp kín liền mạch.
- Quyết định: Gán nhãn `truck`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng to 100% để soi kỹ khớp nối giữa cabin và thùng sau xem có rãnh tách biệt hay khung xương liền. Nếu bị xe khác che khuất phần khớp nối, đánh dấu `review_state=needs_review` và giữ nguyên trạng thái nghi vấn.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, phương tiện ở sát rìa phải khung hình đang đi vào giao lộ.
- Dấu hiệu nhìn thấy khi phóng 100%: Phương tiện bị mép phải của ảnh cắt ngang mất phần đuôi xe (khoảng 35% thân xe); đồng thời phần đầu xe bị một phương tiện đi trước che khuất một phần cản trước. Tuy nhiên, vẫn nhìn rõ vòm mui xe, kính chắn gió và kiểu dáng sedan 4 chỗ đặc trưng.
- Giá trị `visibility`: `occluded`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `confident`
- Lý do: Hộp giới hạn được vẽ ôm sát chính xác phần diện tích thân xe nhìn thấy thực tế (không mở rộng hộp đoán phần bị cắt hay bị che). Dù bị cắt mép (`truncated`) và che khuất (`occluded`), nhưng các đặc trưng hình học nhìn thấy vẫn đủ bằng chứng phân lớp thành `car`.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 59 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
