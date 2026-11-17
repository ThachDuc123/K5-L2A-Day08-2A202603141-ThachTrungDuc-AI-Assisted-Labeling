# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603141 - Thạch Trung Đức

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Gán `motorbike` nếu có sàn để chân rộng, không có bàn đạp hoạt động; gán `bicycle` nếu có bàn đạp chính. | `img_003.jpg`, `img_036.jpg` | `CAL-02` |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Nếu đỗ gần camera, tách được từng xe và cao >= 20 px thì gán từng `motorbike`; chỉ dùng 1 `ignore_region` cho cụm >= 5 vật sát nhau ở xa/mờ hoặc cao < 20 px. | `img_018.jpg`, `img_036.jpg` | `CAL-06` |
| 3 | Hàng nhô ra trên xe đạp / ba gác: box và mask xe có gồm hàng không? | Giỏ gắn cố định vào khung tính vào box xe (giữ đủ bánh sau/khung xe như ở `#549`); hàng rời/thúng nhô ra > 20% chiều dài xe không tính vào box và loại khỏi mask xe. | `img_031.jpg`, `img_006.jpg` | `CAL-03` |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng >= 20% có hợp lý không? | Giữ ngưỡng nhìn thấy >= 20% diện tích cơ thể (ưu tiên nhìn thấy phần đầu hoặc thân); nếu bị che khuất chỉ thấy góc chân/tay nhỏ < 20% thì không gán. | `img_012.jpg`, `img_020.jpg` | `CAL-05` |
| 5 | Kính chắn gió / cửa sổ xe: thuộc mask xe hay không? | Kính chắn gió thuộc mask xe; ngược lại khoảng hở không có kính ở khung tuk-tuk/xích lô (nhìn xuyên ra nền hoặc người ngồi sau) nếu > 5% diện tích vật phải loại khỏi mask. | `img_013.jpg`, `img_004.jpg` | `CAL-04` |
| 6 | Box máy chồng nhau quá nhiều trong ảnh đêm/đông: gộp hay giữ? | Không giữ nhiều box trùng nhau cùng 1 vật (như `three_wheeler` trùng lặp ở `img_003.jpg`, `img_035.jpg`) và không gộp nhiều xe rõ vào 1 box; chỉ gộp `ignore_region` khi đạt điều kiện cụm >= 5 vật mờ không tách được. | `img_003.jpg`, `img_035.jpg` | `CAL-01` |
| 7 | Có đặt mức `conf` tối thiểu để chấp nhận box máy không cần sửa không? | **Không** đặt ngưỡng `conf` tự động chấp nhận. Box `conf` cao vẫn có thể là bẫy (`truck 0.66` ở `img_031.jpg`) hoặc sai cạnh (`person 0.79` cắt chân ở `img_031.jpg`); mọi box đều phải kiểm bằng mắt trước khi giữ. | `img_031.jpg`, `img_035.jpg` | `CAL-05` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003.jpg`, `img_014.jpg`, `img_035.jpg` | Quang gánh bán hàng rong, bàn gỗ, bụi cây hoặc nhiều box `three_wheeler` đè lên 1 xe | YOLOE (`three_wheeler`, ngưỡng `0.15` từ sweep) sinh nhiều box trùng trên 1 xe (`#28/#32` ở `img_003`, `#638/#639` ở `img_035`) và gán nhầm quang gánh/bàn (`#194-#196` ở `img_014`) | Không dùng ngưỡng `0.15` trơn không lọc trùng; xóa hết box quang gánh/bàn/bụi cây và giữ đúng 1 box `three_wheeler` cho mỗi xe | Mục 2 (`three_wheeler`) & Mục 4.1 (`CAL-01`) |
| `img_031.jpg`, `img_035.jpg` | Tranh vẽ người trên tường, mũ cam trên kệ, giá treo áo khoác | Box máy và bẫy `extra` (`#552 truck 0.66` trên áo khoác `img_031`, `#548 person 0.34` trên mũ cam, `#640 person 0.45` trên tranh tường `img_035`) | Thực hiện quét mù 10 giây đầu trên mỗi ảnh, xóa 100% vật không phải người/xe thật và kiểm 4 cạnh cho box cắt cụt (`#549`, `#550`) | Mục 4.1.7 (hình in/tranh vẽ) & Quy trình duyệt (`CAL-05`) |
| `img_014.jpg`, `img_018.jpg`, `img_036.jpg` | Người/xe máy ở xa cao < 20 px | YOLO26m ở ngưỡng `person=0.20` (và người gán ở `img_036`) giữ lại nhiều box `11.6–18.9 px` | Xóa box đơn lẻ < 20 px; chỉ giữ `ignore_region` cho cụm >= 5 vật sát nhau không tách được | Mục 4.1.6 (vật quá nhỏ < 20 px) (`CAL-06`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 7 câu hỏi mở; nâng ngưỡng/lọc trùng `three_wheeler` (`YOLOE`), bổ sung quy tắc kiểm tranh tường/quang gánh/box `< 20 px`, và siết quy trình quét mù chống thiên kiến | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04`, `CAL-05`, `CAL-06` |
