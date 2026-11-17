# Thông tin học viên

- Họ và tên: Thạch Trung Đức
- MSHV: 2A202603141
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: ThachDuc123
- Repo bài nộp (URL): https://github.com/ThachDuc123/K5-L2A-Day08-2A202603141-ThachTrungDuc-AI-Assisted-Labeling
- Commit chốt bài (hash): 5f2039d (xem thêm commit tổng kết STUDENT.md ở HEAD)
- Máy: Windows 11 x64, CPU only (không GPU), Python 3.11.9, torch 2.15.0+cpu
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 628 / 0 / 2 (mẫu số: 630 box máy); added: 1 (mẫu số: 629 đối tượng)
- So với gold: precision / recall của **máy** 49.4% (39/79) / 83.0% (39/47)
- Bắt bẫy: 1/12 = 8.3%
- Thời gian: gán tay 5.50 phút/ảnh (22 phút / 4 ảnh); duyệt có pre-label 0.17 phút/ảnh (4 phút / 24 ảnh)
- Lỗi QC: 100.0% theo ảnh (5/5), 15.2% theo đối tượng (19/125)
- **Quyết định:** Rework — lý do một dòng: ảnh có lỗi 5/5 = 100.0% > 2%, lặp lỗi thừa box (14/19 lỗi) và bắt bẫy 1/12 = 8.3% < 80% do duyệt nhanh 4 phút (automation bias).
