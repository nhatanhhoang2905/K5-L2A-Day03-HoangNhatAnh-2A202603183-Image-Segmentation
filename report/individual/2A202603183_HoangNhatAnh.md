# Báo cáo thực hành Day 3 — Image Segmentation

- Họ và tên: Hoàng Nhật Anh.
- Mã học viên: `2A202603183`.
- Ngày lập báo cáo: 12/11/2026 (UTC+7).
- Công cụ: CVAT để gán nhãn và xuất dữ liệu; SAM ViT-B để hỗ trợ tạo mask tự động sau khi tự vẽ object Medium đầu tiên; Polygon để vẽ lại hai instance xe bị gộp; Python và script `scripts/inspect_submissions.py` để kiểm tra cấu trúc ZIP khi chuẩn bị báo cáo.

## 1. Bài đã nộp

Đã chuẩn bị đủ 9 file ZIP trong thư mục `submissions/`, bao phủ đủ 14 ảnh đầu vào của bài. Tình trạng dưới đây được đối chiếu từ các file export trên máy; chưa xác nhận việc push lên GitHub hoặc nộp link trên LMS.

| Task | File ZIP đúng tên | Số ảnh có trong export | Điểm tối đa (coach chấm sau) |
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
| **Tổng tối đa** | **9 ZIP** | **14 / 14** | **100** |

Kết quả kiểm tra cấu trúc ngày 12/11/2026: **9/9 ZIP đạt `OK`**. Các file đáp ứng kiểm tra tên ảnh, tên lớp và biểu diễn mask của script. Easy và các checkpoint semantic có PNG trong `SegmentationClass/`; các task instance và Hard có COCO JSON với mask dạng RLE.

Số mask ghi nhận trong COCO export:

| Task | Số mask |
| --- | ---: |
| medium_instance | 62 |
| hard_panoptic | 69 |
| cp1_holes | 5 |
| cp2_slice | 12 |
| cp5_occlusion | 36 |

Số mask là số annotation đã xuất, chưa khẳng định số vật đúng trong ảnh. Kiểm tra cấu trúc chưa đánh giá độ chính xác của biên, thiếu/thừa vật hoặc mức độ phủ vùng. Riêng Hard cần kiểm trực quan chồng lấn giữa stuff và thing, cũng như các vùng bị bỏ sót.

## 2. Một quyết định trước khi dùng gợi ý

Tôi tự vẽ object `person 12` ở vị trí giữa ảnh `000000181542.jpg` của task `medium_instance`. Sau khi hoàn thành object này, tôi sử dụng SAM ViT-B để hỗ trợ vẽ tiếp các object khác.

- Ảnh và vị trí: `000000181542.jpg`, người ở giữa màn hình.
- Object và class: `person 12` trong CVAT, class `person`. Số `12` là số object do tôi cung cấp theo giao diện CVAT; chưa đối chiếu với annotation ID trong COCO export.
- Quy tắc đối chiếu biên: mask theo phần người nhìn thấy, tách người khỏi xe máy và nền xung quanh; không suy đoán phần cơ thể bị che. Quy tắc này được dùng để giải thích và kiểm lại biên của object khi chuẩn bị báo cáo.
- Thứ tự thao tác: tự vẽ object đầu tiên, sau đó sử dụng công cụ tự động SAM ViT-B để vẽ tiếp.
- Với các đề xuất tự động sau đó: tôi ghi nhận mask thiếu phần người, thiếu phần xe và trường hợp công cụ gộp hai vật vào một mask. Các lỗi này cho thấy cần kiểm lại độ đầy đủ của vùng và số instance sau khi dùng SAM ViT-B; chi tiết ở mục 3.

## 3. Một lỗi tôi tìm thấy và sửa

Sau khi sử dụng SAM ViT-B, tôi ghi nhận mask thiếu phần người, thiếu phần xe và trường hợp công cụ tự gộp hai vật. Trường hợp cụ thể tôi đã sửa là lỗi gộp hai xe trong task `cp2_slice`.

- **Task/ảnh/vùng:** Task `cp2_slice`, ảnh `000000017627.jpg`, khu vực hai xe ô tô đỗ sát song song nhau ở giữa khung hình.
- **Loại lỗi:** Gộp–tách đối tượng (Instance Merging Error).
- **Bằng chứng quan sát:** Hai xe sát nhau có độ tương phản màu sơn thấp, ban đầu bị gộp vào một mask duy nhất. Với cặp xe này, danh sách Objects chỉ thể hiện một đối tượng xe.
- **Quy tắc áp dụng:** Hai xe cùng class `car` đứng cạnh nhau vẫn là hai instance riêng biệt; không gộp chúng chỉ vì màu gần giống nhau hoặc không có khoảng cách rõ.
- **Hành động sửa:** Tôi xóa mask gộp, phóng to ảnh ở mức 300% để tìm khe ranh giới giữa hai xe, dùng Polygon vẽ lần lượt Car #1 và tạo đối tượng mới cho Car #2. Cả hai đối tượng đều giữ class `car`.
- **Kiểm tra sau sửa:** Tôi đã Save bằng `Ctrl + S`, kiểm tra cặp xe có hai ID riêng biệt trong danh sách Objects và export lại định dạng COCO 1.0 vào `submissions/cp2_slice.zip`.

Kết quả kiểm tra cấu trúc bằng lệnh:

```powershell
py -3 scripts/inspect_submissions.py --dir submissions
```

Kết quả: **9/9 task đều `OK`**, gồm `cp2_slice`; cấu trúc COCO và Segmentation mask hợp lệ theo phép kiểm của script. Kết quả này xác nhận cấu trúc export, chưa tự chứng minh chất lượng biên hay việc tách instance chính xác. Riêng Hard vẫn cần kiểm trực quan chồng lấn và phủ vùng panoptic trong CVAT.

Chưa có kết quả IoU/PQ hoặc điểm tự đánh giá bằng reference trong lần kiểm này. Không tự ghi điểm, PASS, top 3 hoặc bonus. Điểm ba tier nếu được tự đánh giá có mức tối đa 82; điểm toàn bài tối đa 100 theo rubric của lớp.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Ba ca dưới đây được đối chiếu từ ảnh đầu vào và quy tắc của bài khi chuẩn bị báo cáo. Chúng là các vị trí cụ thể cần kiểm lại trên mask; không khẳng định người thực hiện đã xử lý hoặc sửa chúng trước đó.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` — `000000144300.jpg`, cửa kính trên xe ở nền bên trái, phía sau xe máy đỏ | Giữ kính trong mask của xe; hoặc khoét kính như một lỗ nền. | Cửa kính là chi tiết bên trong đường bao xe. Quy tắc checkpoint yêu cầu giữ kính/chi tiết trong mask theo quy định task. | Khi kiểm mask, giữ cửa kính thuộc xe; không khoét chỉ vì vùng kính có màu khác thân xe. |
| `cp4_curb` — `7d83710e-4697c3b2.jpg`, dọc bó vỉa phía phải và dải đất quanh gốc cây | Phân lớp theo màu bề mặt; hoặc theo chức năng và ranh bó vỉa. Riêng dải đất có thể bị gán ép vào sidewalk. | Thấy bó vỉa ngăn mặt đường với khu vực bên phải; sidewalk bê tông có lối đi liên tục. Task chỉ có `road` và `sidewalk`, cần phân biệt vùng thuộc schema với dải đất trồng cây. | Bám ranh chức năng để phân road–sidewalk. Cần coach xác nhận cách xử lý dải đất quanh cây trong task hai lớp, tránh gán ép để đủ coverage. |
| `cp5_occlusion` — `000000336232.jpg`, cụm ô tô phía phải ảnh, phía sau xe vàng ở tiền cảnh | Vẽ toàn bộ hình dáng xe suy đoán sau vật che; hoặc chỉ vẽ phần nhìn thấy và giữ định danh của từng xe. | Xe vàng ở tiền cảnh che một phần khu vực phía sau. Quy tắc yêu cầu không vẽ phần bị che; các mảng nhìn thấy của cùng một vật vẫn thuộc một instance khi đủ bằng chứng. | Chỉ kiểm và gán nhãn phần nhìn thấy; không gộp các xe sát nhau, không tự tách một xe chỉ vì bị che. Nếu không đủ bằng chứng liên kết các mảng, ghi vị trí để hỏi coach. |
