# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 717; đối tượng sau duyệt: 682; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 561 | 561/717 = 78.2% | 717 box máy |
| edited (sửa) | 54 | 54/717 = 7.5% | 717 box máy |
| deleted (xoá) | 102 | 102/717 = 14.2% | 717 box máy |
| added (vẽ thêm) | 67 | 67/682 = 9.8% | 682 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 55.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 333 | 13 | 30 | 10 |
| bicycle | 5 | 1 | 2 | 3 |
| motorbike | 169 | 31 | 36 | 16 |
| car | 43 | 2 | 2 | 22 |
| bus | 2 | 0 | 1 | 3 |
| truck | 1 | 0 | 5 | 5 |
| three_wheeler | 8 | 7 | 26 | 8 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 65 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 57 | 31 | 8 | 64.8% (KTC 95%: 54–74%) | 87.7% (KTC 95%: 78–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 0/3
  - extra: 3/3
  - wrong_class: 2/3
  - loose: 3/3
  - lọt: img_029.jpg (missing), img_008.jpg (missing), img_024.jpg (missing), img_005.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
