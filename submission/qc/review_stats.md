# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 630; đối tượng sau duyệt: 629; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 628 | 628/630 = 99.7% | 630 box máy |
| edited (sửa) | 0 | 0/630 = 0.0% | 630 box máy |
| deleted (xoá) | 2 | 2/630 = 0.3% | 630 box máy |
| added (vẽ thêm) | 1 | 1/629 = 0.2% | 629 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 0.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 355 | 0 | 0 | 0 |
| bicycle | 8 | 0 | 0 | 0 |
| motorbike | 149 | 0 | 0 | 0 |
| car | 55 | 0 | 0 | 0 |
| bus | 3 | 0 | 0 | 1 |
| truck | 6 | 0 | 0 | 0 |
| three_wheeler | 52 | 0 | 2 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 47 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 40 | 8 | 49.4% (KTC 95%: 39–60%) | 83.0% (KTC 95%: 70–91%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 1/12 = 8.3% (KTC 95%: 1–35%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 0/3
  - extra: 1/3
  - missing: 0/3
  - loose: 0/3
  - lọt: img_023.jpg (wrong_class), img_031.jpg (extra), img_013.jpg (missing), img_009.jpg (loose), img_029.jpg (loose), img_028.jpg (wrong_class), img_003.jpg (missing), img_035.jpg (extra), img_017.jpg (missing), img_004.jpg (wrong_class), img_026.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
