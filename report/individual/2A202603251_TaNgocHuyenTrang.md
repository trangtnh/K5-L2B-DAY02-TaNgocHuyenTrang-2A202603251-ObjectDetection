# Báo cáo cá nhân — Ngày 2 Object Detection

Báo cáo kết quả annotation và pipeline YOLO trên 50 ảnh train, 10 ảnh eval. Quy cách nộp theo [SUBMISSION.md](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Tạ Ngọc Huyền Trang
- MSSV: 2A202603251
- Email: tnhtrang.works@gmail.com
- Lớp: L2B
- Link repo bài làm: [GitHub](https://github.com/trangtnh/K5-L2B-Day02-TaNgocHuyenTrang-2A202603251-ObjectDetection)
- Link Colab đã chạy: [Notebook Colab](https://colab.research.google.com/drive/1JO_str_u40fJEq4msosJb7vXa8eGPa_T?usp=sharing)

## 2. Schema và mini guideline

Chi tiết tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema cố định gồm `car=0`, `truck=1`, `bus=2`, `van=3`

Tôi áp dụng nguyên tắc mỗi xe có một bounding box riêng, box ôm sát phần xe nhìn thấy, không tự ước lượng phần bị che khuất và vẫn gán nhãn cho xe đang đỗ nếu thuộc 4 lớp quy định

Ba trường hợp mơ hồ được ghi nhận trong guideline mini:
| Ảnh | Tình huống | Cách xử lý ban đầu |
|---|---|---|
| `drive_031.jpg` | Nhiều xe ở xa, một số bị che khuất | Gán riêng từng xe còn nhận diện được, không gộp nhiều xe vào một box |
| `drive_014.jpg` | Xe cứu hộ chuyên dụng khó xác định lớp | Ghi nhận là trường hợp chưa rõ, không tự tạo thêm class |
| `drive_045.jpg` | Nhiều xe đỗ sát nhau, một số bị che một phần | Gán riêng từng xe còn nhận diện được, box chỉ bao phần nhìn thấy |

Điểm cần làm rõ: tiêu chí phân biệt rõ hơn giữa lớp như `car`, `van`, `bus` và các xe chuyên dụng để hạn chế nhầm lẫn khi annotation

## 3. Annotation và self-QC

- **Số file nhãn train:** 50 file YOLO TXT tương ứng với 50 ảnh train, tổng cộng 875 bounding box
- **Phân bố lớp:** 731 `car`, 26 `truck`, 87 `bus` và 31 `van`. Trong đó, `car` chiếm khoảng 83,54%, cho thấy dữ liệu chưa cân bằng giữa các lớp
- **Kết quả checker:** Chạy `python3 tools/check_yolo_labels.py`, kết quả 50/50 file hợp lệ, không phát hiện lỗi cấu trúc nhãn.
- **Danh sách 10 ảnh đã rà bằng mắt:** `drive_001`, `drive_005`, `drive_006`, `drive_008`, `drive_011`, `drive_016`, `drive_019`, `drive_031`, `drive_035`, `drive_044`.
- **Lỗi phát hiện và cách sửa:** Sau khi nhập nhãn vào CVAT, tôi kiểm tra lại các ảnh và sửa hai trường hợp phân loại chưa phù hợp:
| Ảnh | Box | Nhãn ban đầu | Nhãn sau sửa |
|---|---:|---|---|
| `drive_019` | 11 | `car` | `van` |
| `drive_044` | 15 | `truck` | `bus` |
- **Commit khóa nhãn ban đầu:** [1b43d68 – CP4: lock initial annotations after self-QC](https://github.com/trangtnh/K5-L2B-Day02-TaNgocHuyenTrang-2A202603251-ObjectDetection/commit/1b43d68).

## 4. Gold QC và rework

Sau khi hoàn thành annotation, sử dụng script `evaluate_annotations.py` để so sánh bộ nhãn gồm 875 bounding box với Gold Set gồm 653 box

| Chỉ số | Kết quả đánh giá | Sau rework | Ngưỡng yêu cầu | Đánh giá |
|---|---:|---:|---:|---|
| TP | 448 | — | — | — |
| FP | 427 | — | — | — |
| FN | 205 | — | — | — |
| Precision | 0.5120 (51.20%) | — | — | — |
| Recall | 0.6861 (68.61%) | — | — | — |
| F1 | **0.5864** | — | ≥ 0.50 | **ĐẠT** |
| mean_iou_of_matches | **0.8515** | — | ≥ 0.70 | **ĐẠT** |
| passed | **true** | — | — | **ĐẠT** |
- **File kết quả:** [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json)
- **Tình trạng rework:**
   - Lần chạy đầu sử dụng ngưỡng mặc định F1 ≥ 0.80 nên chưa đạt yêu cầu. Sau khi Lab Coach điều chỉnh ngưỡng F1 xuống 0.50, tôi chạy lại evaluator với tham số --min-f1 0.50 và nhận được kết quả passed: true
   - Không thay đổi bộ nhãn giữa hai lần chạy, vì vậy các chỉ số TP, FP, FN, Precision, Recall, F1 và Mean IoU vẫn giữ nguyên. Bộ nhãn đã đáp ứng ngưỡng đánh giá mới nên không thực hiện rework sau Gold QC
- **Phân tích so sánh giữa bộ nhãn học viên (875 box) và Gold set (653 box):**
   - **Độ khớp bounding box (Mean IoU = 0.8515):** Trong 875 box đã gán nhãn, có 448 box được ghép đúng với Gold Set theo điều kiện cùng class và IoU ≥ 0.50. Mean IoU đạt 0.8515 cho thấy các box được ghép có độ chồng lấp khá tốt với nhãn chuẩn
   - **False Positive (427 box):** Có 427 box không ghép được với Gold Set. Những ảnh có nhiều FP gồm `drive_003` (28 FP), `drive_002` (24 FP) và `drive_012` (21 FP). Các trường hợp này có thể do box thừa, sai class hoặc vị trí box chưa đủ khớp
   - **False Negative (205 box):** Có 205 đối tượng trong Gold Set chưa được ghép đúng với nhãn của tôi. Một số ảnh có FN cao là `drive_041` (19 FN), `drive_027` (15 FN) và `drive_009` (13 FN). Nguyên nhân có thể là bỏ sót phương tiện, phân loại chưa đúng hoặc box chưa khớp với nhãn chuẩn
   - **Tính nhất quán của class:** Bộ nhãn sử dụng đúng schema `car=0`, `truck=1`, `bus=2`, `van=3` và đã vượt qua checker với 50/50 file hợp lệ. Tuy nhiên, checker chỉ kiểm tra cấu trúc nhãn, không đánh giá trực tiếp việc phân loại có chính xác hay không
   - **Precision = 0.5120**, **Recall = 0.6861**. Danh sách chi tiết để rà bổ sung: [50 ảnh ưu tiên rà](assets/cp5_review_queue.csv).
- **Kết luận:** Bộ nhãn **875 box** đạt điều kiện Gold QC theo ngưỡng F1 ≥ 0,50 và mean IoU ≥ 0,70 được xác nhận cho bài này; có thể chuyển sang CP6 (huấn luyện mô hình YOLO). Đây là kết quả đánh giá annotation, không phải mAP của mô hình.

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:** Google Colab, GPU NVIDIA Tesla T4, Ultralytics 8.4.208, PyTorch 2.14.1+cu130.
- **Các tham số thay đổi so với notebook:** Tôi giữ nguyên cấu hình chính theo notebook: `model=yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`. Không chủ động thay đổi các tham số để tối ưu kết quả.
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4105 (41.05%) |
| Recall | 0.2303 (23.03%) |
| mAP50 | 0.2723 (27.23%) |
| mAP50–95 | 0.2017 (20.17%) |

Nguồn số liệu: [model_eval_metrics.json](assets/model_eval_metrics.json)

Đã lưu 10 ảnh prediction tại [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) và các kết quả huấn luyện, đánh giá tại [assets/yolo26_runs/](assets/yolo26_runs/)

Ví dụ, khi đối chiếu ảnh gốc drive_051 với ảnh prediction drive_051, tôi nhận thấy một xe màu đỏ ở tiền cảnh được mô hình dự đoán thành van với confidence 0.99. Trường hợp này cần đối chiếu với nhãn eval để xác định mô hình có phân loại sai hay không

Precision đạt 41,05% và recall đạt 23,03%, cho thấy khả năng phát hiện trên eval còn hạn chế. Các chỉ số này chưa đủ để kết luận nguyên nhân do nhãn, thiếu dữ liệu hay mất cân bằng lớp. F1 annotation ở CP5 và mAP của model ở CP6 đo hai đối tượng khác nhau

### Trả lời bốn câu hỏi cuối notebook

**1. Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `bus` có Recall thấp nhất, bằng **0**. Các lớp còn lại có Recall lần lượt khoảng 0.638 (`car`), 0.200 (`truck`) và 0.083 (`van`).
   - Chọn `drive_051` để minh họa. Trong nhãn eval có một đối tượng thuộc lớp `bus` ở phía trên bên phải ảnh, nhưng ảnh prediction không hiển thị bounding box thuộc lớp `bus` tại ngưỡng confidence sử dụng.
   - Điều này cho thấy mô hình chưa nhận diện tốt lớp `bus` trong trường hợp trên. Tuy nhiên, tập eval chỉ có 2 đối tượng bus nên kết quả theo lớp dễ bị ảnh hưởng bởi từng trường hợp cụ thể

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm tra điều gì trên box?**
   - Cần kiểm tra vị trí và độ ôm sát của bounding box, chẳng hạn box bị lệch tâm, quá rộng, quá hẹp hoặc chứa nhiều phần nền không cần thiết
   - mAP50 đánh giá ở ngưỡng IoU 0.50, còn mAP50–95 sử dụng nhiều ngưỡng từ 0.50 đến 0.95 nên yêu cầu cao hơn về độ chính xác của box
   - Trong bài Lab này, mAP50 đạt 0.2723 và mAP50–95 đạt 0.2017. Cả hai đều còn thấp, vì vậy ngoài vị trí bounding box, tôi cũng cần xem xét khả năng phát hiện và phân loại của mô hình

**3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Các nhóm `car/van` và `truck/bus` là những trường hợp dễ nhầm lẫn.
   - Trong khi self-QC, tôi đã sửa `drive_019` từ `car` sang `van` và `drive_044` từ `truck` sang `bus`. Cho thấy việc phân loại những phương tiện có kiểu dáng tương tự nhau cần được kiểm tra kỹ hơn
   - Ở kết quả prediction, ảnh `drive_051` cũng có một xe con màu đỏ được dự đoán thành `van` với confidence 0.99. Cho thấy confidence cao không đồng nghĩa với dự đoán đúng class
   - Ngoài ra, dữ liệu train chưa cân bằng: 731 nhãn `car` nhưng chỉ có 26 `truck` và 31 `van`. Theo tôi, cần bổ sung dữ liệu và làm rõ hơn các tiêu chí phân loại trong guideline để cải thiện kết quả

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Tập train được sử dụng để mô hình học và cập nhật trọng số, còn 10 ảnh eval đóng vai trò validation để theo dõi kết quả huấn luyện và lựa chọn checkpoint tốt nhất
   - Nếu đưa ảnh eval vào train, mô hình sẽ học trực tiếp từ dữ liệu dùng để đánh giá, khiến kết quả không còn phản ánh đúng khả năng xử lý dữ liệu chưa được dùng để cập nhật trọng số
   - Trong notebook, giữ nguyên cách chia dữ liệu: 50 ảnh train và 10 ảnh eval. Do tập eval đã được sử dụng để lựa chọn `best.pt`, đây là validation set chứ không phải test set độc lập

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Annotation | Một số phương tiện có kiểu dáng tương tự nhau, dễ nhầm class | Kiểm tra lại 10 ảnh trên CVAT và đối chiếu với schema | Sửa 2 trường hợp phân loại ở `drive_019` và `drive_044` |
| Self-QC | Cần xác nhận bộ nhãn đúng định dạng YOLO trước khi đánh giá | Chạy script `check_yolo_labels.py` | 50/50 file nhãn hợp lệ |
| Gold QC | F1 = 0.5864, chưa đạt ngưỡng mặc định 0.80 | Chạy lại evaluator với ngưỡng F1 = 0.50 theo điều chỉnh của Lab Coach | `passed: true`, không thay đổi bộ nhãn và các chỉ số |
| Tổng hợp kết quả CP6 | File JSON chứa metrics nằm trong thư mục kết quả giải nén, chưa ở vị trí dự kiến | Kiểm tra thư mục `outputs/day2_colab_outputs/` và tìm file `model_eval_metrics.json` | Đã lấy được các chỉ số đánh giá và bổ sung vào báo cáo |
| Phân tích kết quả YOLO | Recall tổng hợp thấp, chưa rõ lớp nào hoạt động kém | Xem kết quả validation theo lớp, biểu đồ và ảnh prediction | Xác định `bus` có Recall thấp nhất; `van` cũng có Recall thấp |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md), tự đánh giá kết quả bài Lab như sau:

| Hạng mục | Kết quả và minh chứng |
|---|---|
| Annotation | Hoàn thành 50 ảnh train, tổng cộng 875 bounding box |
| Định dạng YOLO | 50/50 file nhãn hợp lệ khi chạy checker |
| Self-QC | Đã kiểm tra trực quan 10 ảnh trên CVAT và sửa 2 lỗi phân loại |
| Mini guideline | Có định nghĩa 4 lớp, quy tắc gán nhãn và 3 trường hợp mơ hồ |
| Gold QC | Đạt F1 = 0.5864 và Mean IoU = 0.8515 theo ngưỡng Lab Coach điều chỉnh |
| Rework | Không chỉnh sửa nhãn sau Gold QC vì đã đạt ngưỡng yêu cầu |
| YOLO pipeline | Hoàn thành train 50 epochs, đánh giá và prediction trên 10 ảnh eval |
| Báo cáo | Có số liệu, minh chứng và phân tích kết quả thực tế |

**Tự nhận xét:**
   - Qua bài Lab, đã thực hành quy trình từ chuẩn bị dữ liệu, annotation, kiểm tra chất lượng nhãn đến huấn luyện và đánh giá mô hình Object Detection
   - Nhận thấy checker đạt không đồng nghĩa với chất lượng nhãn tốt, và confidence cao cũng không đảm bảo mô hình dự đoán đúng class
   - Cần cải thiện khả năng phân biệt các phương tiện dễ nhầm lẫn, kiểm tra annotation kỹ hơn và hiểu rõ hơn các chỉ số đánh giá mô hình. Ngoài ra, bộ nhãn vẫn còn nhiều FP và FN nên cần tiếp tục rà soát nếu muốn nâng chất lượng dữ liệu

## 8. Công cụ AI và tài liệu tham khảo

**Công cụ AI đã sử dụng:**
   - **Kiểm tra và đánh giá:** Sử dụng Codex hỗ trợ chạy checker, evaluator và tổng hợp kết quả Gold QC
   - **Huấn luyện và phân tích:** Sử dụng AI hỗ trợ đọc kết quả Colab, giải thích các chỉ số Precision, Recall, mAP và phân tích ảnh prediction
   - **Báo cáo:** Sử dụng AI hỗ trợ tổng hợp số liệu và chỉnh sửa cách trình bày. Các chỉ số được lấy từ kết quả thực hành, không sử dụng số liệu minh họa thay cho kết quả thực tế
**Tài liệu tham khảo:**
   - [GUIDE.md](../../GUIDE.md) – Hướng dẫn bài Lab.
   - [CHECKPOINTS.md](../../CHECKPOINTS.md) – Các checkpoint thực hành.
   - [RULES.md](../../RULES.md) – Quy định thực hiện.
   - [RUBRIC.md](../../RUBRIC.md) – Tiêu chí đánh giá.
   - [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) – Guideline cá nhân.
   - Notebook `day2_annotation_to_yolo.ipynb` và kết quả thực tế từ Google Colab.
