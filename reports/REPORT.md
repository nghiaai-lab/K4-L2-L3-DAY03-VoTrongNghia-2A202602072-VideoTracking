# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Võ Trọng Nghĩa — 2A202602072`
Ngày: `15/09/2026`

> Trạng thái: số liệu định lượng bên dưới được lấy từ validator, `eval_vs_gold.json`
> và notebook đã chạy. Các mục đánh dấu **CẦN BỔ SUNG/XÁC NHẬN** phải được người
> thực hiện tự điền hoặc kiểm tra bằng mắt trước khi nộp.

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | **CẦN BỔ SUNG** |
| Thời gian gán `clip_01` | **CẦN BỔ SUNG** |
| Số track đã vẽ trong `clip_01` | 8 track, 507 bbox trên 190 frame |
| Số keyframe trung bình mỗi track | **CẦN BỔ SUNG** — MOT 1.1 không lưu cờ keyframe nên không thể suy ra từ `gt.txt` |

Ba tình huống khó nhất khi gán clip này, và cách xử lý:

1. **CẦN XÁC NHẬN BẰNG MẮT:** đoạn frame 9–16, bbox của xe tương ứng track tham chiếu 3 có IoU khoảng 0.56–0.60; cần mô tả quyết định đặt keyframe/bbox của bạn.
2. **CẦN XÁC NHẬN BẰNG MẮT:** đoạn frame 101–106, bbox của các track 5 và 6 bị evaluator cảnh báo lệch; cần mô tả cách bạn xử lý hai xe trong đoạn này.
3. **CẦN XÁC NHẬN BẰNG MẮT:** các track tham chiếu 4, 5 và 8 chưa được phủ hết quãng đời; cần mô tả quyết định về frame bắt đầu/kết thúc hoặc `outside`.

## 2. Tự kiểm và kiểm chéo

Kết quả kỹ thuật hỗ trợ ba lượt tua:

- Lượt 1 — identity/timeline: file có 8 ID duy nhất; đánh giá với gold ghi nhận 0 ID switch.
- Lượt 2 — frame đầu/cuối: validator cảnh báo track 3 gần như đứng im ở frame 1–15; evaluator cũng cho thấy track tham chiếu 4, 5 và 8 chỉ được phủ lần lượt 73%, 63% và 67%.
- Lượt 3 — geometry/interpolation: có 17 bbox IoU thấp; cụm đáng chú ý nằm ở frame 9–16 và 101–106.

Kiểm chéo với: **CẦN BỔ SUNG TÊN REVIEWER**. Chi tiết cần được ghi trong
`reports/review_partner.md`.

Số lỗi tìm được trong bản của reviewer: **CẦN BỔ SUNG**. Số lỗi reviewer tìm
được trong bản này: **CẦN BỔ SUNG**.

Ca hai người quyết khác nhau và luật còn thiếu trong `GUIDELINE_MINI.md`:
**CẦN BỔ SUNG từ buổi peer review; không có evidence để tự suy diễn.**

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | **CHƯA CÓ MANIFEST** |
| Thời điểm khóa | **CHƯA CÓ EVIDENCE** |
| Số row / frame / track trước khi mở reference | File CVAT hiện tại: 507 row / frame 1–190 / 8 track; không thể khẳng định đây là snapshot pre-gold khi chưa có manifest |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | — | — | — | — | — | — | — | — | — | — |
| Bản hiện tại | 0.690 | 0.669 | 0.715 | 0.812 | 0.913 | 0.836 | 0.787 | 14 | 80 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

Các điểm evaluator đề nghị xem lại; chỉ ghi “đã sửa” sau khi sửa trong CVAT và
export lại:

| Loại lỗi | Frame | ID | Trạng thái / hành động cần làm |
| --- | --- | --- | --- |
| Bbox trôi | 9–16 | track dự đoán 2 / track tham chiếu 3 | Xem lại bằng mắt và thêm keyframe nếu bbox bị interpolation drift |
| Bbox trôi | 101–106 | 5, 6 | Xem lại độ khít của bbox và mật độ keyframe |
| Thiếu đoạn | quãng đời track tham chiếu 4, 5, 8 | 4, 5, 8 | Kiểm tra frame vào/ra và thao tác `outside`; không sửa trực tiếp file MOT |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ notebook đã chạy:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` / `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| device | CPU |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.690 | 0.669 | 0.715 | 0.812 | 0.913 | 0.836 | 0.787 | 14 | 80 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | **CHƯA CÓ DỮ LIỆU — cần chạy lại notebook sau khi repo đã có annotation** | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của bản hiện tại là 0.836, thấp hơn IDF1 0.913. Kết quả này phù hợp với
việc identity được giữ khá tốt (0 IDSW), trong khi độ bao phủ còn thiếu: 80 FN
và 14 FP làm giảm MOTA. Nếu một kết quả có MOTA cao nhưng IDF1 thấp, detector có
thể vẫn tìm được phần lớn vật thể nhưng tracker gắn sai hoặc đổi identity trên
quãng đời của xe. Trong MOTA, mỗi ID switch chỉ đóng góp một lỗi cùng FP và FN,
trong khi IDF1 đánh giá trực tiếp tính nhất quán identity qua các detection nên
nhạy hơn với lỗi gán ID kéo dài.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, BoT-SORT + ReID tăng IDF1 từ 0.875 lên 0.900 (+0.025) và AssA
từ 0.776 lên 0.820 (+0.044), trong khi IDSW vẫn là 2. Danh sách lỗi cho thấy
ByteTrack đổi ID ở frame 59 (gold track 4: ID 14 → 15) và frame 94 (gold track
5: ID 23 → 32). Treatment không báo hai switch này, nhưng lại có switch riêng ở
frame 87 (gold track 5: ID 17 → 18) và frame 113 (gold track 6: ID 24 → 31).
Vì vậy treatment cải thiện association tổng thể nhưng không loại bỏ hoàn toàn
lỗi identity. Đây không phải phép cô lập causal effect của ReID, vì ByteTrack và
BoT-SORT là hai tracker implementation khác nhau ngoài thành phần appearance.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 lên 0.711. FP tăng nhẹ từ 88 lên 91 (+3), nhưng FN giảm mạnh
từ 54 xuống 26 (−28), cho thấy treatment cải thiện độ bao phủ chủ yếu nhờ bỏ sót
ít hơn. AssA cũng tăng 0.044 nhưng IDSW không đổi, nên lỗi còn lại là hỗn hợp:
detector vẫn tạo nhiều false positive và tracker vẫn còn hai lần đổi ID.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Ở frame 87, evaluator của ReID ghi nhận gold track 5 bị đổi từ model ID 17 sang
18. Bản annotation cá nhân có 0 ID switch trên toàn clip khi đối chiếu với gold,
nên đây là evidence cho thấy nhãn cá nhân giữ identity tốt hơn model ở đoạn này.
**CẦN XÁC NHẬN BẰNG MẮT frame 86–88 trước khi nộp.**

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Cụm frame 101–106 làm mình cần xem lại annotation: evaluator của bản cá nhân
cảnh báo bbox trôi ở track 5 và 6, với IoU thấp nhất khoảng 0.51–0.59. Đây là dấu
hiệu nên kiểm tra lại mật độ keyframe và độ khít bbox. **CẦN XÁC NHẬN BẰNG MẮT
trước khi kết luận model hay annotation đúng.**

## 6. Nếu phải gán thêm 10 clip nữa

Từ các lỗi đã thấy, guideline nên quy định cụ thể hơn: (1) ngưỡng bắt đầu/kết
thúc track và cách dùng `outside`; (2) mật độ keyframe tối thiểu khi xe đổi hướng,
thay đổi kích thước hoặc bị che; (3) cách xử lý xe đứng yên lâu; và (4) cách giữ
ID khi hai xe chồng/cắt nhau. Quy trình nên chia self-QC thành ba lượt riêng cho
identity, endpoint/coverage và geometry, chạy validator sau từng chặng, rồi tạo
pre-gold lock trước khi mở reference hoặc chạy model. **Bạn cần đọc lại và sửa
đoạn này theo trải nghiệm thật của mình trước khi nộp.**

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt` — file hiện tại rỗng
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
