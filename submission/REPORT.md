# Báo cáo Day 08 — 2A202603141 - Thạch Trung Đức

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | `person=0.20`; `bicycle`, `motorbike`, `car`, `bus`, `truck` giữ mặc định `0.30` (theo `submission/qc/threshold_sweep.md`) | `r01`, `r02`, `r05`: YOLO26m bắt chắc `person`, `motorbike`, `car`, `bus`, `truck`; conf `0.5` (`r02`) làm mất người ở xa (`img_011` giảm `37→22` box), còn `0.05` (`r05`) sinh 2172 box nhiều rác (`banana`×134, `umbrella`×66). Quét ngưỡng trên 4 ảnh gold (`49` đối tượng) gợi ý `person=0.20` (recall `93%`, precision `32%` trên `14` vật gold); `motorbike` (`14` vật, tối đa `71%`), `car` (`8` vật, tối đa `88%`) và các lớp `< 5` vật (`bicycle=3`, `bus=1`, `truck=0`) giữ mặc định `0.30`. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`, prompt: `three_wheeler`) | `0.15` (từ `threshold_sweep.md`, `7` vật gold, recall `100%`, precision `78%`) | `r03`, `r06`: YOLO26 thuộc tập lớp COCO đóng nên không có `three_wheeler` (bỏ sót tuk-tuk ở `img_002` hoặc gọi nhầm là `car` ở `img_013`). YOLOE open-vocabulary nhận diện được tuk-tuk/xích lô từ prompt chữ (`171` box trên 36 ảnh ở `r06`), nhưng ở ngưỡng thấp `0.15` sinh nhiều box trùng lặp và box ma vào quang gánh/bụi cây nên bắt buộc review kỹ. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (`sam2.1_b.pt` — SAM 2.1 Base, có đối chiếu với `mobile_sam.pt`) và lý do: chạy ổn định trên máy Windows CPU-only (`~5.58s/ảnh` ở `r04`), không cần tải model nặng hơn hay xin quyền truy cập như SAM 3; giữ biên khung kim loại và lốp xe liền mạch thành `1 polygon` tốt hơn `mobile_sam.pt` (vốn hay rách mảng tối ở bánh xe).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?
  - Lỗi **thiếu lớp trong tập nhãn đóng** (YOLO26 không có lớp `three_wheeler` nên bỏ sót tuk-tuk `img_002.jpg` hoặc gọi là `car` `0.42` ở `img_013.jpg`): tăng/giảm `--conf` (`r01` `0.25` vs `r02` `0.50`) vẫn không sửa được tên lớp. Phải sửa bằng **đổi sang YOLOE** (`r03`, `r06`, đặt `yoloe_all` trước `y26_all` khi `merge_prelabels` để `three_wheeler` thắng `car`/`truck` tại `IoU >= 0.6`) hoặc sửa nhãn tay trong CVAT.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: `img_002.jpg` (`r03`), YOLOE bắt `three_wheeler` `0.93` nhưng box lỏng ôm thừa khoảng mái che trên; trong QC batch, `#639`/`#638` trùng nhau trên cùng xích lô ở `img_035.jpg`, `#196 three_wheeler 0.63` ở `img_014.jpg` nhận nhầm quang gánh hàng rong.
  - Điểm thấp mà đúng: trong `r04` (`img_013.jpg`), box `car 0.42` khoanh đúng vị trí chiếc tuk-tuk (chỉ sai lớp); `img_035.jpg` chiếc xích lô có `conf 0.38` vẫn khoanh đúng xe.
- Giới hạn: chọn ngưỡng trên chính 4 ảnh gold (`49` đối tượng, trong đó `bicycle=3`, `bus=1`, `truck=0` có `< 5` vật) nên số đo trên gold là lạc quan (`in-sample`). Hơn nữa việc chọn `person=0.20` và `three_wheeler=0.15` theo tiêu chí recall `>= 90%` trên 4 ảnh gold đã kéo nhiều box giả/trùng vào lô batch (`precision` của `person` trên gold chỉ `32%`).

## 2. AI error profile (từ `submission/qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 628 | 99.7% | 630 box máy |
| edited | 0 | 0.0% | 630 box máy |
| deleted | 2 | 0.3% | 630 box máy |
| added | 1 | 0.2% | 629 đối tượng sau duyệt |

*(Đã loại 23 đối tượng quanh các bẫy khỏi bảng trên; trong `deleted + added` có `1` trường hợp đổi lớp nên số vật máy sót thật ≈ `0` theo đối chiếu trên batch, chủ yếu do người duyệt giữ nguyên gần hết box máy).*

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  - Trên số liệu sau duyệt, chỉ có **`three_wheeler`** (`deleted = 2`, `accepted = 52`) và **`bus`** (`added = 1`, đổi lớp) có thao tác sửa/xóa; tuy nhiên khi soi trực tiếp ở bước QC mẫu 5 ảnh (Mục 7), **`three_wheeler`** vẫn là lớp tệ nhất về lỗi hệ thống `thua` (nhận nhầm quang gánh/bàn gỗ/bụi cây ở `img_014.jpg`, `img_003.jpg` và sinh nhiều box chồng lặp ở `img_003.jpg`, `img_035.jpg` do ngưỡng `0.15`).
- So với gold: precision / recall của **máy** là **49.4%** (39/79, KTC 95%: 39–60%) / **83.0%** (39/47, KTC 95%: 70–91%) (mẫu số: `47` đối tượng gold trên `4` ảnh gold không tính `ignore_region`; TP=39, FP=40, FN=8).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **1/12 = 8.3%** (KTC 95%: 1–35%, dưới rất xa ngưỡng đạt `80%`).
  - Loại nào lọt: `extra` bắt `1/3` (lọt `img_031.jpg`, `img_035.jpg`), `wrong_class` bắt `0/3` (lọt `img_023.jpg`, `img_028.jpg`, `img_004.jpg`), `missing` bắt `0/3` (lọt `img_013.jpg`, `img_003.jpg`, `img_017.jpg`), `loose` bắt `0/3` (lọt `img_009.jpg`, `img_029.jpg`, `img_026.jpg`).
  - Vì sao (bạn đã bỏ bước nào của checklist)? Đã bỏ qua gần như toàn bộ quy trình 4 bước (không ẩn box máy để quét ảnh tìm vật sót, không rà từng box để kiểm sai lớp và 4 cạnh viền, không lọc bỏ các box trùng lặp/box giả của YOLOE ở ngưỡng `0.15`), chỉ duyệt lướt trong **4 phút** cho 24 ảnh (`0.17 phút/ảnh ≈ 10 giây/ảnh`).
- `accepted` cực cao (`628/630 = 99.7%`) mà bắt bẫy cực thấp (`1/12 = 8.3%`) nói gì về cách bạn duyệt?
  - Đây là ví dụ điển hình của **thiên kiến tự động hóa (automation bias)**: vì tin tưởng hoàn toàn vào `725` box pre-label sẵn có trên màn hình và bấm qua ảnh quá nhanh (`10 giây/ảnh`), người duyệt giữ lại `99.7%` box máy nhưng lại để lọt `11/12` bẫy và giữ nguyên cả các lỗi rõ ràng như tranh vẽ tường (`#640` `img_035.jpg`) hay giá treo áo (`#552` `img_031.jpg`). Con số `accepted = 99.7%` không hề phản ánh lô nhãn sạch.

## 4. Máy có đáng dùng không?

- Thời gian (từ `submission/time_log.csv`): gán tay `22 phút / 4 ảnh = 5.50 phút/ảnh` (`gold_manual`) vs duyệt có pre-label `4 phút / 24 ảnh = 0.17 phút/ảnh` (`batch_review`). Thiên lệch: việc duyệt chỉ `0.17 phút/ảnh` là quá nhanh dẫn đến bỏ qua hầu hết lỗi máy và bẫy (`8.3%`), nên không thể dùng thời gian `4 phút` này làm chuẩn tốc độ đạt chất lượng; ngoài ra 4 ảnh gold được gán trước nên đã quen bối cảnh đường phố.
- Kết luận: **Có đáng dùng pre-label làm gợi ý cho `person`, `motorbike`, `car`, `bus`, `truck`**, nhưng **không thể tin và bấm duyệt nhanh `4 phút`**; với `three_wheeler` (ở ngưỡng `0.15`) phải nâng ngưỡng hoặc bắt buộc lọc trùng + kiểm tay 100%, vì máy hiện cho precision trên gold chỉ `49.4%` (`40` FP) dù recall cao (`83.0%`).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quy tắc 10 giây quét mù & thời lượng tối thiểu:** Ở mỗi ảnh trong CVAT, nhấn ẩn toàn bộ box pre-label trong 10 giây đầu để tự đếm vật trước rồi mới bật lên đối chiếu; dành tối thiểu **1.5–2 phút/ảnh** (~36–48 phút cho 24 ảnh) (kiểm tra bằng `time_log.csv` và bắt đạt `>= 10/12` bẫy = `>= 80%`).
  2. **Soi riêng lớp `three_wheeler` qua nút Filters trước khi lưu:** Bật bộ lọc lớp `three_wheeler` trên thanh công cụ CVAT ở từng ảnh để xóa hết box trùng đè lên nhau và box nhận nhầm vật không phải xe (quang gánh, bàn, bụi cây) (kiểm tra bằng `0` lỗi `thua` do box trùng/nhầm đồ vật trong vòng QC).
  3. **Kiểm 4 cạnh & rà box `< 20 px` bằng `validate_labels.py`:** Không chấp nhận box chỉ dựa vào nhìn lướt (kiểm 4 cạnh mọi box `bicycle`, `three_wheeler` và xóa các box đơn lẻ `< 20 px`), chạy `validate_labels.py --all` để xác nhận hết cảnh báo kích thước trước khi chốt release.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 4 clicks (`2 pos`, `2 neg`) | Có dính phần chân/dép và một phần đầu của người lái ngồi bên trong; không dính hàng hóa; kính chắn gió nằm trong mask | Diện tích mask `309203 px` (`44.3%` ảnh), `1 polygon`; giữ liền mạch khung kim loại và lốp bánh xe, nhưng vẫn rò nhẹ lên áo người ngồi sau qua khoảng hở khung hông dù đã đặt `neg` click | [tuktuk_img013.jpg](masks/tuktuk_img013.jpg) |
| `img_013.jpg` *(đối chiếu SAM 1)* | MobileSAM (`mobile_sam.pt`) | 4 clicks (`2 pos`, `2 neg` — cùng tọa độ với `sam2.1_b.pt`) | Loại phần thân/quần người lái tốt hơn ở vùng giữa, không lem ra người ngồi sau qua khoảng hở khung; vẫn dính cổ chân và một phần má người lái | Diện tích mask `235438 px` (`33.7%` ảnh), tách thành `2 polygon`; chạy nhanh trên CPU nhưng bị rách/mất mảng ở lốp bánh sau, bậc lên xuống và lem nhẹ sang cánh tay người phía sau | [tuktuk_img013_sam1.jpg](masks/tuktuk_img013_sam1.jpg) |
| `img_004.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 5 clicks (`3 pos`, `2 neg`) | Không dính 2 người đứng cạnh; có dính thùng nhựa đỏ đặt trên sàn xe | Diện tích mask `92665 px` (`13.3%` ảnh), `1 polygon`; tách sạch người đứng sát bên trái/phải, nhưng bị sót bảng hiệu `TAXI` trắng trên nóc mui | [tuktuk_img004.jpg](masks/tuktuk_img004.jpg) |
| `img_023.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 4 clicks (`2 pos`, `2 neg`) | Không dính người mặc áo vàng đứng cạnh và không dính hành khách ngồi trong xe; loại được túi nilon xanh treo hông | Diện tích mask `49437 px` (`6.3%` ảnh), `1 polygon`; biên bám tốt phần đầu, mui, bánh xe và bảng `TAXI` của chiếc tuk-tuk ở giữa hàng | [tuktuk_img023.jpg](masks/tuktuk_img023.jpg) |

- **So sánh định tính `sam2.1_b.pt` vs `mobile_sam.pt` trên cùng tọa độ click (`img_013.jpg`):** `sam2.1_b.pt` giữ biên khung kim loại và lốp bánh xe cao su liền mạch thành `1 polygon`, ít vỡ lỗ nhưng dễ giữ theo phần chân người lái và lem nhẹ qua ô hở khung; `mobile_sam.pt` nhẹ và tách khoảng hở khung xe sạch hơn, nhưng dễ rách mảng tối ở bánh xe/gầm xe (`2 polygon`) và lem ra người đứng sát phía sau. Vì vậy chọn **`sam2.1_b.pt`** làm bản chính cho Part 5.
- Đối chiếu guideline mục 4.2:
  - Quy tắc 4.2.1 (bám biên phần nhìn thấy) và 4.2.4 (hai vật chạm nhau không chồng lên nhau) giúp đặt đúng `neg` click lên người lái/người đứng cạnh để tách người khỏi xe. Quy tắc 4.2.2 (lỗ thủng `> 5%` diện tích phải bỏ khỏi polygon) và 4.2.3 (kính xe thuộc vật) còn thiếu hướng dẫn cụ thể cho **khung hở không có kính của tuk-tuk/xích lô**, nên đã được bổ sung ở `CAL-04` trong `guideline_v1.1_calibrated.md`.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa dùng trực tiếp Interactor SAM trên CVAT trong bài này mà chạy offline bằng `scripts/sam2_points.py` (`sam2.1_b.pt` và `mobile_sam.pt`). Khác biệt chính: offline cần nhập tọa độ pixel thủ công qua CLI nên khó tinh chỉnh lặp nhiều lần như click chuột trực tiếp trên UI CVAT, nhưng tái lập được chính xác bộ click.

## 6. Calibration (Part 6)

- Số quyết định: **6** (`CAL-01` đến `CAL-06` trong `submission/qc/calibration_decisions.csv`); quan trọng nhất: **`CAL-01`** (khắc phục lỗi YOLOE ngưỡng `0.15` sinh box trùng và box quang gánh/bụi cây) và **`CAL-05`** (chấm dứt thói quen duyệt nhanh `4 phút`, bắt buộc quét mù 10 giây đầu để không lọt `11/12` bẫy).
- Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md`:
  - Chốt đầy đủ 7 câu hỏi mở từ mục 6 của v1.0 (`bicycle` vs `motorbike` cho xe đạp điện ở `CAL-02`, tách cụm xe máy dày đặc vs `ignore_region` ở `CAL-06`, giữ trọn khung/bánh xe nhưng loại hàng rời nhô `> 20%` ở `CAL-03`, giữ ngưỡng che khuất `>= 20%` kèm phần đầu/thân ở `CAL-05`, kính chắn gió thuộc mask nhưng khoảng hở khung xe `> 5%` loại khỏi mask ở `CAL-04`, cấm giữ box trùng lặp cùng 1 vật ở `CAL-01`, không đặt ngưỡng `conf` miễn kiểm tra ở `CAL-05`).
  - Bổ sung 3 nhóm ca khó mới kèm ảnh ví dụ (`img_003.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`, `img_018.jpg`, `img_036.jpg`).
- Nguyên nhân gốc:
  - Kết hợp giữa **máy ngưỡng thấp** (`person=0.20`, `three_wheeler=0.15` từ `threshold_sweep` sinh quá nhiều box trùng và box giả) và **thiên kiến tự động hóa khi duyệt quá nhanh** (`4 phút` cho 24 ảnh khiến hầu như không xóa/sửa box máy và lọt `11/12` bẫy).

## 7. QC và quyết định (Part 6)

- Mẫu: **5/24** ảnh (`0` ảnh `can_review` vì chưa tick ca nào trong lượt duyệt + `5` ảnh ngẫu nhiên, `seed 42`, tổng `125` đối tượng trong mẫu); người kiểm: **Thạch Trung Đức** (**hạn chế**: tự kiểm trên lô do chính mình duyệt và cỡ mẫu chỉ 5 ảnh).
- Lỗi theo **đối tượng**: **19/125 = 15.2%** · theo **ảnh**: **5/5 = 100.0%** (ngưỡng Release ≤ 2%).
- Top lỗi (từ `submission/qc/qc_review.csv` và `submission/qc/qc_report_auto.md`):
  1. `thua`: **14/19** lỗi (ví dụ `img_003.jpg` box `#28`, `#32` trùng `three_wheeler` và `#25`, `#26`, `#27` nhận nhầm bụi cây/xe máy; `img_014.jpg` `#194`–`#196` nhận nhầm bàn gỗ và quang gánh; `img_031.jpg` `#552 truck` trên giá áo khoác và `#548 person` trên mũ cam; `img_035.jpg` `#640 person` trên tranh tường và `#639` trùng xích lô).
  2. `sai_vi_tri_kich_thuoc`: **4/19** lỗi (ví dụ `img_006.jpg` `#110 bicycle` cắt giỏ/yên sau; `img_014.jpg` `#177 person` cao `17.4 px < 20 px`; `img_031.jpg` `#549 bicycle` mất nửa bánh sau và `#550 person` mất chân).
  3. `bo_sot`: **1/19** lỗi (`img_003.jpg` sót xe đạp của người đội nón lá phía sau xe ba gác).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `submission/qc/qc_report_auto.md`): với `5/5` ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngược lại, ngay cả khi mẫu 5 ảnh không phát hiện lỗi nào, quy tắc 3 (`3/n = 3/5 = 60%`) cũng cho thấy mẫu 5 ảnh không thể chứng minh lô đạt ngưỡng `<= 2%`.
- Script kiểm được / không kiểm được: `validate_labels.py` và `qc_report.py` kiểm được tên lớp trong taxonomy, box nằm trong ảnh, kích thước `< 20 px` (cảnh báo `img_014`, `img_018`, `img_023`, `img_024`, `img_025`, `img_027`…), tỉ lệ lỗi trên phiếu QC; **không** tự kiểm được box nhận nhầm tranh vẽ/quang gánh/giá áo (`#640`, `#195`, `#552`), box trùng lặp cùng xe (`#28`/`#32`), hay box bị cắt nửa người/nửa xe (`#549`, `#550`).

**Quyết định: Rework**, ngày **2026-11-17**. Lý do (đi theo Hold → Rework → Release):
1. *Hold?* Không — `guideline_v1.1_calibrated.md` đã chốt đầy đủ các câu hỏi mở (`CAL-01`..`CAL-06`), dữ liệu ảnh gốc không hỏng.
2. *Rework?* **Có** — ảnh có lỗi `5/5 = 100.0% > 2%`, lỗi hệ thống `thua` chiếm `14/19` lỗi (`73.7%`), và tỉ lệ bắt bẫy đạt `1/12 = 8.3% < 80%`.
Nếu Rework / Hold, việc cần làm tiếp:
- Nâng ngưỡng lọc `three_wheeler` lên `>= 0.30` (hoặc gộp NMS cùng lớp) và **duyệt lại toàn bộ 24 ảnh batch trên CVAT** đúng theo quy trình 4 bước (quét mù 10 giây, tối thiểu 1.5 phút/ảnh, xóa box giả/trùng lặp, sửa box cắt cụt, bỏ box đơn lẻ `< 20 px`, tick `can_review` ca phân vân).
- Xuất lại các ảnh đã sửa vào `submission/labels/qc_fixed.json` (hoặc `batch_reviewed.json` vòng 2), chạy lại `build_release.py`, `qc_sample.py --n 5 --force` và `qc_report.py` để xác nhận trước khi Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: sử dụng trợ lý AI (Antigravity / Gemini) hỗ trợ chạy các script (`threshold_sweep.py`, `merge_prelabels.py`, `inject_traps.py`, `sam2_points.py`, `review_stats.py`, `build_release.py`, `qc_sample.py`, `qc_report.py`), mô tả nhận biết đối tượng tham khảo trên 4 ảnh gold trước khi gán `img_036.jpg`, rà soát 5 ảnh QC trực quan và tổng hợp báo cáo `REPORT.md`, `guideline_v1.1_calibrated.md`. Toàn bộ số liệu đều chạy trực tiếp từ script và ảnh thật trên máy.
- **Gold có bị nhiễm không**: có một phần ở cuối Part 1 — đã tự gán tay 3 ảnh đầu (`img_001.jpg`, `img_012.jpg`, `img_020.jpg` = 29 box) trên CVAT trước, sau đó có nhờ AI đọc gợi ý đối tượng khi hoàn thiện nốt `img_036.jpg` (`+20` box, tổng `49` đối tượng trên `4` ảnh) nên ghi nhận trung thực tình trạng tham khảo này.
- **Máy / GPU / CVAT dùng; vướng mắc cài đặt**: Windows 11 x64, CPU only (không GPU), Python 3.11.9, `torch 2.15.0+cpu`, CVAT Docker `v2.81.0`. Chạy trên CPU nên `sam2.1_b.pt` tốn ~5.6s/ảnh (`r04`), `mobile_sam.pt` nhanh hơn nhưng dễ vỡ mask vùng tối.
- **Dùng cache `data/cache/` thay vì tự chạy?** Không — toàn bộ `r01`–`r06` đều tự chạy trên máy local (khớp `work/runs.csv`).
- **Lỗi / bug script đã gặp (nếu có)**: khi chạy lệnh nhiều đối số có dấu phẩy hoặc dấu `=` trên Windows PowerShell cần bọc ngoặc kép (`--thr "person=0.2,three_wheeler=0.15"`, `--split "batch,gold"`, `--pos "400,480"`) để PowerShell không tách chuỗi sai.
- **Một điều bạn làm khác lần sau**: không bao giờ duyệt lướt `24` ảnh trong `4 phút` theo hộp sẵn có của máy; phải tắt pre-label 10 giây đầu ở mỗi ảnh và chạy `merge_prelabels` với ngưỡng `three_wheeler >= 0.30` thay vì giữ ngưỡng quá thấp (`0.15`) từ tập gold nhỏ.
