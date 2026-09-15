# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Nguyễn Văn Tiến — 2A202602056 (cá nhân)`

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

Bổ sung: chỉ tạo bbox khi đủ dấu hiệu nhận diện là xe bốn bánh; không gán một vật thể chỉ vì có chuyển động hoặc có hình dạng giống xe ở khoảng cách xa.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật áp dụng | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu thời gian che dưới 25 frame và vị trí, hướng di chuyển, đặc điểm nhìn thấy được vẫn khớp. | Identity là của cùng một xe, không phải của từng bbox rời rạc. |
| Xe bị che từ 25 frame trở lên | Chỉ giữ ID khi có bằng chứng liên tục, đáng tin cậy; nếu không đủ bằng chứng, bắt đầu ID mới khi xe xuất hiện lại. | Tránh nối nhầm hai xe giống nhau sau một khoảng mất dấu dài. |
| Xe rời khung hình rồi quay lại | Kết thúc track ở frame cuối còn nhìn thấy; xe quay lại sau khi đã rời khung dùng track ID mới. | Không có bằng chứng quan sát liên tục ngoài khung hình. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo quỹ đạo trước/sau crossing; xem các frame liền kề trước khi thay ID. | Tránh swap ID do chỉ nhìn một frame chồng lấp. |

## 3. Luật bbox

| Tình huống | Luật áp dụng |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, không đoán phần nằm ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox chỉ ôm phần **nhìn thấy được**, không bao phủ vùng bị che. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track tại frame đầu tiên nhận diện được chắc chắn là xe bốn bánh; nếu chưa đủ chắc chắn thì chờ frame kế tiếp. |
| Xe đang đỗ, không di chuyển | Vẫn gán nếu xe bốn bánh còn nhìn thấy; kiểm tra để bbox không bị treo khi xe thực sự đã rời khung. |
| Keyframe đặt dày ở đâu | Đặt keyframe tại lúc xe vào/ra khung, đổi hướng/tốc độ, bị che/hiện lại, hoặc khi interpolation làm bbox không còn ôm sát xe. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Các ca dưới đây được ghi từ diagnostics của `outputs/eval_vs_gold.json`; trạng thái là việc cần rework, không phải xác nhận đã sửa.

### Ca 1

- Clip / frame / ID: `clip_01`, frame `97–100`, ID `6`.
- Tình huống: bbox của ID 6 xuất hiện trước thời điểm track tham chiếu xuất hiện; cần kiểm tra chính xác frame đầu tiên xe đủ rõ để bắt đầu track.
- Quyết định: kiểm tra các frame liền kề để xác định frame đầu tiên xe được nhận diện chắc chắn; chỉ bỏ bbox nếu xác nhận bbox thực sự xuất hiện trước thời điểm xe đủ rõ để track.
- Lý do: không được giữ bbox khi xe chưa xuất hiện/không đủ bằng chứng; diagnostics đánh dấu 4 bbox ghost ở các frame này.

### Ca 2

- Clip / frame / ID: `clip_01`, frame `149–151`, ID `4`.
- Tình huống: bbox còn tồn tại sau khi xe đã rời khung.
- Quyết định: kết thúc track ở frame cuối còn phần xe nhìn thấy; bật `outside` ở frame tiếp theo.
- Lý do: không ngoại suy xe ra ngoài ảnh; diagnostics đánh dấu 3 bbox ghost ở các frame này.

### Ca 3

- Clip / frame / ID: `clip_01`, frame `168`, ID `8`.
- Tình huống: bbox còn khớp xe nhưng lỏng, IoU với reference là `0,598`.
- Quyết định: vẽ lại bbox ôm sát phần xe nhìn thấy ở frame này và đặt thêm keyframe nếu cần.
- Lý do: bbox phải phản ánh hình học phần nhìn thấy tại đúng frame; không dùng box nội suy nếu nó quá rộng hoặc lệch xe.

## 5. Quy tắc cần làm rõ sau khi chấm với gold và kiểm chéo

- Quy định rõ `outside`: kết thúc bbox ở frame cuối xe còn nhìn thấy, không để bbox tồn tại ở frame xe đã rời khung; cần kiểm lại thêm ID 7 tại frame `103–105` và ID 8 tại `133–135`, `169–171` theo diagnostics.
- Khi xe vào khung hoặc còn nhỏ/mờ, chỉ bắt đầu track khi nhận diện được xe bốn bánh; dùng frame liền kề để xác nhận thay vì tạo bbox sớm.
- Sau khi sửa trong CVAT, phải export lại, chạy evaluation lại và cập nhật report; không sửa trực tiếp MOT file.
