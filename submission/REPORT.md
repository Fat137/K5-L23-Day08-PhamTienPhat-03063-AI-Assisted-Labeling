# Báo cáo Day 08 — 03063 - Phạm Tiến Phát

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | Không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 82%, 34 người gold). 0.15 là ngưỡng cao nhất còn giữ recall 82% (28/34), precision 65%; ở mặc định 0.3 recall rơi xuống 65% (22/34). r01 vs r02: lên 0.5 mất người nhỏ ở xa (img_015: 9 → 3 box person). |
| motorbike | YOLO26m | 0.1 | Gợi ý của threshold_sweep: recall 93% (13/14), precision 59%. |
| car | YOLO26m | 0.4 | Recall đứng yên 88% (7/8) từ 0.1 đến 0.5; 0.4 cho precision 64% (7/11) thay vì 54% ở 0.3 mà không mất box đúng. |
| bicycle / bus / truck | YOLO26m | 0.3 (mặc định) | Dưới 5 vật trong gold (bicycle 1, bus 1, truck 0): không đủ để chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | Recall 100% (7/7), precision 78% (7/9). r01/r02: YOLO26 không có lớp này, bỏ sót tuk-tuk (img_002) hoặc gọi xích lô là `bicycle` (img_032); r03: YOLOE bắt được (0.93, 0.92). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,car=0.4,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 837 box trên 28 ảnh (person 430, motorbike 274, car 57, three_wheeler 52, bicycle 13, truck 7, bus 4).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`, r04). Máy không có GPU, RAM đủ cho bản base; mask bám biên tốt khi box đúng (img_004: tuk-tuk xanh lá, người đứng bên phải). Chậm 11.8 s/ảnh trên CPU, chấp nhận được vì chỉ làm mask cho 3 vật. Không dùng SAM 3 vì cần quyền truy cập HuggingFace và GPU.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp xe ba bánh: tuk-tuk bị bỏ sót (img_002) hoặc gọi `car` với box chỉ phủ nửa xe (img_013, kéo theo mask SAM cụt), xích lô bị gọi `bicycle` (img_032). Sửa bằng cách lấy `three_wheeler` từ YOLOE và đặt YOLOE ưu tiên cao hơn khi gộp (IoU ≥ 0.6 thì box YOLO26 bị bỏ): lần gộp này loại 6 car, 6 truck chồng lên three_wheeler. Box lỏng của YOLOE và xích lô bị gộp 2 thành 1 (img_032) vẫn phải sửa tay khi review.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_002 pickup → `truck` 0.73 (guideline: bán tải là `car`). Điểm thấp mà đúng: img_032 xích lô bên trái 0.48 (YOLOE); img_015 người lái xe máy ở xa 0.27–0.42 (YOLO26), mất hết khi lên ngưỡng 0.5.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng được chọn và chấm trên cùng 4 ảnh gold (68 đối tượng), nên precision / recall ở trên là cận lạc quan. Lớp ít vật (bicycle, bus, truck) không có kết luận; car (8 vật) và three_wheeler (7 vật) chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 561 | 78.2% | 717 box máy |
| edited | 54 | 7.5% | 717 box máy |
| deleted | 102 | 14.2% | 717 box máy |
| added | 67 | 9.8% | 682 đối tượng sau duyệt |

(717 box máy = 734 box trong file nạp CVAT sau khi script loại các đối tượng nằm quanh 12 bẫy, vì đó là lỗi nhân tạo, không phải lỗi của máy. Trong deleted + added có 12 ca là **đổi lớp**, nên số vật máy sót thật ≈ 67 − 12 = 55.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **`three_wheeler`**: 26 box bị xoá trên 41 box máy (8 accepted, 7 edited) = 63% bị xoá. YOLOE với prompt `three_wheeler` bắt cả vật không phải xe ba bánh: xe buýt nền img_013 (#155, còn lọt tới QC), xe đẩy hàng và mặt tiền cửa hàng ở img_002 (r03). Ngưỡng 0.15 chọn ở Part 3 để đạt recall 100% trên gold nên chấp nhận nhiều box thừa. Sau đó là **`car`**: 22 box phải vẽ thêm so với 43 accepted (máy sót xe ở xa, bị che), và **`truck`**: 5/6 box máy bị xoá (pickup bị gọi `truck`, guideline xếp vào `car`).
- So với gold: precision / recall của **máy** 57/88 = 64.8% (KTC 95%: 54–74%) / 57/65 = 87.7% (KTC 95%: 78–94%) (mẫu số: 65 đối tượng gold, không tính 3 `ignore_region`). Lạc quan vì ngưỡng được chọn trên chính 4 ảnh này. Cứ 3 box máy thì khoảng 1 box thừa, và máy sót khoảng 1/8 vật: accepted 78% **không** có nghĩa máy tốt 78%, vì accepted chỉ đếm trên box máy đã vẽ, không thấy vật máy sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 = 66.7%** (KTC 95%: 39–86%), dưới ngưỡng 80%. Theo loại: box thừa 3/3, box lỏng 3/3, sai lớp 2/3, **máy bỏ sót 0/3**. Lọt: img_029, img_008, img_024 (missing), img_005 (wrong_class). Vì sao: tôi duyệt bằng cách đi qua từng box máy đã vẽ để sửa, **không ẩn nhãn máy và không quét ảnh trước** (bỏ bước 0 của checklist), nên chỉ thấy lỗi nằm trên box có sẵn; vật không có box thì mắt không dừng lại. Hai trong ba ảnh lọt bẫy missing (024, 029) nằm trong 6 ảnh dày xe máy: càng nhiều box máy, mắt càng bám theo box.
- Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt? Accepted 78% và bắt hết bẫy box thừa / box lỏng cho thấy tôi có kiểm từng box, nhưng 0/3 bẫy missing cho thấy đó là kiểm **theo box máy**, không phải kiểm **theo ảnh**. Đây đúng là lỗi "bỏ qua" của thiên kiến tự động hóa: máy không vẽ thì tôi không kiểm. Một số liệu cùng chiều: 20 box cao < 20 px (cảnh báo của `validate_labels.py`) vẫn còn trong nhãn sau duyệt, tức là tôi cũng "làm theo" máy với box nhỏ.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 20 phút / 4 ảnh = **5.0 phút/ảnh** vs duyệt 90 phút / 24 ảnh = **3.75 phút/ảnh** (nhanh hơn 25%). Theo đối tượng: gán tay 20 phút / 65 đối tượng = 0.31 phút/đối tượng; duyệt 90 phút / 682 đối tượng = 0.13 phút/đối tượng. Thiên lệch: (1) gold làm trước nên khi duyệt tôi đã quen kiểu cảnh và guideline; (2) ảnh batch đông hơn (28 đối tượng/ảnh so với 16 ở gold); (3) tốc độ duyệt đi kèm lọt 3/3 bẫy bỏ sót, tức là một phần thời gian "tiết kiệm" là do không quét ảnh.
- Kết luận: **có, cho `person` và `motorbike`** (phần lớn accepted, YOLO26 bắt tốt vật rõ), **không tin được cho `three_wheeler`** (63% box máy bị xoá) và **`truck`** (5/6 bị xoá). Dựa trên: accepted 561/717 = 78.2%, recall máy trên gold 87.7%, thời gian 3.75 so với 5.0 phút/ảnh. Điều kiện: phải duyệt theo ảnh (bước 0), vì máy vẫn sót khoảng 55 vật trong lô.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Mỗi ảnh: ẩn toàn bộ nhãn máy, tự đếm vật theo lớp và ghi số đếm ra giấy **trước** khi hiện nhãn; cuối ảnh so số box với số đếm. Kiểm được: bắt bẫy missing 3/3 ở lô sau.
  2. Mọi box `three_wheeler` và `truck` của máy phải được đếm bánh và đổi lớp nếu cần, không accept thẳng. Kiểm được: QC không còn lỗi `sai_lop` ở hai lớp này.
  3. Trước khi export, chạy `validate_labels.py --all` và xử lý hết cảnh báo box < 20 px (xoá hoặc gộp `ignore_region`). Kiểm được: 0 cảnh báo < 20 px.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có tài xế) | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương: thân, mui; 1 âm: tài xế) | Tài xế **vẫn bị tô một phần** (lốm đốm) dù có click âm | Khoang hở bên hông (nhìn thấy người và quán phía sau) bị tô kín thay vì bỏ ra như lỗ lớn (trái 4.2.2); mask 45.4% ảnh | `tuktuk_img013.jpg` |
| img_032 (xích lô phía trước) | SAM 2.1 (`sam2.1_b.pt`) | 4 (3 dương: đệm ghế, khung trước, bánh; 1 âm: người đứng cạnh) | Không có người lái; đệm ghế, mui gập và khung đều trong mask | Vùng giữa nan hoa của bánh bị tô kín; không lấn sang người đứng cạnh; mask 8.6% ảnh | `cyclo_img032.jpg` |
| img_004 (tuk-tuk lớn phía trước, YOLO26 bỏ sót) | SAM 2.1 (`sam2.1_b.pt`) | 5 (3 dương: đầu xe, ghế, bánh sau; 2 âm: người phía sau, người đứng bên phải) | Không có người; gồm mui, kính chắn gió, khoang ghế, bánh | Khoang ghế nhìn qua thành hở được tô kín; biên mui và bánh bám tốt; mask 13.1% ảnh | `tuktuk_img004.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc 4.2.2 (lỗ thủng) cho biết khoang hở lớn của tuk-tuk / xích lô phải bỏ ra, nhưng SAM không tự làm theo, phải thêm click âm vào khoảng hở. Quy tắc 4.2.4 (vùng chạm thuộc vật phía trước) đủ để nói tài xế thuộc `person`, nhưng SAM khó tách người ngồi **trong** xe chỉ bằng một click âm (img_013). Thiếu: v1.0 chưa nói kính chắn gió (đã chốt ở v1.1, CAL-04) và chưa nói vùng giữa nan hoa bánh xe có tính là lỗ không.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không làm.

## 6. Calibration (Part 6)

- Số quyết định: **8** (`calibration_decisions.csv`: 2 `loi_may`, 3 `cau_hoi_mo`, 2 `can_review`, 1 `bay_lot`); quan trọng nhất: **CAL-02** (box `three_wheeler` của YOLOE phải đếm bánh: lớp có tỉ lệ xoá cao nhất và còn lọt một xe buýt tới QC) và **CAL-06** (quy tắc < 20 px áp dụng cả với box máy: nguồn của 5/6 lỗi QC).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 2 (xe máy đỗ dày gần camera), câu 5 (kính chắn gió thuộc mask), câu 7 (không đặt conf tự chấp nhận); thêm 4.3 (đếm bánh box `three_wheeler`), 4.1.6 (đo bằng CVAT, áp dụng cả box máy), 4.1.3 (ngoại lệ cắt mép < 20 px). Câu 1, 3, 4, 6 chưa chốt vì không ghi nhận ca nào để làm ví dụ.
- Nguyên nhân gốc: chủ yếu **máy** (CAL-01, CAL-02: model không có lớp / prompt bắt nhầm) và **người duyệt chưa theo quy tắc** (CAL-06: giữ box máy < 20 px; CAL-08: bỏ bước 0); phần **guideline mơ hồ** là CAL-03, CAL-04, CAL-05, CAL-07.

## 7. QC và quyết định (Part 6)

- Mẫu: **7** ảnh (**2** ảnh `can_review` + **5** ngẫu nhiên, seed 42), 170 đối tượng; người kiểm: phiếu `qc_review.csv` do trợ lý AI điền từ ảnh `work/qc/review/`, sau đó tôi xem lại và đồng ý (cột reviewer = `AI-assist, 03063 xác nhận`) (**hạn chế**: tôi vừa là người duyệt lô vừa là người xác nhận QC, nên kiểm tra không độc lập).
- Lỗi theo **đối tượng**: 6/170 = 3.5% · theo **ảnh**: 4/7 = 57.1% (ngưỡng Release ≤ 2%)
- Top lỗi: **thừa** 5 (box < 20 px còn giữ: img_014 #162 person 19 px, img_022 #315 car 18 px, img_034 #633 / #639 / #643 cao 16–18 px); **sai lớp** 1 (img_013 #155: xe buýt nền bị gán `three_wheeler`). Không ghi nhận lỗi bỏ sót, sai vị trí / kích thước, sai thuộc tính trong mẫu; lô là detection nên không có lỗi mask.
- **Giới hạn mẫu nhỏ**: 4 ảnh lỗi trong 7 ảnh mẫu, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 84% (`qc_report_auto.md`). Với 7 ảnh, kể cả 0 lỗi cũng không chứng minh được ngưỡng 2%; ở đây số lỗi đủ để kết luận lô **không** đạt.
- Script kiểm được / không kiểm được: script kiểm được chiều cao box < 20 px (`validate_labels.py` phát hiện 20 box trong toàn lô), lớp có trong taxonomy, số đếm accepted / deleted / added, tỉ lệ lỗi và cận trên. Script **không** kiểm được box có ôm sát vật thật không, lớp có đúng thực tế không (xe buýt gán `three_wheeler` chỉ phát hiện bằng mắt), vật mà cả người duyệt lẫn máy cùng bỏ sót ngoài gold, và việc tôi có làm bước 0 của checklist hay không (chỉ thấy gián tiếp qua bẫy missing 0/3).

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): **không Hold** vì ảnh không hỏng và các câu hỏi guideline còn mở (1, 3, 4, 6) không gặp trong lô; **Rework** vì ảnh lỗi 4/7 = 57.1% > 2%, lỗi lặp có hệ thống (thừa: 5/6 lỗi, cùng một quy tắc < 20 px) và bắt bẫy 8/12 = 66.7% < 80% nên chính lượt duyệt chưa đáng tin.
Nếu Rework / Hold, việc cần làm tiếp: (1) xoá hoặc gộp `ignore_region` 20 box < 20 px mà `validate_labels.py` liệt kê và #322 ở img_023 (CAL-07); (2) sửa img_013 #155 thành `bus`, đếm bánh lại mọi box `three_wheeler`; (3) duyệt lại cả lô theo bước 0 (ẩn nhãn, đếm trước), thêm `ignore_region` cho 6 ảnh dày xe máy (005, 018, 021, 024, 028, 029); (4) export `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, ghi vòng 2 vào báo cáo. Vòng 2 chưa làm.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): có, trợ lý lập trình AI trong VS Code. AI làm: sửa môi trường (cài `transformers` còn thiếu, chép model vào `models/`), chạy các script Part 2–6, nháp nhận xét `model_log.csv` từ ảnh `viz/` (tôi xem lại ảnh và đồng ý), chọn điểm click SAM, điền phiếu QC `qc_review.csv` (tôi xem lại và đồng ý), nháp `calibration_decisions.csv`, guideline v1.1 và REPORT. Tôi tự làm: gán tay 4 ảnh gold và duyệt 24 ảnh batch trong CVAT, chọn phương án ngưỡng (thêm person 0.15, car 0.4) và quyết định xoá 2 box < 20 px ở gold. AI không mở `traps_key.json` trước khi tôi export.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Có, một phần.** Không chạy model nào lên 4 ảnh gold và không xem pre-label trước khi gán xong; nhưng trong lúc gán tôi gửi ảnh chụp img_001 cho AI nhận xét 3 lần (AI chỉ ra box xe máy đỗ chưa ôm sát, thiếu box `bicycle`, dùng `ignore_region` sai và thiếu `ignore_region` cho hàng xe đỗ) và hỏi AI lớp của một chiếc xe buýt trong ảnh gold. Vì vậy gold img_001 là nhãn người đã được AI góp ý, không hoàn toàn độc lập.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, không có GPU (PyTorch CPU; YOLO26m 1.0–1.4 s/ảnh, SAM 2.1 11.8 s/ảnh); CVAT 2.74.1 chạy bằng Docker. Vướng: `check_env.py` báo thiếu `transformers`; model tải về thư mục ngoài repo nên phải chép vào `models/`.
- Dùng cache `data/cache/` thay vì tự chạy? không, tự chạy mọi model trên CPU.
- Lỗi / bug script đã gặp (nếu có): không gặp.
- Một điều bạn làm khác lần sau: ẩn nhãn máy và tự đếm vật trước mỗi ảnh ngay từ ảnh đầu tiên, thay vì đi theo box máy.
