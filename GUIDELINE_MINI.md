# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Ngô Xuân Nam
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm: vẫn gán xe thật đang đỗ dù không di chuyển; không gán biển chỉ đường, rào chắn, thùng giao thông hoặc vật thể có hình dáng/màu sắc dễ bị nhầm thành xe. Không dùng kết quả của model làm đáp án; model chỉ dùng để chỉ ra frame cần xem lại.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame**, với điều kiện hướng chuyển động, vị trí và đặc điểm xe trước/sau khi bị che vẫn khớp. | Khoảng che dưới 2 giây ở 12.5 fps còn đủ ngắn để duy trì identity và tránh tách một xe thành nhiều track. |
| Xe bị che lâu hơn ngưỡng trên | Mở track với ID mới, trừ khi có bằng chứng liên tục rất rõ và đã ghi chú để cả nhóm áp dụng giống nhau. | Sau hơn 25 frame, độ chắc chắn về identity giảm; dùng ID mới an toàn và nhất quán hơn suy đoán. |
| Xe rời khung hình rồi quay lại | Dùng **track mới**, không nối lại ID cũ. | Khi xe đã ra hoàn toàn khỏi ảnh, không có bằng chứng quan sát liên tục để khẳng định xe quay lại là cùng một chiếc. |
| Hai xe cắt nhau / chồng lên nhau | Không gộp track. Giữ mỗi xe một ID, đối chiếu hướng/tốc độ, vị trí trước và sau giao cắt, màu và hình dáng; đặt keyframe sát hai phía của đoạn chồng lấp. | Dựa vào cả chuỗi frame giúp tránh đổi ID khi IoU hoặc vị trí tức thời trở nên mơ hồ. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu từ frame đầu tiên có thể xác định là xe bốn bánh khi xem ở mức zoom 100%; phải thấy được thân/dáng xe qua ít nhất hai frame liền kề, không quyết định chỉ dựa vào một đốm màu. |
| Xe đang đỗ, không di chuyển | Vẫn gán suốt thời gian xe thật còn nhìn thấy; kiểm tra hình dáng và bối cảnh để phân biệt với biển báo hoặc vật thể tĩnh khác. |
| Keyframe đặt dày ở đâu | Đặt dày ở lúc xe mới vào/chuẩn bị ra khỏi khung, đổi hướng hoặc tốc độ, bị che, giao cắt với xe khác và khi bbox nội suy bắt đầu trôi. Với đoạn khó dùng khoảng 2–5 frame/keyframe; đoạn đi thẳng đều có thể thưa hơn nhưng phải kiểm tra frame giữa. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, frame 14–16, ID 2
- Tình huống: Xe taxi đi sát ba thùng giao thông màu vàng nên màu sắc và một phần đường bao dễ bị lẫn với các thùng.
- Quyết định: Giữ ID 2 bằng cách đối chiếu vị trí ở các frame trước/sau; bbox chỉ ôm phần xe nhìn thấy, không kéo bbox bao gồm các thùng.
- Lý do: Chuyển động liên tục qua chuỗi frame xác nhận đó vẫn là cùng một xe; các thùng là vật thể tĩnh và nằm ngoài lớp `vehicle`.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 81–120, bus ID 4; xe ID 5 và ID 6
- Tình huống: Xe buýt lớn che một phần các xe ở làn xa, khiến bbox và identity của hai xe nhỏ khó theo dõi; riêng ID 6 có bbox trôi ở frame 108–112.
- Quyết định: Giữ ID riêng cho từng xe, bbox theo phần còn nhìn thấy và đặt keyframe dày hơn trước, trong và sau đoạn bị che. Không nội suy một đoạn dài qua xe buýt.
- Lý do: Hướng chuyển động và vị trí trước/sau che khuất vẫn liên tục, nhưng chấm với gold cho thấy bbox ID 6 ở frame 108, 110–112 chỉ còn IoU gần ngưỡng 0.5–0.6.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 1–190, ID 3
- Tình huống: Xe màu trắng đỗ gần như đứng yên trong toàn bộ clip, dễ bị cho là nền hoặc nhầm với false positive tĩnh.
- Quyết định: Vẫn gán ID 3 liên tục vì đây là xe thật và còn nhìn thấy; không kết thúc track chỉ vì xe không chuyển động.
- Lý do: Phạm vi gán nhãn gồm cả xe đang đỗ. Cần dựa vào hình dáng và bối cảnh, không dùng chuyển động làm điều kiện bắt buộc.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Phải bắt đầu track ngay từ frame đầu tiên đủ nhận diện, kể cả xe còn nhỏ. Gold cho thấy ID 6 xuất hiện từ frame 101, trong khi bản pre-gold của tôi bắt đầu ở frame 108, làm bỏ sót frame 101–107. ReID chỉ giúp phát hiện chỗ đáng xem lại; dù model dùng các ID 24 → 28 → 31 không nhất quán, gold mới là căn cứ xác nhận.
- Khi xe nhỏ, bị che hoặc đi gần xe lớn, phải thêm keyframe quanh đoạn khó và kiểm tra frame giữa. Với ID 6, cần rà kỹ frame 108–112 để tránh bbox nội suy trôi và chỉ ôm phần xe thực sự nhìn thấy.
- Không gán vật thể tĩnh chỉ vì model nhận thành xe. ReID ID 7 tạo 43 bbox trên bảng chỉ đường trong khoảng frame 16–116; phải kiểm tra hình dáng, vị trí và bối cảnh để loại FP, đồng thời vẫn giữ nhãn cho xe thật đang đỗ như ID 3.
- Sau mỗi lần kiểm chéo, reviewer phải ghi rõ `frame – ID – loại lỗi – cách sửa` vào `reports/review_partner.md`; nếu hai người quyết định khác nhau thì cập nhật luật tại đây trước khi tiếp tục gán các clip sau.
