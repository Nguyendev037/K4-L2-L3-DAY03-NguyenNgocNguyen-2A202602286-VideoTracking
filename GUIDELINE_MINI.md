# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Ngọc Nguyên`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                  | Vì sao                                                                                                                   |
| -------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Xe bị che một phần rồi hiện lại  | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)    | `Quảng thời gian che lại khuất ngắn, vẫn có thể suy đoán tiếp được quỷ đạo của vật thể`                                  |
| Xe bị che lâu hơn ngưỡng trên    | `Ngắt track khi xe xuất hiện thì bật lại track cùng Id theo dỏi tiếp`                          | `Khi vật bi che quá lâu, nếu vẫn giữ track sẽ tăng bất ổn và gây tình trạng gắn nhầm lên xe khác, dễ bị lỗi nhầm ID`     |
| Xe rời khung hình rồi quay lại   | mặc định: Cấp trách mới                                                                        | `Mỗi lần xe khỏi khung hình không có căn cứ nào biết xe mới hay củ, theo chuẩn thuật toán MOT  sẽ phải cấp phát ID mới ` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ nguyên ID độc lập của từng xe ,và theo sát chuyển động của từng xe để tránh gán nhầm lẫn` | `Điểm cắt nhau là nơi dễ phát sinh lỗi hoán đổi ID vật nhất `                                                            |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                                                                         |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                         |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                                                                                       |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `diện tích nhìn thấy ? 20% thân xe và kích thước tối thiểu 15px*15px` |
| Xe đang đỗ, không di chuyển            | `Vẫn gán nhãn vào vật thể chỉ gán ở đầu và cuối không tạo thêm frame thừa ở giữa `                                                                    |
| Keyframe đặt dày ở đâu                 | `Đặt dày tại các đoạn xe thay đổi góc, từ vị trí xa vào vị trí gần,tốc độ xe thay đổi đột ngột`                                                       |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: ` clip_01 Frame 89 – 100 id:6`
- Tình huống: `Xe ID 6 di chuyển dần về phía rìa khung hình và chỉ còn lộ một phần mép đuôi xe mờ ở góc ảnh trước khi khuất hẳn.`
- Quyết định: `bbox vẫn được kéo dài và duy trì nội suy đến tận frame 100. Sau khi đối chiếu, quyết định cắt track và bấm Outside ngay từ frame 88 (thời điểm thân xe bị khuất quá 80% diện tích). `
- Lý do: `Giữ lại bbox vì xe đã gần như ra khỏi khung hình nhưng vẫn còn dấu hiếu nhận dạng `

### Ca 2

- Clip / frame / ID: `clip_01 Frame 82-95 id:5`
- Tình huống: `Xe id 5 đổi hướng di chuyển và  zay đổi góc nhìn từ xa thành gần, khiến box bị trôi`
- Quyết định: `Thêm box vào 3 frame để nắn lại viền bounding box`
- Lý do: `Nếu để Cvat tự nội suy thì sẽ dẫn đến tình trạng trôi box và box không khớp vào vật thể`

### Ca 3

- Clip / frame / ID: `clip_01 Frame 74-78  id:5`
- Tình huống: `Xe vừa xuất hiện phần đâu nhưng không rỏ, kích thước nhỏ ảnh hưởng bởi motion blur.`
- Quyết định: `Gắn nhãn ngay ở fram 74`
- Lý do: `Nếu không đánh nhãn ở frame 74 mô hình có thể bị thiếu vật thể`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Chỉ bắt đầu tạo track khi diện tích nhìn thấy của xe đạt tối thiểu >20% hoặc kích thước 15px*15px`
- `Khi xe di chuyển ra khỏi rìa, mép bbox phải bám sát mép ảnh; ngay tại frame đầu tiên xe bị khuất quá 80%, chuyển sang Outside`
