# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Ngô Xuân Nam 
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 20 phút |
| Thời gian gán `clip_01` | 40 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 2 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Có 2 xe bị che bởi xe buýt ở frame 81–120; tôi đợi đến khi xác định rõ vị trí xuất hiện của từng xe rồi mới đánh dấu và giữ đúng ID.
2. Ở frame 14, xe taxi bị che bởi 3 chiếc thùng vàng có màu gần giống nên dễ nhầm lẫn; tôi đối chiếu vị trí của xe từ các frame trước và giữ bbox ổn định.
3. Ở frame 108–112, xe ID 6 còn nhỏ và di chuyển ở làn xa nên bbox nội suy bị trôi, đặc biệt khi xe buýt lớn đi qua vùng quan sát. Tôi giữ nguyên ID của xe và cần đặt keyframe dày hơn, đồng thời chỉnh bbox ôm sát phần xe nhìn thấy ở các frame này.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `dfd13cbf026e2602e1b8e84d3d39ea3a356150572f6d2f53a2daf7742dd1e0c2` |
| Thời điểm khóa | `2026-09-15T10:47:14.517253+00:00` (17:47:14, GMT+7) |
| Số row / frame / track trước khi mở reference | 542 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8321 | 0.8219 | 0.8437 | 0.8827 | 0.9704 | 0.9424 | 0.8722 | 1 | 32 | 0 |
| Sau rework | 0.8321 | 0.8219 | 0.8437 | 0.8827 | 0.9704 | 0.9424 | 0.8722 | 1 | 32 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

Lưu ý: file annotation hiện tại có cùng SHA-256 với bản pre-gold, nên chưa có rework thực tế; vì vậy hai hàng metric đang giống nhau.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox trôi | 108 | 6 | Chưa sửa trong bản hiện tại; cần thêm keyframe và chỉnh bbox ôm sát xe. |
| Bbox trôi | 110–112 | 6 | Chưa sửa trong bản hiện tại; cần đặt keyframe dày hơn để tránh nội suy lệch. |
| ID switch / tách track / bbox thừa | Toàn clip | Các ID | Không phát hiện lỗi thuộc các nhóm này; kết quả có 0 IDSW, đủ 8/8 track và không có ghost track. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Chưa có `outputs/model_run_config.json` để xác nhận version |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.70` / `960` / `[2, 5, 7]` |
| device | Chưa có `outputs/model_run_config.json` để xác nhận |

  So sánh      HOTA    DetA    AssA    LocA    IDF1    MOTA    MOTP      FP      FN    IDSW
------------------------------------------------------------------------------------------------
ban_vs_gold        0.832   0.822   0.844   0.883   0.970   0.942   0.872       1      32       0
bytetrack_vs_gold   0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2
reid_vs_gold       0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2
reid_vs_ban        0.783   0.726   0.845   0.882   0.907   0.797   0.872     103       7       0

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi thấp hơn IDF1: MOTA = 0.9424, còn IDF1 = 0.9704. Kết quả này cho thấy bản nhãn giữ identity tốt (0 ID switch), nhưng vẫn còn 32 FN và 1 FP làm MOTA giảm. Nếu MOTA cao mà IDF1 thấp thì thường nhãn có vấn đề về identity/association, chẳng hạn một xe bị tách thành nhiều ID hoặc đổi ID trên một đoạn dài. MOTA không phạt nặng lỗi này vì mỗi lần ID switch chỉ bị cộng một lỗi, bất kể ID sai tiếp tục trong bao nhiêu frame; IDF1 đánh giá sự nhất quán danh tính trên toàn bộ quãng đời của track nên phản ánh lỗi ID rõ hơn.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack về IDF1 (0.9001 so với 0.8746, tăng 0.0255) và AssA (0.8204 so với 0.7761, tăng 0.0443), còn IDSW không đổi: cả hai đều có 2 lần đổi ID. Một sequence thể hiện treatment tốt hơn là gold track 4 ở frame 55–63: ReID giữ cùng ID 9 liên tục từ frame 55 trở đi, trong khi ByteTrack chỉ bắt được bằng ID 14 ở frame 56–57, mất ở frame 58 rồi chuyển sang ID 15 từ frame 59. Điều này góp phần giải thích AssA và IDF1 của treatment cao hơn. Tuy nhiên đây là so sánh hai hệ thống hoàn chỉnh; không thể quy toàn bộ cải thiện cho riêng ReID vì ByteTrack và BoT-SORT còn khác implementation và cách quản lý track.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.6487 ở ByteTrack lên 0.7110 ở ReID. FP tăng nhẹ từ 88 lên 91 (+3), nhưng FN giảm mạnh từ 54 xuống 26 (-28), nên treatment bao phủ được nhiều bbox thật hơn dù sinh thêm một ít bbox thừa. Lỗi còn lại đến từ cả detection và association: 91 FP cùng 26 FN cho thấy detection/output filtering vẫn là nguồn lỗi lớn, trong khi 2 IDSW và 3 gold track bị tách ở ReID cho thấy association vẫn chưa giải quyết xong. Vì detector input được giữ cố định nhưng tracker và cách duy trì/xuất track khác nhau, chênh lệch FP/FN này phải được hiểu là khác biệt ở đầu ra toàn hệ thống, không phải bằng chứng rằng bản thân detector đã thay đổi.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở frame 60 (thuộc chuỗi frame 16–116), ReID ID 7 đặt bbox gần như cố định tại khoảng `(497, 213, 98, 58)` lên bảng chỉ đường phía trên đường. Đây là vật thể tĩnh, không phải xe bốn bánh; cả nhãn của tôi và gold đều không gán bbox ở đó. Vì vậy tôi đúng khi bỏ qua đối tượng này, còn ReID tạo 43 bbox false positive rải trong khoảng frame 16–116.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID làm tôi xem lại thời điểm bắt đầu của xe ID 6. Ở frame 104, ReID ID 24 đã phát hiện xe tại khoảng `(710, 252, 67, 17)`; frame 107, ReID ID 28 tiếp tục phát hiện xe này, trong khi nhãn của tôi chỉ bắt đầu ID 6 từ frame 108. Gold xác nhận xe ID 6 đã xuất hiện từ frame 101 và tại frame 104 có bbox `(707, 252, 66, 29)`. Như vậy nhãn của tôi đã bắt đầu track muộn và bỏ sót frame 101–107. Dù ReID cũng chưa giữ được ID nhất quán ở đoạn này (24 → 28 → 31), các bbox của model là bằng chứng hữu ích để phát hiện FN trong annotation.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Nếu phải gán thêm 10 clip, tôi sẽ hoàn thiện `GUIDELINE_MINI.md` trước khi bắt đầu: ghi rõ xe bị che dưới 25 frame thì giữ ID, quá 25 frame thì mở track mới; xe đã rời khung rồi xuất hiện lại phải dùng ID mới; khi hai xe chồng nhau phải theo dõi hướng chuyển động, vị trí trước/sau che khuất và đặc điểm ngoại hình để giữ đúng ID. Tôi cũng sẽ bổ sung luật bbox cho xe nhỏ, xe bị che và xe đứng yên, đồng thời quy định đặt keyframe dày hơn ở đoạn xe đổi hướng, bị che, đi nhanh hoặc nội suy bắt đầu trôi—đặc biệt là tình huống giống ID 6 ở frame 108–112.

Về quy trình, tôi sẽ làm xong từng xe từ lúc xuất hiện đến lúc `outside`, lưu bằng nút Save rồi reload để xác nhận dữ liệu còn nguyên. Sau mỗi clip, tôi sẽ chạy kiểm tra định dạng, xem lại ba lượt (ID; frame đầu/cuối; đoạn giữa các keyframe), kiểm chéo với bạn cùng nhóm và sửa hết lỗi bbox trước khi khóa pre-gold. Tôi cũng sẽ lưu ngay các output JSON/MOT sau khi chạy model để có đủ bằng chứng trả lời phần so sánh ByteTrack và ReID.

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
