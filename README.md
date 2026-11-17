# BÁO CÁO THỰC HÀNH TỔNG HỢP
## Kỹ thuật Prompting Nâng cao (Logic & Chuỗi)

## 1. Bối cảnh bài toán

**Bài toán:** Xây dựng quy trình AI tự động phân tích phản hồi khách hàng cho một cửa hàng bán hàng online.

**Mục tiêu:**
- Phân loại cảm xúc khách hàng.
- Xác định vấn đề chính trong phản hồi.
- Gắn thẻ nội dung.
- Đề xuất hướng xử lý.
- Tổng hợp kết quả thành báo cáo.

Quy trình sử dụng Prompt Chaining để chia nhiệm vụ lớn thành các bước nhỏ, đồng thời sử dụng kiểm tra logic để hạn chế sai sót.

## 2. Thiết kế giải pháp

### 2.1. Sơ đồ quy trình

```text
Dữ liệu phản hồi
       |
       v
Bước 1: Làm sạch dữ liệu
       |
       v
Bước 2: Phân tích cảm xúc
       |
       v
Bước 3: Kiểm tra và xử lý lỗi
       |
       v
Bước 4: Tổng hợp báo cáo
       |
       v
Kết quả hoàn chỉnh
```

### 2.2. Cơ chế xử lý lỗi

- Dữ liệu bị thiếu: đánh dấu là chưa xác định.
- Dữ liệu sai định dạng: bỏ qua hoặc đưa vào danh sách lỗi.
- Nội dung trùng lặp: chỉ giữ một bản ghi.
- AI trả về kết quả không hợp lệ: yêu cầu tạo lại kết quả.
- Quá trình bị gián đoạn: lưu kết quả bước gần nhất để tiếp tục.

## 3. Prompt thực thi các bước

### Prompt hệ thống

```text
Bạn là AI chuyên phân tích phản hồi khách hàng.
Hãy xử lý dữ liệu theo từng giai đoạn được yêu cầu.
Chỉ sử dụng thông tin có trong dữ liệu đầu vào.
Không tự tạo thông tin khi dữ liệu bị thiếu.
Luôn kiểm tra định dạng và tính hợp lệ của kết quả.
Nếu gặp dữ liệu lỗi, hãy đánh dấu thay vì làm dừng toàn bộ quy trình.
Trả kết quả đúng định dạng JSON khi được yêu cầu.
```

### Prompt 1: Làm sạch dữ liệu

```text
Hãy kiểm tra và làm sạch danh sách phản hồi khách hàng.

Yêu cầu:
- Loại bỏ khoảng trắng thừa.
- Loại bỏ phản hồi trùng lặp.
- Đánh dấu phản hồi trống hoặc không hợp lệ.
- Không tự sửa nội dung làm thay đổi ý nghĩa.

Đầu ra: Danh sách dữ liệu sạch và danh sách lỗi.
```

### Prompt 2: Phân tích cảm xúc

```text
Dựa trên dữ liệu sạch từ bước 1, hãy phân loại từng phản hồi:

- Positive: Tích cực.
- Negative: Tiêu cực.
- Neutral: Trung lập.

Đồng thời xác định vấn đề chính và gắn thẻ:
- Sản phẩm.
- Giao hàng.
- Giá cả.
- Dịch vụ.

Nếu không đủ thông tin, ghi "Chưa xác định".
Trả kết quả dưới dạng JSON.
```

### Prompt 3: Kiểm tra kết quả

```text
Hãy kiểm tra kết quả từ bước 2.

Yêu cầu:
- Kiểm tra cảm xúc có phù hợp nội dung không.
- Kiểm tra thẻ có đúng vấn đề không.
- Kiểm tra dữ liệu có bị thiếu hoặc sai định dạng không.
- Không tự suy đoán thông tin không có trong phản hồi.

Nếu phát hiện lỗi, hãy sửa và ghi rõ lỗi đã xử lý.
Nếu dữ liệu không đủ căn cứ, giữ trạng thái "Chưa xác định".
```

### Prompt 4: Tổng hợp báo cáo

```text
Dựa trên kết quả đã kiểm tra, hãy tạo báo cáo gồm:

1. Tổng số phản hồi hợp lệ.
2. Số phản hồi tích cực, tiêu cực và trung lập.
3. Các vấn đề khách hàng thường gặp.
4. Đề xuất hướng cải thiện.

Trình bày bằng bảng Markdown.
Chỉ sử dụng dữ liệu đã được kiểm tra.
```

## 4. Kết quả chạy thử

### 4.1. Dữ liệu đầu vào

| ID | Phản hồi khách hàng |
|---|---|
| 01 | Sản phẩm đẹp, giao hàng nhanh. |
| 02 | Đóng gói kém, sản phẩm bị móp. |
| 03 | Giá hơi cao nhưng chất lượng tốt. |
| 04 | Giao hàng quá chậm, rất thất vọng. |
| 05 | (Trống) |
| 06 | Sản phẩm đẹp, giao hàng nhanh. |

### 4.2. Kết quả sau bước 1

| ID | Trạng thái | Kết quả |
|---|---|---|
| 01 | Hợp lệ | Giữ lại |
| 02 | Hợp lệ | Giữ lại |
| 03 | Hợp lệ | Giữ lại |
| 04 | Hợp lệ | Giữ lại |
| 05 | Không hợp lệ | Bỏ qua vì dữ liệu trống |
| 06 | Trùng lặp | Loại bỏ |

Kết quả: Có 4 phản hồi hợp lệ và 2 bản ghi cần xử lý.

### 4.3. Kết quả sau bước 2 và 3

| ID | Cảm xúc | Thẻ | Vấn đề chính |
|---|---|---|---|
| 01 | Positive | Sản phẩm, giao hàng | Đánh giá tốt |
| 02 | Negative | Sản phẩm | Đóng gói kém, hàng bị móp |
| 03 | Neutral | Giá cả, sản phẩm | Giá cao nhưng chất lượng tốt |
| 04 | Negative | Giao hàng | Giao hàng chậm |

Bước kiểm tra xác nhận các bản ghi đều có đủ thông tin cần thiết và cảm xúc phù hợp với nội dung phản hồi.

### 4.4. Báo cáo tổng hợp cuối cùng

| Tiêu chí | Số lượng |
|---|---:|
| Tổng phản hồi hợp lệ | 4 |
| Tích cực | 1 |
| Tiêu cực | 2 |
| Trung lập | 1 |

**Các vấn đề chính:**
- Đóng gói sản phẩm chưa tốt.
- Thời gian giao hàng còn chậm.
- Một số khách hàng chưa hài lòng về giá.

**Đề xuất cải thiện:**
1. Kiểm tra và cải thiện quy trình đóng gói.
2. Theo dõi thời gian giao hàng.
3. Xem xét giá bán so với chất lượng sản phẩm.

## 5. Đánh giá giải pháp

| Tiêu chí | Kết quả |
|---|---|
| Chain-of-Thought | Chia nhiệm vụ thành các bước logic nhỏ |
| Prompt Chaining | Các prompt liên kết và truyền kết quả tuần tự |
| Xử lý đứt gãy | Có thể tiếp tục từ bước gần nhất đã lưu |
| Kiểm soát dữ liệu | Phát hiện dữ liệu trống và trùng lặp |
| Tối ưu token | Mỗi prompt chỉ thực hiện một nhiệm vụ |
| Hạn chế lỗi | Có bước kiểm tra trước khi tổng hợp |

## 6. Kết luận

Giải pháp đã kết hợp Chain-of-Thought, Prompt Chaining và cơ chế xử lý lỗi để xây dựng quy trình phân tích phản hồi khách hàng.

Việc chia nhiệm vụ thành nhiều bước giúp dễ kiểm soát, phát hiện lỗi và tiếp tục xử lý khi quy trình bị gián đoạn. Đây là cách ứng dụng Prompting nâng cao vào một bài toán thực tế.
