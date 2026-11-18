# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 7 ảnh (2 ca khó + 5 ngẫu nhiên), 170 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/170 = 3.5% | 170 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 4/7 = 57.1% | 7 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 561/717 = 78.2% / 54/717 = 7.5% / 102/717 = 14.2% | 717 box máy |
| Máy: added | 67/682 = 9.8% | 682 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 5 | img_014.jpg, img_022.jpg, img_034.jpg |
| sai_lop | 1 | img_013.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 4 ảnh lỗi trong 7 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **84%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 57.1% > 2%
- lỗi hệ thống: 'thua' chiếm 5/6 lỗi
- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
