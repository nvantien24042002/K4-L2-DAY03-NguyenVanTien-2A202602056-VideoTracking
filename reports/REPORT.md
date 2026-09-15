# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Văn Tiến — 2A202602056`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT `...` |
| Thời gian gán `clip_02` (warm-up) | `N/A` phút |
| Thời gian gán `clip_01` | `N/A` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `5` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Entry/exit của vehicle:** xác định thời điểm xe bắt đầu đủ rõ để track và thời điểm xe thực sự rời khỏi frame; áp dụng rule không giữ bbox khi xe đã rời khung.
2. **Occlusion/crossing:** duy trì cùng ID khi vẫn có bằng chứng liên tục về vị trí, hướng di chuyển và đặc điểm nhìn thấy của xe; không đổi ID chỉ vì xe bị che hoặc đi qua vùng overlap.
3. **Bbox ở vùng mép ảnh và interpolation:** chỉ annotate phần nhìn thấy trong ảnh, không đoán phần nằm ngoài khung; kiểm tra các keyframe ở entry/exit và các đoạn có nguy cơ interpolation drift.
`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `N/A bài làm cá nhân`
- Lượt 2: `N/A bài làm cá nhân`
- Lượt 3: `N/A bài làm cá nhân`

Kiểm chéo với: `N/A`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A`. Số lỗi bạn ấy tìm được trong bản của bạn: `N/A`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`N/A bài làm cá nhân`


## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `a64bffcaf734f806162420342228da3336f67e5dc2c107f096e618419e36971f` |
| Thời điểm khóa | `2026-09-15T05:25:35.820346+00:00` |
| Số row / frame / track trước khi mở reference | `574 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `N/A — không có file eval riêng cho snapshot pre-gold` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` |
| Sau rework | `N/A — chưa rework` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` | `N/A` |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost/entry cần kiểm tra | 97–100 | 6 | `Chưa rework; cần kiểm tra thời điểm xe đủ rõ để bắt đầu track.` |
| Ghost/entry cần kiểm tra | 103–105 | 7 | `Chưa rework; cần kiểm tra thời điểm xe đủ rõ để bắt đầu track.` |
| Ghost/entry cần kiểm tra | 133–135 | 8 | `Chưa rework; cần kiểm tra thời điểm xe đủ rõ để bắt đầu track.` |
| Ghost/exit cần kiểm tra | 149–151 | 4 | `Chưa rework; cần kiểm tra frame cuối cùng xe còn nhìn thấy.` |
| Loose bbox | 168 | 8 | `Chưa rework; cần kiểm tra và chỉnh bbox nếu thực sự lệch/lỏng.` |
| Ghost/exit cần kiểm tra | 169–171 | 8 | `Chưa rework; cần kiểm tra frame cuối cùng xe còn nhìn thấy.` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | `0.8183` | `0.8111` | `0.8261` | `0.8854` | `0.9643` | `0.9284` | `0.8750` | `21` | `20` | `0` |
| ByteTrack control vs gold | `0.7085` | `0.6487` | `0.7761` | `0.8463` | `0.8746` | `0.7487` | `0.8226` | `88` | `54` | `2` |
| BoT-SORT + ReID vs gold | `0.7635` | `0.7110` | `0.8204` | `0.8721` | `0.9001` | `0.7923` | `0.8595` | `91` | `26` | `2` |
| ReID vs bạn | `0.8004` | `0.7464` | `0.8589` | `0.9191` | `0.8960` | `0.7840` | `0.9121` | `93` | `29` | `2` |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Trong kết quả của tôi, MOTA = 0.9284 thấp hơn IDF1 = 0.9643. IDF1 tập trung vào chất lượng duy trì identity qua các detection đúng; MOTA tổng hợp nhiều lỗi gồm FP, FN và ID switch. Vì vậy hai metric có thể khác nhau. Trường hợp MOTA cao nhưng IDF1 thấp cho thấy detection có thể khá tốt nhưng association/identity chưa ổn. MOTA không phản ánh lỗi ID trực tiếp mạnh bằng IDF1 vì ID switch chỉ là một thành phần trong công thức MOTA, trong khi IDF1 đánh giá trực tiếp khả năng duy trì identity.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`So với gold, BoT-SORT + ReID có IDF1 = 0.9001 và AssA = 0.8204, cao hơn ByteTrack với IDF1 = 0.8746 và AssA = 0.7761. Tuy nhiên cả hai đều có 2 IDSW. Một sequence đáng chú ý là quanh ID 6: ReID bị fragment tại frame 107 và 110 (24 → 28 → 31), còn ByteTrack có các switch khác ở frame 59 của ID 4 và frame 94 của ID 5. Như vậy ReID cải thiện identity association tổng thể nhưng không loại bỏ hoàn toàn ID switch. Không thể kết luận đây là causal effect riêng của ReID vì treatment đồng thời thay đổi tracker implementation từ ByteTrack sang BoT-SORT + ReID.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ByteTrack có DetA = 0.6487, FP = 88, FN = 54; ReID có DetA = 0.7110, FP = 91, FN = 26. ReID giảm FN mạnh nhưng FP tăng nhẹ, nên khả năng bao phủ detection tốt hơn nhưng vẫn có false positives. AssA của ReID = 0.8204 cao hơn ByteTrack = 0.7761, cho thấy association cũng tốt hơn. Vì vậy lỗi còn lại không chỉ thuộc detector; có cả detection/localization và association. Các ID switch/fragmentation trong diagnostics cho thấy association vẫn là một nguồn lỗi rõ ràng.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Một ví dụ là ID 6 quanh frame 107–110: ReID bị fragment thành track 24 → 28 → 31, với ID switch tại frame 107 và 110. Trong khi annotation của tôi không có IDSW trong eval_vs_gold.json. Đây là evidence rằng tại sequence này model ReID không duy trì identity đúng như annotation của tôi/gold.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 168, ID 8 là điểm cần xem lại annotation vì eval_vs_gold báo loose bbox với IoU = 0.598. Tuy nhiên model không phải ground truth: diagnostics của ReID cũng có loose boxes và fragmentation. Do đó tôi chưa sửa annotation chỉ dựa trên model; tôi sẽ kiểm tra trực tiếp frame 168 trong CVAT rồi mới quyết định rework.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ làm rõ hơn ba điểm: (1) định nghĩa cụ thể “confidently identifiable” để thống nhất frame bắt đầu track; (2) quy tắc kết thúc track khi xe rời khung, đặc biệt ở vùng mép ảnh; (3) quy tắc đặt keyframe ở các đoạn occlusion/crossing và nơi interpolation dễ drift. Về quy trình, tôi sẽ ghi thời gian annotate, số keyframe và nhật ký self-QC ngay khi làm; sau pre-gold sẽ kiểm tra từng finding trong CVAT, chỉ sửa finding được xác nhận là lỗi, export lại MOT và chạy evaluation lần hai để có before/after rõ ràng.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
