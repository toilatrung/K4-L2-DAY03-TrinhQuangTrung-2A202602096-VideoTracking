# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Trinh Quang Trung`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT local |
| Thời gian gán `clip_02` (warm-up) | chưa ghi |
| Thời gian gán `clip_01` | chưa ghi |
| Số track đã vẽ trong `clip_01` | 12 |
| Số keyframe trung bình mỗi track | chưa ghi |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe đi sát rìa ảnh: chỉ khoanh phần nhìn thấy, bbox chạm rìa ảnh, không đoán phần ngoài khung.
2. Xe xuất hiện rất ngắn ở frame 117-119 và 146-148: giữ thành track ngắn riêng để không gộp nhầm với xe khác.
3. Các xe bị che/cắt nhau quanh đoạn giữa clip: giữ ID nếu cùng xe còn liên tục hợp lý; nếu rời khung thì tạo track mới.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: phát hiện một số track ngắn cần xem lại, đặc biệt track 9, 10, 11, 12.
- Lượt 2: kiểm tra frame đầu/cuối để tránh bbox treo sau khi xe rời khung.
- Lượt 3: kiểm tra các frame giữa quanh đoạn che khuất để giảm ID switch.

Kiểm chéo với: chưa ghi. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: chưa ghi. Số lỗi bạn ấy tìm được trong bản của bạn: chưa ghi.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Cần bổ sung rõ luật cho xe chỉ xuất hiện 1-2 frame ở rìa ảnh và luật giữ ID khi xe bị che ngắn nhưng không liên tục trong export.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | chưa có manifest |
| Thời điểm khóa | chưa có manifest |
| Số row / frame / track trước khi mở reference | chưa có manifest |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có | chưa có |
| Sau rework | 0.6454 | 0.6108 | 0.6854 | 0.8030 | 0.8609 | 0.7504 | 0.7759 | 26 | 115 | 2 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| ID âm từ CVAT export | 117, 119 | 9 | đổi `track_id = -1` thành ID dương riêng |
| ID âm từ CVAT export | 146, 148 | 10, 11 | tách thành track dương để không trùng ID trong cùng frame |
| ID âm từ CVAT export | 177 | 12 | đổi thành track dương riêng |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / ultralytics 8.4.145 / torch 2.11.0+cpu / lap 0.5.13 |
| weights / hai tracker | `yolo26n.pt`; `bytetrack.yaml`; `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | conf 0.25 / IoU 0.7 / imgsz 960 / COCO classes 2,5,7 |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.6454 | 0.6108 | 0.6854 | 0.8030 | 0.8609 | 0.7504 | 0.7759 | 26 | 115 | 2 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.5910 | 0.5301 | 0.6625 | 0.8022 | 0.7861 | 0.5165 | 0.7750 | 194 | 40 | 0 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi thấp hơn IDF1: MOTA = 0.7504, IDF1 = 0.8609. Điều này nói rằng nhãn của tôi giữ danh tính tương đối ổn, nhưng còn thiếu nhiều bbox so với gold: FN = 115 trên 573 bbox gold. MOTA chủ yếu cộng phạt FP, FN và IDSW theo số lượng detection; lỗi đổi ID chỉ đi qua IDSW nên nếu số IDSW ít, ví dụ chỉ 2, MOTA không giảm mạnh như các metric association/identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack ở IDF1 và AssA: IDF1 tăng từ 0.8746 lên 0.9001, AssA tăng từ 0.7761 lên 0.8204. IDSW không đổi, cả hai đều là 2. Một chuỗi đáng xem là đoạn quanh frame 152-167 của `gt_track 8`: ByteTrack có nhiều bbox lỏng cho `pred_track 54` ở các frame 152, 161-167, còn ReID không còn bị liệt kê `gt_track 8` trong nhóm partially covered. Điều này gợi ý treatment giữ/khớp track tốt hơn ở đoạn đó. Tuy vậy không được kết luận chênh lệch chỉ do ReID, vì treatment dùng BoT-SORT + ReID còn control dùng ByteTrack, tức là hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Từ ByteTrack sang BoT-SORT + ReID, DetA tăng từ 0.6487 lên 0.7110. FP tăng nhẹ từ 88 lên 91, nhưng FN giảm mạnh từ 54 xuống 26. Vì IDSW giữ nguyên ở 2 và AssA tăng, phần cải thiện chính nằm ở việc model/treatment giữ được nhiều bbox khớp gold hơn. Lỗi còn lại có cả detector và association: detector vẫn tạo ghost tracks như ReID `pred_track 7` frame 16-116 và `pred_track 27` frame 106-121, nhưng association vẫn có fragment ở `gt_track 5`, `gt_track 6`, `gt_track 7`.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở frame 16-116, ReID tạo `pred_track 7` bị diagnostics đánh dấu là ghost, không khớp track tham chiếu nào. Đây là chỗ tôi không nên thêm bbox chỉ vì model có detection; nếu gold không có object hợp lệ thì ReID đang sai do FP/ghost track.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID làm tôi cần xem lại đoạn `gt_track 8`: nhãn của tôi chỉ phủ 9/33 frame theo `eval_vs_gold.json`, trong khi ReID vs gold không còn báo `gt_track 8` là partially covered. Cụ thể nên xem lại quanh frame 136-170, nơi `eval_reid_vs_me.json` cho thấy model track 34 xuất hiện trước/sau track tham chiếu của tôi. Đây có khả năng là xe tôi bắt đầu muộn hoặc kết thúc sớm.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ sửa `GUIDELINE_MINI.md` theo ba điểm:

- Thêm luật bắt đầu/kết thúc track cho xe rất nhỏ hoặc chỉ xuất hiện 1-2 frame ở rìa ảnh.
- Thêm rule kiểm bắt buộc với các track ngắn dưới 3 frame: xác nhận là vật thể thật hay xóa.
- Thêm bước QC sau export: chạy validator, xem danh sách partially covered/fragmented tracks, rồi quay lại CVAT kiểm frame cụ thể trước khi nộp.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
