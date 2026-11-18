# Thông tin học viên

- Họ và tên: Phạm Tiến Phát
- MSHV: 03063
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Fat137
- Repo bài nộp (URL): https://github.com/Fat137/K5-L23-Day08-PhamTienPhat-03063-AI-Assisted-Labeling
- Commit chốt bài (hash): c1b56c6
- Máy: Windows 11, không có GPU (PyTorch CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 78.2% / 7.5% / 14.2% (561 / 54 / 102, mẫu số: 717 box máy); added: 9.8% (67, mẫu số: 682 đối tượng)
- So với gold: precision / recall của **máy** 64.8% (57/88) / 87.7% (57/65)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 5.0 phút/ảnh (20 phút / 4 ảnh); duyệt có pre-label 3.75 phút/ảnh (90 phút / 24 ảnh)
- Lỗi QC: 57.1% theo ảnh (4/7), 3.5% theo đối tượng (6/170)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 4/7 > 2%, lỗi thừa lặp lại (box < 20 px) và bắt bẫy 8/12 < 80%
