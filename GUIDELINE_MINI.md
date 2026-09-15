# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: **Vương Trọng Nghĩa — 2A202602090**
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

Bổ sung của nhóm (nếu có): làm cá nhân; dùng Rectangle → Track với nhãn `vehicle`. Frame trong các ca bên dưới là MOT (bắt đầu từ 1); frame CVAT tương ứng bằng MOT trừ 1. Các quy tắc được tổng hợp và làm rõ sau khi đối chiếu kết quả ngày 15/09/2026.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID khi vẫn theo dõi được xe. Nếu mất dấu hoàn toàn rồi hiện lại trong **dưới 25 frame** (2 giây @ 12.5 fps), nối lại ID cũ khi vị trí, hướng đi và hình dáng phù hợp. Trong khoảng hoàn toàn không nhìn thấy, dùng Outside; khi xuất hiện lại, tiếp tục track cũ và tắt Outside. | Che khuất tạm thời không làm chiếc xe trở thành đối tượng mới; không vẽ bbox vô hình trên vật cản. |
| Xe bị che lâu hơn ngưỡng trên | Với khoảng mất dấu từ 25 frame trở lên, mở track mới; dùng mốc này để xử lý nhất quán cả trường hợp đúng 25 frame. Nếu không đủ căn cứ nhận diện cùng xe trong khoảng ngắn hơn, đánh dấu ca cần kiểm tra thay vì nối theo phỏng đoán. | Khoảng mất dấu dài làm tăng nguy cơ nối nhầm hai xe; đây là quy ước ngưỡng áp dụng cho bài. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Rời hẳn ảnh là kết thúc track, khác với việc tạm đi sau vật cản bên trong ảnh. |
| Hai xe cắt nhau / chồng lên nhau | Giữ một ID riêng cho từng xe; so sánh vị trí và hướng chuyển động trước/sau giao cắt, kết hợp màu và hình dáng. Không đổi ID theo thứ tự trái/phải và không gộp bbox của hai xe. | Thứ tự vị trí có thể đổi nhưng identity của từng xe vẫn giữ nguyên. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng chọn theo khả năng nhận diện thân xe và phân biệt với xe máy/vật nền, không đặt ngưỡng pixel tùy ý. Xem frame liền trước và liền sau để chọn mốc; chưa nhận diện được thì chưa gán. |
| Xe đang đỗ, không di chuyển | Vẫn gán `vehicle`, giữ cùng ID trong thời gian xe hiện diện. Cảnh báo bbox đứng im của validator cần kiểm bằng ảnh, không tự xóa track. |
| Keyframe đặt dày ở đâu | Quanh lúc xe xuất hiện/rời ảnh, đi sau cột hoặc xe khác, giao cắt, rẽ hoặc thay đổi kích thước nhanh. Kiểm từng frame quanh chuyển tiếp; thêm keyframe khi nội suy không còn ôm phần nhìn thấy. Khi xe đi đều, dùng khoảng keyframe thưa hơn và kiểm các frame giữa. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, MOT **190** / CVAT **189**, nhãn tay **ID 3** (xe đỗ màu sáng; gold ID 1, treatment ID 3).
- Tình huống: xe tải đi phía trước che một phần xe đang đỗ. Phần xe nhìn thấy thu hẹp, dễ nhầm giữa bbox toàn thân và bbox phần còn lộ ra.
- Quyết định: giữ ID 3 và dùng bbox phần nhìn thấy theo quy tắc lab; không mở rộng theo kích thước xe trước lúc bị che. Bản nhãn ở frame này rộng 53,30 pixel; giữ nguyên bản hiện tại trong lúc đối chiếu geometry, chưa rework theo gold/model.
- Lý do: xe vẫn là cùng một đối tượng dù bị che. Reference rộng 93,92 pixel và treatment rộng 61,45 pixel, nên cần đối chiếu quy tắc phần nhìn thấy thay vì lấy bbox rộng hơn làm đáp án.

### Ca 2
- Clip / frame / ID: `clip_01`, MOT **99–103** / CVAT **98–102**, nhãn tay **ID 5**.
- Tình huống: xe con đi từ phải sang trái qua phía sau cột giữa ảnh. Ở frame 99 xe còn ở bên phải cột; quanh frame 101 cột che một phần thân xe; đến frame 103 phần đầu xe đã sang trái cột. Đây là che khuất tạm thời một phần, không phải xe rời ảnh hoặc biến mất hoàn toàn.
- Quyết định: giữ ID 5 xuyên suốt đoạn đi sau cột, theo dõi bbox ở các frame trước/trong/sau che. Không bật Outside khi vẫn nhìn thấy xe. Nếu gặp frame bị che hoàn toàn ở tình huống tương tự, bật Outside cho khoảng vắng mặt và nối lại cùng ID khi đủ căn cứ, theo ngưỡng đã ghi ở mục 2.
- Lý do: hướng di chuyển liên tục, vị trí trước/sau cột và hình dáng cho thấy cùng chiếc xe. Bản MOT giữ ID 5 trong cả đoạn; phần thân bị cột che không phải một xe mới.

### Ca 3
- Clip / frame / ID: `clip_01`, MOT **149–151** / CVAT **148–150**, nhãn tay **ID 4** (xe buýt rời mép trái; treatment ID 9).
- Tình huống: xe gần rời hẳn khung hình, chỉ còn một dải rất hẹp sát mép trái. Bbox nhãn tay thu từ khoảng 76,28 pixel chiều rộng ở frame 149 xuống 17,69 pixel ở frame 151; reference đã kết thúc tại frame 148.
- Quyết định: bbox cắt đúng mép ảnh; còn nhìn thấy xe thì giữ track, frame đầu không còn thấy xe thì dùng Outside. Giữ ca này trong danh sách cần đối chiếu endpoint với Lab Coach vì ảnh frame 151 vẫn còn một phần xe; chưa sửa annotation chỉ để khớp điểm reference. Xe đã ra hẳn rồi quay lại sẽ dùng ID mới.
- Lý do: cần phân biệt phần xe còn nằm trong ảnh với bbox treo sau khi xe đã đi hết. Bấm Outside quá sớm làm mất phần còn thấy, quá muộn tạo bbox thừa.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Làm rõ che khuất và nối ID:** phân biệt che một phần (bbox phần nhìn thấy, giữ ID) với vắng mặt hoàn toàn (Outside). Ghi rõ số frame mất dấu, căn cứ nối lại ID và cách xử lý mốc 25 frame; không suy ra xe mới chỉ vì model đổi ID. Ca xe đi sau cột ở ID 5, frame 99–103 là ví dụ để kiểm lại tính liên tục.
- **Làm rõ geometry và endpoint sau chấm:** dùng ca ID 3/frame 190 để thống nhất bbox phần nhìn thấy và ca ID 4/frame 149–151 để thống nhất lúc kết thúc track. Đánh giá hiện tại có 43 FP, 7 FN, 0 IDSW; annotation vẫn giống snapshot pre-gold, chưa rework. Phần kiểm chéo sẽ bổ sung khi có reviewer và biên bản; các quy tắc trên được làm rõ từ lần đối chiếu hiện tại.
