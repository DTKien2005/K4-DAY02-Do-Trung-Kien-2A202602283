# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Đỗ Trung Kiên<br>
**MSSV:** 2A202602283<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038`
- Số vật thể thực tế: `59`
- Mã SHA-256 của gói YOLO của bạn: `b6f38ceadaf89ef5a67561cfc406da57c8b4205aac079afdc58961456a8883a2`
- Mã SHA-256 của gói CVAT gốc của bạn: `8dbd645de93f9a371dd10e74fde0b7b1f073119d360e93af7f10f044a6685f2e`
- Nguồn đối chiếu: `bộ tham chiếu do người hướng dẫn thực hành cấp`
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Mã lần phát và thời điểm nhận bộ tham chiếu: `Nhận bộ tham chiếu chuẩn phút 180 từ Lab Coach`

Toàn bộ quá trình gán nhãn 4 bức ảnh và thiết lập 3 thuộc tính được tôi thực hiện độc lập hoàn toàn trên CVAT. Sau khi hoàn thành, tôi đã lưu và xuất ra hai gói dữ liệu riêng (`day2-my-export.zip` và `day2-native-export.zip`), kiểm tra mã băm SHA-256 và chạy thử nghiệm huấn luyện trước khi nhận và mở bộ nhãn đối chiếu từ Lab Coach.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_008` / xe sedan làn giữa | `car` | Thân xe sedan 4 chỗ, vòm mui thấp, kính chắn gió và cụm đèn nhận diện rõ | Áp dụng mục 2: ô tô con loại sedan chở người |
| `drive_033` / xe tải chở hàng | `truck` | Cabin lái tách rời hoàn toàn với khoang thùng chở hàng phía sau qua khe kết cấu | Áp dụng mục 2: xe có thùng hàng hoặc sàn chở hàng riêng biệt |
| `drive_038` / xe chở khách nhỏ | `van` | Khối thân hộp kín liền khối trần cao, khoảng 3–4 hàng cửa sổ bên hông (dạng 16 chỗ) | Áp dụng mục 2: xe van thân hộp nhỏ/trung bình, không phải xe buýt lớn |

Lớp (`class`) trả lời cho câu hỏi đối tượng là gì (bản chất vật thể), trong khi thuộc tính (`attribute`) mô tả tình trạng quan sát của vật thể trong khung hình. Ví dụ: một chiếc xe ô tô con bị khuất một nửa thân sau do mép ảnh cắt ngang thì lớp của nó vẫn là `car`, nhưng thuộc tính `boundary` là `truncated` và `visibility` là `occluded`. Lớp phục vụ cho việc phân loại danh mục, còn thuộc tính giúp đánh giá độ tin cậy và mức độ đầy đủ của dữ liệu quan sát được.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Hộp xe van làn phải bị vẽ bao trùm bóng đổ trên mặt đường | hình học | Phóng to ảnh 100% trong bước tự kiểm tra | Thu nhỏ mép dưới hộp ôm sát lốp xe tiếp xúc mặt đường theo quy tắc hộp giới hạn ôm sát vật thể |
| Xe bán tải bị phân vân giữa ô tô con và xe tải | lớp | Soi kỹ kết cấu cabin và mục đích sử dụng | Xác định là bán tải gia đình cabin kép dùng như xe con, gán lớp `car` theo quy tắc mục 2 |

- Số hộp `needs_review` trước và sau khi kiểm: Trước khi kiểm có 3 hộp; sau khi rà soát phóng to 100% đã xác định dứt điểm và đưa về 0 hộp `needs_review`.
- Một quyết định chưa đủ bằng chứng và cách tôi xin hỗ trợ: Ở ảnh `drive_033`, một phương tiện ở hậu cảnh rất xa bị mờ và che khuất, không thể phân biệt chính xác giữa SUV và xe van nhỏ. Tôi đã ghi nhận tình huống vào nhật ký và xin ý kiến Lab Coach dựa trên kích thước tương quan với làn đường.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.263805 0.571422 0.122547 0.076594`
- Tên lớp và tọa độ điểm ảnh `xyxy`: Lớp `car` (mã 0), tọa độ điểm ảnh góc trên-trái và dưới-phải `xyxy = [129.6, 341.2, 208.1, 390.2]`

Định dạng YOLO chỉ kiểm tra tính hợp lệ về mặt cú pháp số học (gồm 5 giá trị số thực chuẩn hóa trong khoảng 0 đến 1). Trình đọc không thể biết được nội dung ngữ nghĩa bên trong: dòng nhãn vẫn có thể bị gán sai lớp (ví dụ xe van nhưng gán mã 0 là `car`), sai phạm vi (gán cả người đi bộ hoặc xe máy) hoặc sai hình học (vẽ hộp quá rộng chứa nhiều nền hoặc cắt mất một phần xe).

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: Mô hình dự đoán được chính xác các xe ô tô con ở gần với độ tin cậy trên 0.6, nhưng có xu hướng bỏ sót xe ở cự ly xa và nhầm lẫn giữa `van` và `car`. Dự đoán đó gợi ý tôi cần kiểm tra lại độ đồng nhất trong việc gán nhãn các xe ở khoảng cách xa và ranh giới phân biệt giữa xe van gia đình và xe ô tô con cỡ lớn (SUV/MPV). Để bác bỏ nhận định này cần kiểm tra mô hình trên một tập thẩm định lớn hơn với nhiều góc máy và mật độ giao thông khác nhau.

Tập dữ liệu chỉ có 4 bức ảnh (3 ảnh train, 1 ảnh val), quá nhỏ để mô hình học được đặc trưng tổng quát hóa. Việc huấn luyện thử nghiệm này chỉ nhằm mục đích kiểm tra tính thông suốt của đường ống dữ liệu và tìm lỗi gán nhãn thô, hoàn toàn không đại diện cho năng lực triển khai thực tế.

## 6. Đối chiếu nhãn

- Số hộp ghép được: `46`
- IoU trung bình và trung vị: IoU trung bình đạt `0.8697` (~87%), IoU trung vị đạt `0.8958` (~90%)
- Mức đồng thuận lớp: `73.9%` (`0.7391`)
- Số hộp phía tôi không ghép được: `13`
- Số hộp phía đối chiếu không ghép được: `4`
- Một điểm khác biệt cụ thể: Ở xe chở khách tầm trung tại ảnh `drive_038`, tôi gán là `van` trong khi bộ tham chiếu gán là `bus`.
- Quy tắc hoặc hành động sửa phát sinh: Làm rõ quy ước đếm số khung cửa sổ hoặc ngưỡng chiều dài thân xe để phân định rõ ràng giữa xe van chở khách và xe buýt nhỏ.

Mức đồng thuận cao chỉ phản ánh tính nhất quán và khả năng tái lập giữa hai bên gán nhãn theo cùng một cách hiểu. Hai bên hoàn toàn có thể cùng đạt đồng thuận 100% nhưng vẫn cùng sai nếu cả hai đều hiểu sai quy tắc hoặc cùng mắc một lỗi hệ thống (ví dụ cùng bỏ sót một nhóm phương tiện chạm mép ảnh).

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Điểm tôi đánh giá là minh chứng thuyết phục nhất trong bài là độ chồng khít IoU trung bình ~87% kết hợp với tính đồng nhất tuyệt đối giữa hai gói xuất YOLO và CVAT XML — cả 59 vật thể đều có đủ 3 thuộc tính, cross-format IoU tối thiểu đạt 0.9999. Điều tôi vẫn chưa chắc chắn sau bài là ranh giới về mức độ che khuất: khi xe bị che khuất trên 70% thân xe, tôi không rõ nên giữ hộp gán nhãn đó hay bỏ qua, vì quy tắc hiện tại chưa đề cập ngưỡng cụ thể.
