# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Vũ Tuấn Minh`
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

Bổ sung của nhóm (nếu có): **Không bổ sung, chỉ sử dụng class `vehicle` theo guideline.**

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** | Đảm bảo cùng một xe được duy trì cùng ID khi bị che khuất trong thời gian ngắn |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới khi xe xuất hiện lại và không đủ bằng chứng để nối với track cũ | Tránh gán nhầm ID cho một đối tượng khác |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Xe có thể đã thay đổi trạng thái hoặc không thể xác nhận chắc chắn đó là cùng một track |
| Hai xe cắt nhau / chồng lên nhau | giữ ID dựa trên vị trí, hướng di chuyển và quỹ đạo trước khi giao nhau | Giảm nguy cơ đổi ID giữa hai xe khi chúng đi qua nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **không dùng ngưỡng kích thước cố định, chỉ gán khi đủ bằng chứng nhận diện là vehicle** |
| Xe đang đỗ, không di chuyển | **vẫn gán bbox và duy trì track nếu xe vẫn xuất hiện rõ trong clip** |
| Keyframe đặt dày ở đâu | **đặt dày hơn tại các đoạn xe tăng/giảm tốc, đổi hướng, bị che khuất hoặc có chuyển động phức tạp; thưa hơn khi xe chuyển động ổn định** |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

> Lưu ý: phần này phải điền theo **frame và ID thực tế trong clip**, không nên tự tạo số.

### Ca 1
- Clip / frame / ID: `Điền frame thực tế / ID thực tế`
- Tình huống: `Một vehicle bị che khuất một phần bởi vehicle khác`
- Quyết định: `Giữ nguyên ID`
- Lý do: `Đối tượng xuất hiện lại trong thời gian dưới 25 frame và có thể xác định đó là cùng một vehicle.`

### Ca 2
- Clip / frame / ID: `Điền frame thực tế / ID thực tế`
- Tình huống: `Hai vehicle đi gần nhau và bbox bị chồng lấn`
- Quyết định: `Giữ ID của từng vehicle`
- Lý do: `Dựa vào vị trí, hướng di chuyển và quỹ đạo của từng vehicle trước khi chúng giao nhau.`

### Ca 3
- Clip / frame / ID: `Điền frame thực tế / ID thực tế`
- Tình huống: `Một vehicle rời khỏi khung hình và xuất hiện lại`
- Quyết định: `Tạo track mới`
- Lý do: `Theo luật nhóm, khi vehicle rời khỏi khung hình rồi quay lại thì mặc định tạo ID mới để tránh nối sai track.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy định rõ thời gian che khuất tối đa là 25 frame; nếu vượt quá 25 frame thì không tự động nối với ID cũ nếu không có đủ bằng chứng.**
- **Khi hai vehicle chồng lấn, cần ưu tiên tính liên tục của vị trí và quỹ đạo để giữ ID, không đổi ID chỉ vì bbox bị giao nhau.**