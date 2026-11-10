# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con phục vụ hoặc sedan, SV, taxi, xe bán tải | Phân loại theo kiểu phương tiện, không dự vào màu sắc hây mục đích sử dụng xe. Không nhầm với xe tải lớn |
| `truck` | Xe tải hoặc xe đầu kéo có chức năng chính phục vụ vận chuyển hàng hoá | Không nhầm với xe bán tải, xe có kết cấu khác xe tải thông thường thì đối chiếu schema hoặc thắc mắc labcoach |
| `bus` | Xe buýt, minibus thiết kế để vận chuyển khách | VD: Bus công cộng, minibus chở khách. Phân biệt với xe van dựa trên kiểu thân xe, cửa, dấu hiệu nhận diện xe |
| `van` | Xe van or minivan có thân xe dạng thùng/khoang kín, phục vụ vận chuyển người or hàng hoá | Có thể nhầm với xe buýt nhỏ hoặc xe con |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: mỗi xe hợp lệ được gán bounding box riêng, ôm sát phần xe nhìn thấy, không lấy quá nhiều nền/mặt đường/phương tiên khác/bóng đổ. 
- Vật thể bị che khuất: xe bị vật khác che khuất một phần nhưng vẫn nhận diện được lớp, tiếp tục gán nhãn cho phần nhìn thấy; không tự đoán kích thước phần bị che
- Vật thể bị cắt bởi rìa ảnh: bounding box kết thúc tại rìa ảnh, không tự tưởng tưởng phần ngoài khung hình; không mở rộng ra ngoài khung hình
- Vật thể rất nhỏ/mơ hồ: chỉ gán khi đủ dấu hiệu xác định đối tượng thuộc 1 trong các lớp. Ghi lại case mơ hồ để hỏi labcoach, không tự đoán
- Tính nhất quán: xe đang đỗ or di chuyển đu0ợc phân loại theo cùng 1 schema. Không gán người đi bộ, các phương tiện, hình xe trên quảng cáo hoặc hình phản chiếu 

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_031.jpg` | Nhiều xe ở xa và bị che 1 phần trên đường | Gán riêng từng xe nếu vẫn nhận diện được loại xe; không vẽ box bao cả cụm xe | Đảm bảo ngyên tắc 1 đối tượng 1 box. Các xe quá nhỏ, không xác định đươc lớp cần note lại để hỏi labcoach |
| `drive_014.jpg` | Xe cứu hộ màu trắng ở dưới ảnh có kết cấu chuyên dụng, khó xếp vào car, truck hay van | Tạm đánh dấu là trường hợp chưa xác định; hỏi labcoach trước khi gán | Schema chỉ có 4 loại xe và chưa quy định rõ xe cứu hộ. Không tự create class hay gán theo suy đoán |
| `drive_045.jpg` | Hàng xe đỗ sát nhau ở bên trái ảnh, 1 số xe bị che bởi xe khác hoặc công trình dọc đường | Gán từng xe có thể nhận diện được, chỉ bao phần thân nhìn thấy của từng xe| Xe đang đỗ vẫn thuộc phạm vi annotation. Không gộp nhiều xe thành 1 box hoặc ước tính phần bị che |
