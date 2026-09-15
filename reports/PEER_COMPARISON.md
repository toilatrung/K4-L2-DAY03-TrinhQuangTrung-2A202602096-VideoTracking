# So sánh với repo Nguyễn Xuân Việt Anh

Repo đối chiếu: `https://github.com/Vietanhhhhhh2003/K4-L2-DAY03-NguyenXuanVietAnh-2A202602102-VideoTracking`

Ngày đọc: `2026-09-15`

## Annotation vs gold

| Người | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW | PRED boxes | Tracks |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Trinh Quang Trung | 0.6953 | 0.6652 | 0.7322 | 0.8230 | 0.9091 | 0.8360 | 0.7967 | 7 | 86 | 1 | 494 | 10 |
| Nguyễn Xuân Việt Anh | 0.7833 | 0.7746 | 0.7931 | 0.8501 | 0.9677 | 0.9337 | 0.8328 | 35 | 3 | 0 | 605 | 8 |

## Nhận xét nhanh

- Cả hai đều qua cổng annotation. Bài của Trung đạt IDF1 0.9091, MOTA 0.8360, MOTP 0.7967; bài của Việt Anh cao hơn ở cả ba metric chính.
- Khác biệt lớn nhất nằm ở coverage: Trung còn FN = 86, trong khi Việt Anh chỉ FN = 3. Điều này kéo DetA và MOTA của Trung xuống dù FP thấp hơn.
- Về identity, Trung có 1 IDSW và `gt_track 7` vẫn bị tách thành các ID 7, 9, 10. Việt Anh có IDSW = 0 và không có fragmented track trong diagnostics.
- Trung ít bbox thừa hơn: FP = 7 so với FP = 35. Nhưng việc bỏ sót đoạn xe trong gold làm tổng điểm thấp hơn rõ rệt.
- Nếu sửa tiếp, ưu tiên của Trung nên là mở lại các đoạn thiếu coverage: `gt_track 5` chỉ phủ 31/60 frame, `gt_track 8` phủ 21/33 frame, `gt_track 4` phủ 72/95 frame. Sau đó kiểm lại ID của `gt_track 7` quanh frame 117-123.

## Kết luận

Bài của Trung đã cải thiện mạnh so với bản trước và qua gate chắc hơn, nhưng bài của Việt Anh hiện sát gold hơn nhiều về coverage và association. Trung nên học cách bạn ấy kéo dài track đầy đủ hơn ở các đoạn vào/ra khung và che khuất, đồng thời vẫn giữ lợi thế FP thấp.
