# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: **Vương Trọng Nghĩa**
MSSV: **2A202602090**
Ngày tổng hợp: **15/09/2026**
Công cụ: CVAT tự host và Google Colab (GPU T4).

Báo cáo tổng hợp bản nhãn đã khóa, kết quả đánh giá trong repo và lần chạy mới trong [notebook Colab](https://colab.research.google.com/drive/1oMiY09kHddP2kTy3-Ecpo4Y4Nw-Y9qgY). Lần chạy này đã hoàn thành cả ByteTrack control và BoT-SORT + ReID treatment, cùng thí nghiệm ngưỡng appearance 0,70 / 0,80 / 0,90. Các bảng model dưới đây lấy từ notebook mới, không trộn với kết quả của notebook cũ. Phần annotation vẫn qua cổng chất lượng.

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT tại `localhost:8080`, Rectangle Track |
| Project | `DAY03-SOLO-2A202602090` |
| Warm-up | Job #6, `clip_02`: 60 frame, 6 track, 238 bbox trong bản MOT |
| Bài chính | Job #7, `clip_01`: 190 frame, 8 track, 609 bbox trong bản MOT |
| Độ phân giải / tốc độ dữ liệu | 960 × 540, 12,5 fps |
| Nhãn | `vehicle`: xe bốn bánh |
| Định dạng export | MOT 1.1; frame từ 1, cột thứ hai là track ID |
| Thời gian gán `clip_02` | Chưa ghi nhận thời lượng |
| Thời gian gán `clip_01` | Chưa ghi nhận thời lượng |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | Chưa thống kê; bản MOT không lưu cờ keyframe |

Hai file export ban đầu là `nghia-mot11-1.zip` cho `clip_02` và `nghia-mot11-2.zip` cho `clip_01`. Nhãn được đặt vào `annotations/<clip>/gt.txt`. Bản `clip_01` được khóa trước khi nhận reference và trước các lần inference thể hiện trên Colab.

Ba tình huống khó nhất tôi gặp khi gán nhãn là xe bị che một phần, xe tạm thời khuất sau một vật thể rồi xuất hiện lại, và xe rời khỏi khung hình. Với mỗi tình huống, điểm cần quyết định và nguyên tắc xử lý là:

1. **Xe bị che khuất một phần:** khó xác định biên bbox khi một phần thân xe bị xe khác hoặc vật thể phía trước che mất. Nguyên tắc là giữ ID của chiếc xe và khoanh phần còn nhìn thấy theo guideline của lab, không đoán phần thân bị che. Cần kiểm tra các frame lân cận và đặt thêm keyframe khi phần nhìn thấy thay đổi nhanh để bbox nội suy không trôi sang xe khác.
2. **Xe chạy qua phía sau một vật thể, tạm thời biến mất rồi xuất hiện lại:** khó nhất là xác định xe xuất hiện lại có phải chiếc xe vừa mất dấu hay không. Cần đối chiếu hướng di chuyển, vị trí trước/sau vật cản, hình dáng và thời gian bị che. Nếu đủ căn cứ là cùng xe và khoảng che nằm trong ngưỡng của lab (25 frame, khoảng 2 giây), giữ cùng ID khi xe xuất hiện lại; đánh dấu khoảng không nhìn thấy để tránh bbox treo trên vật cản. Khi không đủ căn cứ hoặc vượt ngưỡng, cần xem lại quy tắc ID thay vì nối theo phỏng đoán.
3. **Xe rời khỏi khung hình:** khó chọn frame kết thúc khi xe chỉ còn một phần rất nhỏ sát mép ảnh. Bbox cần được cắt đúng rìa ảnh, không kéo ra ngoài; khi xe không còn nhìn thấy thì kết thúc bằng Outside để không tiếp tục nội suy. Nếu xe đã rời khung rồi quay lại, áp dụng track mới theo quy tắc của lab. Đây cũng là điểm cần xem lại ở nhãn ID 4, MOT frame 149–151, được phân tích cụ thể ở mục 3 và mục 5.

Các nguyên tắc trên mô tả cách xử lý ba tình huống khó; trạng thái sửa nhãn sau khi chấm được ghi riêng ở mục 3. Bản nhãn hiện tại vẫn giữ nguyên so với snapshot pre-gold.

## 2. Tự kiểm và kiểm chéo

### Kiểm tra định dạng

| Clip | Bbox | Track | Lỗi | Cảnh báo |
| --- | ---: | ---: | ---: | ---: |
| `clip_02` | 238 | 6 | 0 | 2 |
| `clip_01` | 609 | 8 | 0 | 1 |

Validator cảnh báo bbox gần như đứng im tại MOT frame 1–15: ID 2 và ID 3 của `clip_02`, ID 3 của `clip_01`. Xe đang đỗ vẫn cần được gán nhãn, nên cảnh báo này không tự chứng minh bbox sai hoặc quên Outside.

Ba lượt tự kiểm cần tập trung vào các điểm sau từ kết quả đánh giá:

- Lượt 1 — ID và timeline: nhãn tay có IDSW = 0, không có track gold bị bỏ hoàn toàn hoặc bị phân mảnh trong chẩn đoán.
- Lượt 2 — Frame đầu/cuối: xem lại ID 6 bắt đầu sớm và ID 4 kết thúc muộn so với reference.
- Lượt 3 — Frame giữa và hình học bbox: xem lại 19 cặp bbox có độ khít thấp, nhất là các đoạn che khuất và nội suy. MOTP tổng thể là 0,8758.

Kiểm chéo với: chưa ghi nhận. Biên bản `reports/review_partner.md` chưa được bổ sung.
Số lỗi tìm được trong bản của bạn cùng review và số lỗi bạn ấy tìm được trong bản của tôi: chưa thống kê.

Các luật cần làm rõ trong `GUIDELINE_MINI.md`: thời điểm bắt đầu track cho xe nhỏ/mờ, thời điểm kết thúc khi xe chạm rìa, và bbox phần nhìn thấy khi xe bị che. Các điểm này sẽ được đối chiếu khi kiểm chéo.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| Snapshot | `evidence/pre-gold/clip_01/gt.txt` |
| Manifest | `evidence/pre-gold/clip_01/manifest.json` |
| SHA-256 | `ddb34e3b869204645b52c729d9fe75c176e92e39f73bde3bbf4b797ec950c81e` |
| Thời điểm khóa (UTC) | 2026-09-15 03:40:02.183196 |
| Thời điểm khóa (Việt Nam) | 15/09/2026 10:40:02 |
| Số row / frame / track | 609 / 190 / 8 |
| Dung lượng snapshot | 36.018 byte |
| Reference | `gold/clip_01/gt.txt`, do Lab Coach cung cấp; 573 bbox, 8 track |
| Kiểm tra tính toàn vẹn | Hash snapshot khớp manifest; nhãn hiện tại giống snapshot từng byte |

| Bản nhãn | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Pre-gold | 0,8198 | 0,8013 | 0,8405 | 0,8883 | 0,9577 | 0,9127 | 0,8758 | 43 | 7 | 0 |
| Hiện tại — chưa rework | 0,8198 | 0,8013 | 0,8405 | 0,8883 | 0,9577 | 0,9127 | 0,8758 | 43 | 7 | 0 |

**Qua cổng: có.** IDF1 ≥ 0,80; MOTA ≥ 0,75; MOTP ≥ 0,70. Hai dòng giống nhau vì tôi chưa rework annotation sau khi khóa. Nguồn: [eval_pre_gold.json](../outputs/eval_pre_gold.json) và [eval_vs_gold.json](../outputs/eval_vs_gold.json).

| Loại finding | MOT frame | CVAT frame | ID | Trạng thái và hướng xử lý |
| --- | --- | --- | --- | --- |
| Bắt đầu sớm so với reference | 82–100 | 81–99 | Nhãn 6 | Chưa sửa; xem lại thời điểm xác định được xe, rồi quyết định ranh giới track |
| Kết thúc muộn so với reference | 149–151 | 148–150 | Nhãn 4 | Chưa sửa; kiểm tra frame xe rời khung và Outside |
| Bbox khít chưa tốt | 108–109 | 107–108 | Gold 6, cần xem cặp ghép | Chưa sửa; xem phần nhìn thấy và nội suy quanh đoạn này |
| Bbox khít chưa tốt ở cuối clip | 190 | 189 | Gold 1, cần xem cặp ghép | Chưa sửa; kiểm tra bbox ở frame cuối |

ID gold và ID nhãn không bắt buộc có cùng số. Khi mở CVAT cần nhận diện đúng xe tương ứng; không sửa chỉ dựa vào việc hai ID khác số. CVAT hiển thị frame từ 0, còn MOT từ 1.

## 4. Kết quả model: ByteTrack control vs ReID treatment

### Cấu hình thực tế đã chạy

Nguồn: output “Cấu hình đã khóa” của ô 8, lần thực thi `[31]` trên notebook mới. Ô này đã ghi `outputs/model_run_config.json` trên Colab. Trạng thái lưu file về repo được ghi riêng ở mục 7.

| Mục | Giá trị thực chạy |
| --- | --- |
| Python | 3.13.15 |
| Ultralytics | 8.4.145 — khớp bản pin của notebook mới |
| torch | 2.11.0+cu128 |
| lap | 0.5.13 |
| Weights | `yolo26n.pt` |
| Control | `bytetrack.yaml` |
| Treatment | `/content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| Cấu hình treatment trong repo | `with_reid: true`, `model: auto`, `proximity_thresh: 0.50`, `appearance_thresh: 0.80`, `gmc_method: none` |
| Confidence / NMS IoU / imgsz | 0,25 / 0,70 / 960 |
| COCO classes | `[2, 5, 7]`: car, bus, truck |
| Device | GPU T4, `device="0"` |
| Giữ trạng thái giữa frame | `persist=True` |
| Dữ liệu | 190 frame của `clip_01`, sắp theo tên ảnh |
| Đầu ra control | `model_bytetrack_clip_01.txt`: 607 bbox, 16 track |
| Đầu ra treatment | `model_reid_clip_01.txt`: 638 bbox, 16 track |

Hai run giữ nguyên ảnh đầu vào, weights, conf, IoU, kích thước ảnh và classes. Đây là so sánh hai hệ thống tracking khác implementation, không phải thí nghiệm chỉ bật/tắt ReID trên cùng một tracker.

### Bảng tổng hợp từ notebook mới

Số liệu được chép từ mục 4 của Colab, ô 14, lần thực thi `[35]`; giữ độ chính xác ba chữ số thập phân mà notebook hiển thị.

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Nhãn tay vs gold | 0,820 | 0,801 | 0,841 | 0,888 | 0,958 | 0,913 | 0,876 | 43 | 7 | 0 |
| ByteTrack control vs gold | 0,709 | 0,649 | 0,776 | 0,846 | 0,875 | 0,749 | 0,823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0,763 | 0,711 | 0,820 | 0,872 | 0,900 | 0,792 | 0,860 | 91 | 26 | 2 |
| ReID vs nhãn tay | 0,801 | 0,745 | 0,861 | 0,926 | 0,888 | 0,777 | 0,920 | 81 | 52 | 3 |

Phép so ReID với nhãn tay dùng nhãn tay làm tham chiếu: FP/FN thể hiện bất đồng với nhãn tay, không tự chứng minh model hoặc người gán đúng. Không so trực tiếp điểm ở dòng này với điểm vs gold để kết luận chất lượng tăng, vì bộ tham chiếu đã khác.

### Thí nghiệm bổ sung: ngưỡng appearance của ReID

`RUN_EXPERIMENT=True`; ô 21, lần thực thi `[39]`, đã chạy ba ngưỡng. Conf vẫn là 0,25; chỉ ngưỡng appearance trong cấu hình treatment được thay đổi.

| appearance_thresh | HOTA | DetA | AssA | IDF1 | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0,70 | 0,763 | 0,711 | 0,820 | 0,900 | 91 | 26 | 2 |
| 0,80 — cấu hình chính | 0,763 | 0,711 | 0,820 | 0,900 | 91 | 26 | 2 |
| 0,90 | 0,763 | 0,710 | 0,820 | 0,899 | 91 | 27 | 2 |

Ngưỡng 0,70 và 0,80 có cùng các số được in; chưa thể khẳng định từng bbox/ID giống hệt chỉ từ bảng làm tròn. Tại 0,90, FN tăng một bbox và IDF1 giảm 0,001, trong khi IDSW vẫn bằng 2. Trên clip này, tăng ngưỡng không đem lại cải thiện identity rõ rệt. Giữ 0,80 cho báo cáo chính; không dùng khảo sát theo gold để chọn nhãn hoặc tuyên bố một ngưỡng tối ưu cho mọi clip. Output stretch chưa chỉ ra frame tạo ra FN tăng thêm nên không gán nguyên nhân cho một xe cụ thể.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA nhãn tay là 0,9127, thấp hơn IDF1 0,9577 khoảng 0,0450. Với 43 FP, 7 FN, 0 IDSW và 573 bbox gold: `MOTA = 1 − (43 + 7 + 0) / 573 ≈ 0,9127`. Phần giảm MOTA ở bài này đến từ FP/FN; không xuất hiện tình huống MOTA cao nhưng IDF1 thấp.

MOTA tính số sự kiện đổi ID, còn IDF1 đánh giá tính nhất quán identity trên toàn chuỗi. Một lần đổi ID có thể khiến nhiều frame mang identity không khớp nhưng chỉ cộng một sự kiện vào IDSW. Vì vậy MOTA cao đi cùng IDF1 thấp là dấu hiệu cần xem lại việc giữ ID, dù không thể chỉ từ hai số đó khẳng định mọi lỗi đều thuộc association.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

| Metric | ByteTrack | BoT-SORT + ReID | Treatment − control |
| --- | ---: | ---: | ---: |
| IDF1 | 0,875 | 0,900 | +0,025 |
| AssA | 0,776 | 0,820 | +0,044 |
| IDSW | 2 | 2 | 0 |
| HOTA | 0,709 | 0,763 | +0,054 |

Treatment có identity tốt hơn theo IDF1 và AssA, nhưng không giảm tổng số sự kiện đổi ID. Hai lỗi ByteTrack nằm tại frame 59 và 94; hai lỗi treatment nằm tại frame 87 và 113. Vì vậy “IDSW không đổi” không có nghĩa là hai hệ thống mắc cùng lỗi.

**Đoạn treatment tốt hơn — MOT frame 58–60, gold track 4 (CVAT 57–59):** chẩn đoán ByteTrack ghi đổi ID 14 → 15 tại frame 59 và gold track 4 bị chia thành `[15, 14]`. File MOT treatment đọc trực tiếp trên Colab giữ ID 9 ở cả ba frame 58, 59, 60; bbox lần lượt là `(921,45; 226,89; 38,23; 62,58)`, `(916,11; 226,19; 43,60; 66,44)`, `(910,28; 225,90; 49,38; 69,88)` theo dạng `(x; y; w; h)`. Xe ở rìa phải được nối thành một track liên tục; chẩn đoán treatment không liệt kê gold track 4 là track bị tách.

**Đoạn treatment vẫn lỗi — MOT frame 85–88, gold track 5 (CVAT 84–87):** treatment có ID 17 tại frame 85, không xuất ID 17/18 tại frame 86, rồi dùng ID 18 tại frame 87–88. Evaluator xác nhận IDSW tại frame 87. ByteTrack cũng gặp vấn đề với gold track 5 nhưng sự kiện được ghi ở frame 94. Đây là bằng chứng treatment chưa giải quyết dứt điểm việc nối identity khi mất dấu.

Chênh lệch trên là hiệu quả của cả hệ thống BoT-SORT + ReID so với ByteTrack. Hai tracker khác implementation, association và cấu hình; không được quy toàn bộ phần tăng IDF1/AssA cho riêng ReID. Để cô lập tác động của ReID cần thêm đối chứng cùng BoT-SORT và cùng các tham số, chỉ thay `with_reid`.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Treatment tăng DetA từ 0,649 lên 0,711 (+0,062), giảm FN từ 54 xuống 26 (−28), nhưng tăng FP từ 88 lên 91 (+3). Số bbox xuất ra tăng từ 607 lên 638 (+31), còn số track vẫn là 16. MOTA tăng 0,749 → 0,792 chủ yếu nhờ giảm FN; IDSW giữ nguyên. MOTP tăng 0,823 → 0,860, cho thấy các cặp bbox được ghép có độ khít tốt hơn theo phép đánh giá này.

Lỗi còn lại có cả coverage/geometry lẫn association. Treatment vẫn có 91 FP, 26 FN và 9 finding bbox lệch; gold track 6 chỉ được phủ 44/56 frame. Đồng thời gold track 5 bị chia cho ID `[18, 17]`, track 6 cho `[31, 24]`, track 7 cho `[29, 39]`; có hai IDSW tại frame 87 và 113. DetA 0,711 vẫn thấp hơn AssA 0,820 khoảng 0,109.

Không thể gọi mọi FN là lỗi detector: tracker có thể loại hoặc không khởi tạo một detection. Hai run dùng cùng detector input nhưng tập bbox cuối khác nhau do tracking. Muốn phân chia chính xác cần đối chiếu detection thô với track được xuất trên từng frame. ReID giúp nối track dựa thêm vào appearance, nhưng không bảo đảm loại mọi FP hay tạo lại một detection đã thiếu.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**MOT frame 85–88, nhãn tay ID 5 / gold ID 5 / treatment ID 17 → 18.** Nhãn tay vs gold có 0 IDSW và không có finding phân mảnh track 5. Treatment chỉ xuất ID 17 tại frame 85, mất dấu ở frame 86 và dùng ID 18 từ frame 87. Cả phép so treatment vs gold lẫn treatment vs nhãn tay đều ghi IDSW tại frame 87 trên track 5. Theo hai phép đối chiếu này, nhãn tay giữ identity nhất quán hơn treatment ở đoạn đó. Đổi ID sau khoảng trống không biến chiếc xe thành một đối tượng mới; không nên tách nhãn tay theo ID model.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Ranh giới cuối track — MOT frame 150–151 (CVAT 149–150), nhãn tay ID 4 / treatment ID 9 / gold ID 4.** Reference kết thúc track 4 ở frame 148. Treatment ID 9 còn ở frame 149 nhưng không còn ở frame 150–151; nhãn tay vẫn có bbox tại hai frame này, kích thước lần lượt khoảng `47,47 × 145,32` và `17,69 × 130,67` pixel, sát mép trái. Ở frame 150–151, quyết định không xuất track này của treatment phù hợp reference hơn nhãn tay, nên tôi cần xem lại endpoint/Outside ở đoạn này.

Tuy nhiên ảnh frame 151 vẫn còn một dải xe rất hẹp ở mép trái. Tôi cần đối chiếu quy định về phần xe còn nhìn thấy và ngưỡng kết thúc track với Lab Coach. Không sửa annotation chỉ để bắt chước model hoặc tăng điểm. Treatment cũng kéo track đến frame 149, muộn hơn reference một frame, nên không phải đáp án đúng tuyệt đối cho toàn đoạn.

**Một ca cần xem lại geometry — MOT frame 190 (CVAT 189), nhãn tay ID 3 / treatment ID 3 / gold ID 1:** bbox nhãn tay rộng 53,30 pixel, treatment rộng 61,45 pixel, reference rộng 93,92 pixel; IoU theo output lần lượt khoảng 0,50 và 0,59. Treatment gần reference hơn nhưng vẫn bị evaluator liệt kê bbox lệch. Ảnh cho thấy xe bị xe tải phía trước che một phần: phải thống nhất rule bao phần nhìn thấy hay toàn thân xe, không tự mở rộng bbox theo model.

Các frame bất đồng được notebook mới xếp cao nhất:

| MOT frame | Chỉ treatment có | Chỉ nhãn tay có | Cặp khác ID |
| --- | ---: | ---: | ---: |
| 105 | 2 | 2 | 0 |
| 106 | 2 | 2 | 0 |
| 91 | 1 | 2 | 0 |
| 107 | 2 | 1 | 0 |
| 108 | 2 | 1 | 0 |
| 109 | 2 | 1 | 0 |
| 118 | 2 | 1 | 0 |
| 52 | 1 | 1 | 0 |

Trong tám frame được chọn, có tổng 14 bbox chỉ treatment có, 11 bbox chỉ nhãn tay có và 0 cặp khác ID theo cách đếm của notebook. Không suy rộng thống kê này cho cả clip: lỗi ID đã được tìm ở các frame khác. Ví dụ treatment ID 27 trong frame 106–121 không ghép được reference, nên tôi cần kiểm tra hình ảnh trước khi kết luận mình bỏ sót xe.

Câu hỏi thứ 5 trong phần cuối notebook — sửa gì nếu gán thêm 10 clip — được trả lời ở mục 6 dưới đây, theo bố cục của mẫu báo cáo.

## 6. Nếu phải gán thêm 10 clip nữa

Nếu phải gán thêm 10 clip, tôi sẽ bổ sung vào `GUIDELINE_MINI.md` ba quy tắc gắn với các tình huống khó đã gặp và tổ chức lại bước tự kiểm:

- **Phân biệt che một phần và mất dấu hoàn toàn:** với che một phần, bbox ôm phần nhìn thấy; với mất dấu tạm thời, ghi frame cuối trước che, frame đầu xuất hiện lại và căn cứ nối ID. Ghi rõ ngưỡng 25 frame của lab và cách xử lý trường hợp vượt ngưỡng hoặc không chắc cùng xe.
- **Ghi rule bắt đầu/kết thúc track cụ thể hơn:** lưu frame đầu nhận diện được xe và frame đầu xe không còn trong khung; xem lại các đoạn như track 6 tại frame 82–100 và track 4 tại frame 149–151.
- **Tập trung QC theo từng loại lỗi:** rà identity riêng, endpoint riêng và bbox ở giữa keyframe riêng. Với IoU thấp ở cuối clip hoặc quanh đoạn biến đổi hình dạng, kiểm thêm frame lân cận thay vì chỉ nhìn hai đầu đoạn nội suy.
- **Ghi nhật ký ngay khi làm:** thời lượng từng clip, ít nhất ba ca mơ hồ có frame/ID, quyết định và lý do. Bản MOT và metric không thể khôi phục đầy đủ các thông tin này sau khi hoàn tất.
- **Giữ bằng chứng trước/sau:** khóa pre-gold trước reference/model, lưu các lần đánh giá, ghi lại frame/ID đã sửa và so sánh metric trước/sau rework.
- **Quản lý đúng notebook và cấu hình:** dùng notebook phù hợp phiên bản đề, lưu actual versions và tracker YAML; phân biệt BoT-SORT mặc định với BoT-SORT bật ReID.

Điều tôi muốn cải thiện là phân biệt rõ mất dấu tạm thời với rời hẳn khung hình, đồng thời kiểm tra bbox ở các đoạn che khuất thay vì chỉ nhìn ID. Ba tình huống này cần có ví dụ bằng frame/ID trong guideline để người gán tiếp theo áp dụng nhất quán.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
