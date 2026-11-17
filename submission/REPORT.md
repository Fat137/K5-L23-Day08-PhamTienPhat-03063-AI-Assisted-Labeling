# Báo cáo Day 08 — <<MSHV - Họ và tên>>

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
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
