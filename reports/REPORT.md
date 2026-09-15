# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Ngọc Nguyên`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị            |
| --------------------------------- | ------------------ |
| Công cụ                           | CVAT / khác: `...` |
| Thời gian gán `clip_02` (warm-up) | `20` phút          |
| Thời gian gán `clip_01`           | `30` phút          |
| Số track đã vẽ trong `clip_01`    | `8`                |
| Số keyframe trung bình mỗi track  | `4`                |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe đi dần ra khỏi khung hình, hướng xử lý chỉ gán xe khi thấy xe từ 10% trở lên, khi xe vừa khuất dân điểu chỉnh lại track và lập tức outside khi không nhìn thấy`
2. `Xe bị che khuất, đi sau xe khác nhưng vẫn lòi một phần ra, hướng xử lý giữ box bao quanh phần nhìn thấy được của vật thể, điều chỉnh box theo xa liên tục và cập nhập khi xe thấy rỏ và rời xa  `
3. `Nhiều vật thể gây loạn, quá trình gán nhãn. Hướng xử lỷ theo dỏi từng xe kết thúc cho đến khi rời khung hình, kiên nhẫn từng vật thể một`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị     |
| ---------------------------------------------------- | ----------- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...`       |
| Thời điểm khóa                                       | `11h30`     |
| Số row / frame / track trước khi mở reference        | `598 190 9` |

|              |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------ | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| Bản pre-gold | 0.823 | 0.808 | 0.841 | 0.874 | 0.970 | 0.939 | 0.862 |  30 |   5 |    0 |
| Sau rework   |       |       |       |       |       |       |       |     |     |      |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi                 | Frame  | ID  | Đã sửa thế nào                                             |
| ------------------------ | ------ | --- | ---------------------------------------------------------- |
| Box xuất hiện sớm        | 89,100 | 6   | Cắt bỏ 2-3 frame trước khi xe xuất hiện, dời frame trễ lại |
| Box bị to so với vật thể | 82,95  | 5   | Điều chỉnh lại box, bám sát vào vật thể                    |
| Box bị thừa              | 1      | -1  | Bỏ frame                                                   |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                        |
| ---------------------------------- | ---------------------------------------------- |
| Python / ultralytics / torch / lap | `3.13.15`                                      |
| weights / hai tracker              | `yolo26n.pt, bytetrack.yaml,botsort-reid.yaml` |
| conf / IoU / imgsz / classes       | `0.25, 0.70, 960, [2,5,7]`                     |
| device                             | `GPU`                                          |

| So sánh           |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ----------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| ban_vs_gold       | 0.823 | 0.808 | 0.841 | 0.874 | 0.970 | 0.939 | 0.862 |  30 |   5 |    0 |
| bytetrack_vs_gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 |  88 |  54 |    2 |
| reid_vs_gold      | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 |  91 |  26 |    2 |
| reid_vs_ban       | 0.746 | 0.689 | 0.810 | 0.871 | 0.892 | 0.781 | 0.854 |  84 |  44 |    3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA thấp hơn 0.939 < 0.97`, Nếu Mota cao mà IDF1 thấp nghĩa là người gán nhãn đã theo dỏi vật thể rất tốt bắt đúng vị trí qua từng frame, nhưng lại duy trì identity kém, xe mất dạng, track bị thay đổi nhầm,...

Bởi vì trong công thức IDSW (lỗi ID) chỉ tính trọng số phạt là 1 lần duy nhất tại frame có sự thay đổi. Ngay sau đó, dù đối tượng mang ID sai suốt 100 frame, công thức vấn tính là đúng, còn ID tính trên toàn khung ảnh của vật thể nên bổng đột nhiên mà bị gãy hay lỗi thì điểm ID sẽ bi trừ nặng

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ByteTrack thuần túy dựa vào vị trí dự đoán (Kalman Filter) và độ chồng lấp hình học (IoU) qua cơ chế ghép nối 2 vòng (vòng 1 khớp detection điểm cao, vòng 2 tận dụng detection điểm thấp bị che khuất).
BoT-SORT kết hợp đồng thời 3 nguồn tín hiệu (cues): bù trừ chuyển động camera (Camera Motion Compensation - CMC), vị trí hình học Kalman Filter/IoU, và vector đặc trưng ngoại hình (ReID embedding).

Dẫn chứng frame sequence:Ở track gold 4 (kéo dài 95 frame): ByteTrack bị nhảy ID tại frame 59 (từ ID 14 sang 15), làm track bị đứt gãy chia làm 2 nửa. Trong khi đó, BoT-SORT + ReID giữ nguyên vẹn ID cho track 4 suốt 95 frame mà không hề bị đứt đoạn. Ở track gold 6 (56 frame): ByteTrack bị mất dấu hẳn 14 frame (chỉ phủ 42/56 frame, tức 75%). BoT-SORT + ReID nhờ có thêm đặc trưng ngoại hình (appearance embeddings) nên dù xe bị che khuất nhẹ vẫn tái định danh tốt hơn (phủ 44/56 frame, tức 79%), dẫu vậy vẫn bị switch ID ở frame 113 (ID 24 sang 31).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Sự thay đổi chỉ số:DetA: Tăng từ 0.649 (ByteTrack) lên 0.711 (BoT-SORT + ReID). FP: Tăng nhẹ từ 88 lên 91 (tăng 3 FP). FN: Giảm mạnh từ 54 xuống còn 26 (giảm hơn một nửa số bbox bị bỏ sót).

Lỗi lớn nhất còn lại nằm ở Detector, thể hiện qua việc số lượng False Positive (FP = 91) vẫn rất cao so với tổng số 573 bbox của gold. Cả hai mô hình đều bị detector khoanh nhầm các đối tượng tĩnh ở hậu cảnh

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**
Frame 16 – 116, Track model ID 7
Tại chuỗi frame này, model ReID sinh ra một track ảo (ID 7) tồn tại cố định suốt 43 frame không hề chuyển động ở rìa đường/lề phố (vật thể tĩnh như biển quảng cáo/vật thể dạng khối bị detector nhận nhầm là ô tô). Bản nhãn tay của bạn đã nhìn nhận đúng ngữ cảnh thực tế của video và không gán bbox cho chướng ngại vật tĩnh này, hoàn toàn khớp với Ground Truth (gold cũng không có track này)

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Vị trí cụ thể: Frame 105 – 107, Track bản A ID 6 (tương ứng vùng xung quanh frame bất đồng cao 106 theo log).
Lý do: Tại frame 106, bảng bất đồng cho thấy nhãn của bạn và model lệch nhau đáng kể (chỉ model có: 2, chỉ bạn có: 2). Khi xem lại, đây là thời điểm xe ID 6 chuẩn bị rời khỏi hoặc đi lướt qua một vùng giao cắt; nhãn của bạn bị gán trôi/kéo dài thêm trong khi thân xe thực tế đã khuất dần ra khỏi góc quan sát (dẫn đến việc bạn bị dư 12 frame từ 89–100 như log eval_vs_gold cảnh báo). Model tuy có bị nhảy ID (ID 24 sang 28 sang 31) nhưng việc nó thay đổi bounding box đã chỉ ra chính xác frame mà đối tượng bị biến dạng hình học/mất dấu thị giác, nhắc nhở người gán cần cắt keyframe (bấm Outside) sớm hơn thay vì để nội suy chạy tự do.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Chỉ bắt đầu vẽ track khi xe xuất hiện trong khung hình và diện tích nhận biết đạt tối thiểu > 20% thân xe (hoặc chiều rộng/dài >= 15 pixel).
Khi xe di chuyển ra ngoài rìa, mép bbox chạm sát mép ảnh (không vẽ bbox vượt ra ngoài vùng nhìn thấy). Ngay tại frame đầu tiên mà xe khuất khỏi tầm nhìn quá 80%, bắt buộc tạo keyframe và chuyển trạng thái sang Outside, tuyệt đối không để chế độ nội suy tự kéo dài qua các frame kế tiếp.

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
