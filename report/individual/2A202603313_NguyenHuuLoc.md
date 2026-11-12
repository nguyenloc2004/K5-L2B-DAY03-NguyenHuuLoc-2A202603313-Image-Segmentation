# Báo cáo thực hành Day 3 — Segmentation

- Mã học viên theo lớp: 2A202603313
- Ngày / CVAT local: 2026-11-11 / CVAT local
- Công cụ đã dùng: CVAT Brush, Polygon, SAM3

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `submissions/cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `submissions/cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `submissions/cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `submissions/cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `submissions/cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `submissions/cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Không có export lỗi. `scripts/inspect_submissions.py` báo cả 9 ZIP `OK`

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, người phụ nữ mặc áo dài trắng đi bộ ở giữa ảnh, trên vạch qua đường.
- Class và quy tắc tôi dùng để chọn biên: `person`. Mask phủ toàn bộ phần cơ thể nhìn thấy từ tóc đến chân, gồm cả tà áo dài và quần. Biên theo mép vải/da, không lấy bóng đổ trên mặt đường. Phần xe máy phía sau nằm ngoài mask vì là object `motorcycle` riêng.
- Nếu dùng gợi ý sau đó: không dùng.
- Quyết định gán nhãn: người ngồi trên xe máy và chiếc xe là hai object riêng (`person` + `motorcycle`), không gộp thành một mask.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `easy_semantic`, ranh road–sidewalk ở `817bca71-00000000` (vỉa hè bên trái và bên phải đường) và `81ae7cbb-6bc63a4a`; cùng các mảng cây/bụi ven đường.
- Lỗi thuộc loại: biên + phủ vùng.
- Bằng chứng tôi nhìn thấy: khi so overlay với reference, dải sidewalk tôi tô quá hẹp/lệch khỏi bó vỉa, một phần vỉa hè bị gán `road` hoặc để trống; một số bụi cây ven đường chưa được tô `vegetation`. IoU `sidewalk` chỉ 0.485, `vegetation` 0.604.
- Quy tắc và hành động sửa: ranh road–sidewalk theo bó vỉa/chức năng (phần người đi bộ là sidewalk), không theo màu mặt đường. Phóng to từng ảnh, kéo biên sidewalk ra đúng mép bó vỉa, tô bổ sung các mảng cây còn trống.
- Sau sửa đã Save và export lại chưa? Đã Save trong CVAT và export lại `easy_semantic.zip` (cùng `medium_instance.zip`, `hard_panoptic.zip`).



## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. Medium `000000181542.jpg`, người mẫu trên poster quảng cáo ở mặt tiền cửa hàng (phía trên, giữa ảnh) | (a) Có hình người → gán `person`; (b) chỉ là hình in trên biển quảng cáo, không phải người thật trong cảnh → không gán | Instance chỉ đếm vật thật nhìn thấy trong cảnh; reference không có object ở vị trí này | Lúc làm tôi đã gán `person`; sau khi đối chiếu tôi nghiêng về (b). Câu hỏi cho coach: hình người trên poster/biển quảng cáo có luôn bị loại không? |
| 2. Hard `000000460147.jpg`, xe chở ô tô ở giữa đường | (a) Cả xe chở + các ô tô trên thùng là một `truck`; (b) đầu kéo là `truck`, mỗi ô tô trên thùng là một `car` riêng | Các ô tô trên thùng nhìn thấy rõ và đếm được; nhưng chúng là hàng hóa, không phải phương tiện đang lưu thông. Reference coi cả khối là một `truck` | Tôi đã chọn (b) nên bị tính nhiều `car` thừa. Câu hỏi cho coach: hàng hóa là xe chở trên xe tải có nên tách instance không? |
| 3. Hard `000000350023.jpg`, vỉa hè và dải cây hai bên đường | (a) Sidewalk chỉ là dải lát sát mép đường, cây là phần riêng; (b) sidewalk gồm cả khu lát rộng phía ngoài, cây lớn hai bên tô `vegetation` toàn bộ tán | Ranh sidewalk theo bó vỉa/chức năng; stuff phải phủ hết vùng nhìn thấy. Reference tô sidewalk rộng và cây chiếm khoảng 18% ảnh | Tôi chỉ tô dải sidewalk hẹp nên `sidewalk` PQ 0 và thiếu `vegetation`. Cần mở rộng sidewalk theo chức năng và tô tán cây; ghi nhận đây là phần chưa sửa xong. |
