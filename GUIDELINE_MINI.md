# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Trinh Quang Trung`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, SUV, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: nếu vật thể quá nhỏ nhưng vẫn xác định được là xe bốn bánh và xuất hiện liên tục ít nhất 2 frame thì vẫn tạo track. Nếu chỉ là đốm mờ 1 frame và không xác định được là xe bốn bánh thì bỏ qua.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới 25 frame | 25 frame xấp xỉ 2 giây ở 12.5 fps; cùng hướng chuyển động và vị trí dự đoán vẫn đủ tin cậy |
| Xe bị che lâu hơn ngưỡng trên | xem lại bằng frame trước/sau; chỉ giữ ID nếu vị trí, hướng đi và kích thước rất khớp | che lâu dễ nhầm xe khác, đặc biệt khi có nhiều xe cùng màu/hướng |
| Xe rời khung hình rồi quay lại | mặc định tạo track mới | khi đã ra khỏi frame thì không còn evidence liên tục để chứng minh cùng xe |
| Hai xe cắt nhau / chồng lên nhau | tua chậm từng frame, giữ ID theo quỹ đạo trước/sau crossing; không đổi ID chỉ vì bbox gần nhau | lỗi ID switch ảnh hưởng trực tiếp IDF1/AssA; đoạn crossing là nơi dễ tráo ID nhất |
| Xe chỉ xuất hiện 1-2 frame ở rìa ảnh | tạo track ngắn riêng nếu nhìn rõ là xe bốn bánh; không gộp vào track gần đó nếu không liên tục | tránh vừa bỏ sót xe thật, vừa tránh reuse ID sai |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần nhìn thấy được, không khoanh phần bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; nếu không chắc, tua thêm 2-3 frame rồi quay lại |
| Xe đang đỗ, không di chuyển | vẫn giữ track nếu xe còn nhìn thấy; nếu bbox gần như đứng im nhiều frame thì kiểm lại xem có phải xe thật hay object nền |
| Keyframe đặt dày ở đâu | đặt dày quanh entry/exit, crossing, occlusion, và các đoạn xe đổi tốc độ/kích thước nhanh; không chỉ đặt đầu-cuối rồi tin interpolation |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01 / frame 117-123 / ID 9 -> 7`
- Tình huống: xe của `gt_track 7` bị tách thành ID 9 rồi quay lại ID 7; evaluator báo ID switch ở frame 123.
- Quyết định: ghi nhận là lỗi cần merge đúng cặp trong CVAT nếu còn rework.
- Lý do: cùng track gold, cùng quỹ đạo; tách ID làm giảm AssA/IDF1 dù detection vẫn có.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 113-127 / ID 5`
- Tình huống: nhiều bbox của ID 5 chỉ đạt IoU khoảng 0.50-0.59 với gold.
- Quyết định: thêm keyframe quanh đoạn này, khoanh sát phần nhìn thấy hơn.
- Lý do: lỗi không phải ID mà là hình học/interpolation drift; ảnh hưởng LocA/MOTP và có thể làm mất match nếu IoU tụt dưới 0.5.

### Ca 3
- Clip / frame / ID: `clip_01 / gt_track 8 / frame khoảng 136-170`
- Tình huống: bản của tôi mới phủ 21/33 frame, trong khi gold cho thấy xe tồn tại dài hơn.
- Quyết định: cần mở lại đoạn entry/exit và kéo dài track nếu xe còn nhìn thấy.
- Lý do: thiếu đoạn làm FN tăng; MOTA bị kéo xuống dù FP thấp.

### Ca 4
- Clip / frame / ID: `clip_01 / gt_track 4 / coverage 72/95 frame`
- Tình huống: track có đủ ID chính nhưng bắt đầu/kết thúc chưa phủ hết vòng đời xe.
- Quyết định: khi QC phải kiểm frame đầu/cuối của từng track, không chỉ nhìn đoạn giữa.
- Lý do: thiếu endpoint gây FN nhưng khó thấy nếu tua nhanh.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Track ngắn ở rìa ảnh phải có luật riêng: nếu là xe bốn bánh thật thì tạo ID riêng, nhưng không reuse ID của xe gần đó khi không có continuity.
- Sau export phải chạy evaluator và xem ngay danh sách `partially_covered_gt_tracks`; lỗi lớn nhất của bản này là thiếu coverage ở `gt_track 5`, `gt_track 8`, `gt_track 4`.
- Bắt buộc QC frame đầu/cuối của từng track, nhất là entry/exit và occlusion. Đây là cách giảm FN hiệu quả hơn so với chỉ sửa bbox lẻ.
- Với đoạn crossing/occlusion, ghi lại frame nghi ngờ vào guideline ngay lúc annotate để lần review sau kiểm đúng nơi có rủi ro ID switch.
