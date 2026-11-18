# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03063 - Phạm Tiến Phát

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Gán **từng xe** khi thấy được ít nhất một bộ phận nhận dạng riêng (bánh, đầu xe, yên) **và** còn thấy ≥ 20% xe. Phần cuối hàng không tách được từng chiếc, từ 5 xe trở lên: **một** `ignore_region` bao phần đó. Box các xe chồng nhau là bình thường. | img_006 | CAL-03 |
| 5 | Kính chắn gió / cửa sổ có thuộc mask xe không? | **Có**: kính là một phần thân xe, kể cả khi nhìn xuyên thấy người / nội thất. Khoảng **hở không có kính** (thành bên tuk-tuk, xích lô) theo quy tắc lỗ thủng 4.2.2. | img_004 (`masks/tuktuk_img004.jpg`) | CAL-04 |
| 7 | Mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | **Không đặt.** Box điểm cao vẫn sai (img_002: bán tải bị gọi `truck` 0.73). `conf` chỉ dùng để xếp thứ tự nhìn, không bao giờ là lý do bỏ qua bước kiểm. | img_002 | CAL-05 |

**Chưa chốt** (không ghi nhận ca nào lúc gán / duyệt / QC nên không có ảnh ví dụ; giữ nguyên là câu hỏi mở, không tính là thiếu quy tắc cho lô này):

- Câu 1 (xe đạp điện): không ghi nhận xe đạp điện nào.
- Câu 3 (hàng nhô ra): không ghi nhận xe chở hàng nhô quá ~20% chiều dài xe.
- Câu 4 (người trong xe ba bánh có mui): các ca gặp (img_013 tài xế, img_023 khách ngồi trong tuk-tuk) đều thấy rõ > 20% người, chưa thử được ngưỡng 20%.
- Câu 6 (box máy chồng nhau ảnh đêm): chưa xem riêng; ảnh đêm trong mẫu QC (img_022) không có lỗi loại này.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013 | Xe buýt đỏ-trắng ở nền | YOLOE (prompt `three_wheeler`) gán `three_wheeler` (#155) | `bus` | **4.3:** mọi box `three_wheeler` của máy phải **đếm bánh** trước khi giữ; xe ≥ 4 bánh hoặc xe buýt không bao giờ là `three_wheeler` (CAL-02) |
| img_014, img_034 (gold: img_001) | Người / xe máy ở xa cao 16–19 px | YOLO26 vẽ box; người duyệt giữ nguyên | Xoá | **4.1.6:** đo chiều cao bằng thông số box trong CVAT, không ước lượng bằng mắt; quy tắc < 20 px áp dụng **cả với box máy có sẵn** (CAL-06) |
| img_023 | Ô tô bị cắt ở mép trên, phần nhìn thấy chỉ cao 15 px (#322) | YOLO26 gán `car` | Không vẽ | **4.1.3:** ngoại lệ: nếu **phần nhìn thấy** sau khi bị cắt mép cao < 20 px thì không vẽ, dù xe thật lớn (CAL-07) |

Ca **không** đổi guideline (chỉ đổi recipe / quy trình):

- CAL-01 (img_002, img_013, img_032): YOLO26 không có lớp xe ba bánh. v1.0 mục 2 đã đúng; sửa bằng cách lấy `three_wheeler` từ YOLOE và cho YOLOE ưu tiên khi gộp.
- CAL-08 (img_008, img_024, img_029, img_005): bẫy lọt do người duyệt bỏ bước 0 (quét ảnh trước). Lỗi quy trình, không phải guideline mơ hồ.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 5, 7 của mục 6; thêm kiểm đếm bánh cho box `three_wheeler` của máy (4.3); quy tắc < 20 px áp dụng cả box máy, đo bằng CVAT (4.1.6); ngoại lệ cắt mép (4.1.3) | CAL-02, CAL-03, CAL-04, CAL-05, CAL-06, CAL-07 |
