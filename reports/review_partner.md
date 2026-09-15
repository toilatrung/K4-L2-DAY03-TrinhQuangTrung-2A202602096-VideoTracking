# Peer review — Day 3

| Trường | Giá trị |
| --- | --- |
| Author | `Trinh Quang Trung` |
| Reviewer | `Nguyen Xuan Viet Anh / peer repo comparison` |
| Pair ID | `K4-L2-DAY03 peer comparison` |
| CVAT version | `local CVAT, export MOT 1.1` |
| Thời điểm review | `2026-09-15` |

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 122 | 123 | 7 / 9 | ID switch / tách track | `eval_vs_gold.json` báo `gt_track 7` đổi từ ID 9 sang ID 7 ở frame 123; cùng xe phải giữ một ID qua occlusion ngắn. | Merge đúng track 9 vào track 7 trong CVAT, giữ continuity quanh frame 117-123. | needs-review |
| 2 | 112-126 | 113-127 | 5 | Bbox trôi | Nhiều bbox ID 5 chỉ IoU 0.50-0.59; rule bbox phải ôm sát phần nhìn thấy. | Thêm keyframe quanh frame 113-127, chỉnh sát thân xe ở từng đoạn thay đổi kích thước. | needs-review |
| 3 | 135-169 | 136-170 | 8 | Thiếu đoạn track | `gt_track 8` chỉ phủ 21/33 frame; rule entry/exit phải bắt đầu từ frame đầu tiên xác định được xe. | Tua chậm đoạn 136-170, kéo dài track đến đúng frame xe xuất hiện/rời khung. | needs-review |
| 4 | 79-138 | 80-139 | 5 | Thiếu coverage | `gt_track 5` chỉ phủ 31/60 frame; FN cao là lỗi chính còn lại. | Kiểm frame đầu/cuối của track 5, thêm outside đúng frame và bổ sung bbox còn thiếu. | needs-review |
| 5 | 50-148 | 51-149 | 4 | Thiếu endpoint | `gt_track 4` phủ 72/95 frame; endpoint chưa đầy đủ. | Soi đoạn entry/exit của track 4, thêm keyframe nếu xe còn nhìn thấy. | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | `clip_01`: 10 track, 494 bbox |
| Một xe giữ một ID; không reuse ID cho xe khác | FINDING | frame 123, `gt_track 7`, ID 9 -> 7 |
| Occlusion ngắn giữ ID; crossing không đổi ID | FINDING | frame 117-123 cần review continuity của `gt_track 7` |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | `gt_track 5`, `gt_track 8`, `gt_track 4` thiếu coverage |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | ID 5 quanh frame 113-127 có IoU thấp |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | frame 103-127, IDs 5/6 có nhiều loose boxes |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | validator: 0 lỗi, 4 cảnh báo |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | closure hiện là `needs-review` cho các lỗi còn tồn tại |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | NEEDS-REVIEW | frame 123, `gt_track 7`, ID 9 -> 7 |
| 2 — endpoint/scope | NEEDS-REVIEW | `gt_track 5` 31/60, `gt_track 8` 21/33, `gt_track 4` 72/95 |
| 3 — geometry/interpolation | NEEDS-REVIEW | frame 113-127 ID 5, frame 103-106 ID 6 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: thiếu coverage ở `gt_track 5`, `gt_track 8`, `gt_track 4`; rule là mọi xe bốn bánh nhìn thấy được đều phải có bbox từ entry đến exit.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: chưa có; các finding chính đều là `needs-review` nếu còn thời gian rework.
3. Một rule cần Lab Coach làm rõ: với xe chỉ xuất hiện 1-2 frame ở rìa ảnh, nên bắt buộc gán nếu xác định được là xe bốn bánh hay có ngưỡng kích thước tối thiểu?
