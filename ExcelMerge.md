# PROMPT BỔ SUNG: KIỂM TRA TRÙNG LẶP SAU KHI MERGE VÀ ĐÁNH DẤU DỮ LIỆU

## 1. Mục tiêu

Bổ sung chức năng kiểm tra trùng lặp sau khi hoàn tất gộp dữ liệu trong ứng dụng Excel Merger Pro.

Chức năng chỉ được kích hoạt khi người dùng chọn chế độ:

**“Kiểm tra trùng theo cột khóa”**

Sau khi quá trình merge hoàn tất, ứng dụng phải đọc lại toàn bộ dữ liệu đã được ghi vào đích đầu ra (`.db`, `.xlsx` hoặc `.xls`), phân tích các bản ghi trùng nhau dựa trên cột khóa đã chọn, bổ sung một cột ghi chú để đánh dấu trùng lặp và sắp xếp dữ liệu theo thứ tự tăng dần.

Phải bảo đảm dữ liệu được đánh dấu chính xác, không bỏ sót bản ghi trùng và không tự ý xóa dữ liệu gốc.

## 2. Điều kiện kích hoạt

- Nếu người dùng chọn “Không kiểm tra trùng lặp”, không thực hiện bước rà soát trùng sau khi merge.
- Nếu người dùng chọn “Bỏ qua dòng trùng hoàn toàn”, giữ nguyên chức năng bỏ qua dòng trùng đã được cấu hình; không tự động thay thế bằng chức năng kiểm tra trùng theo khóa.
- Nếu người dùng chọn “Kiểm tra trùng theo cột khóa”, bắt buộc thực hiện bước rà soát sau merge.
- Cột khóa phải được người dùng lựa chọn trước khi bắt đầu xử lý.
- Cho phép chọn một hoặc nhiều cột làm khóa so sánh.
- Nếu chưa chọn cột khóa hoặc cột khóa không tồn tại trong dữ liệu đầu ra, phải thông báo lỗi rõ ràng và không tiếp tục bước đánh dấu trùng.

## 3. Quy trình xử lý sau khi merge

Thực hiện theo đúng thứ tự sau:

### Bước 1: Hoàn tất quá trình merge

Ghi dữ liệu từ các file nguồn vào đích đầu ra theo cấu hình hiện hành.

Đảm bảo tất cả giao dịch ghi dữ liệu đã hoàn tất và tài nguyên ghi file đã được giải phóng trước khi thực hiện bước đọc lại.

### Bước 2: Đọc lại dữ liệu đầu ra

Đọc dữ liệu từ chính đích đã merge:

- SQLite: mở file `.db`, đọc bảng đích được cấu hình.
- Excel `.xlsx`: đọc sheet đích.
- Excel `.xls`: đọc sheet đích bằng thư viện hỗ trợ định dạng Excel 97–2003.

Không sử dụng danh sách dữ liệu nguồn còn trong bộ nhớ làm căn cứ duy nhất để đánh dấu trùng. Phải kiểm tra dữ liệu đã được ghi thực tế vào đầu ra.

Nếu không thể đọc lại dữ liệu hoặc phát hiện lỗi đầu ra, ghi log lỗi và thông báo rõ ràng. Không báo hoàn tất toàn bộ quy trình nếu bước hậu kiểm bắt buộc thất bại.

### Bước 3: Xác định các bản ghi trùng theo cột khóa

Sử dụng cột hoặc tổ hợp cột khóa do người dùng chọn.

Ví dụ, người dùng chọn cột `MaNV` làm khóa. Nếu giá trị `NV001` xuất hiện ở nhiều dòng, tất cả các dòng có `MaNV = NV001` đều phải được đánh dấu trùng.

Ví dụ, nếu chọn tổ hợp khóa `MaNV` và `DonVi`, chỉ những dòng có cùng giá trị ở cả hai cột mới được coi là trùng theo khóa.

Quy tắc bắt buộc:

- Đánh dấu tất cả bản ghi thuộc nhóm có từ hai dòng trở lên.
- Không chỉ đánh dấu các dòng xuất hiện sau dòng đầu tiên.
- Không tự ý xóa bản ghi trùng.
- Không thay đổi các giá trị dữ liệu gốc ngoài cột ghi chú được bổ sung và thứ tự sắp xếp.
- So sánh theo giá trị của cột khóa, không so sánh theo số thứ tự dòng.
- Xử lý nhất quán giá trị null, ô trống, khoảng trắng và kiểu dữ liệu.
- Cho phép cấu hình có phân biệt chữ hoa/chữ thường hay không.
- Mặc định không coi khóa rỗng là một nhóm trùng hợp lệ; những dòng có khóa rỗng được ghi chú riêng hoặc để trống theo quy tắc được cấu hình.

### Bước 4: Thêm cột ghi chú trùng lặp

Sau khi phân tích, thêm một cột mới ở cuối bảng dữ liệu với tên mặc định:

`GhiChuTrung`

Nếu tên cột đã tồn tại, không ghi đè cột cũ. Hãy tạo tên thay thế phù hợp hoặc cho phép người dùng cấu hình tên cột.

Nội dung ghi chú:

- Với dòng không trùng: để trống hoặc ghi `Không trùng`, theo cấu hình.
- Với dòng trùng: ghi `TRÙNG`.
- Nếu có nhiều nhóm trùng, có thể ghi chi tiết khóa trùng để người dùng dễ nhận biết.

Ví dụ:

| MaNV | HoTen | DonVi | GhiChuTrung |
|---|---|---|---|
| NV001 | Nguyễn Văn A | Phòng CNTT | TRÙNG |
| NV002 | Trần Thị B | Phòng Kế toán | |
| NV001 | Nguyễn Văn A | Phòng CNTT | TRÙNG |
| NV003 | Lê Văn C | Phòng HC | |
| NV004 | Phạm Thị D | Phòng CNTT | TRÙNG |
| NV004 | Phạm Thị D | Phòng HC | TRÙNG |

Trong ví dụ trên, nếu khóa chỉ là `MaNV`, cả hai dòng `NV004` đều được đánh dấu trùng dù cột `DonVi` khác nhau.

Nếu khóa là tổ hợp `MaNV + DonVi`, hai dòng `NV004` trong ví dụ không được đánh dấu trùng theo khóa vì giá trị `DonVi` khác nhau.

### Bước 5: Sắp xếp dữ liệu mặc định theo thứ tự tăng dần

Sau khi thêm cột `GhiChuTrung`, sắp xếp dữ liệu theo thứ tự tăng dần dựa trên cột khóa đã chọn.

Quy tắc:

- Nếu có một cột khóa, sắp xếp tăng dần theo cột đó.
- Nếu có nhiều cột khóa, sắp xếp tăng dần lần lượt theo thứ tự người dùng đã chọn.
- Nếu các khóa bằng nhau, có thể sắp xếp tiếp theo tên file nguồn, tên sheet hoặc thứ tự dòng nguồn để kết quả ổn định nếu những thông tin này có sẵn.
- Với dữ liệu số và ngày tháng, phải sắp xếp theo giá trị thực tế, không sắp xếp theo chuỗi hiển thị.
- Với chuỗi, sử dụng quy tắc so sánh nhất quán và hỗ trợ tiếng Việt.
- Không tự động sắp xếp theo cột `GhiChuTrung` thay cho cột khóa, trừ khi người dùng chọn riêng.

Nếu sử dụng nhiều cột khóa, ưu tiên sắp xếp theo đúng thứ tự khóa được cấu hình. Không tự ý đổi thứ tự ưu tiên.

## 4. Quy tắc triển khai theo loại đầu ra

### A. Đầu ra SQLite `.db`

Sau khi merge:

1. Mở lại database.
2. Đọc bảng đích.
3. Phân nhóm các bản ghi theo khóa.
4. Bổ sung cột `GhiChuTrung` nếu chưa tồn tại.
5. Cập nhật ghi chú cho từng bản ghi.
6. Sắp xếp kết quả khi truy vấn dữ liệu.
7. Ghi dữ liệu và thông tin đánh dấu trong transaction phù hợp.

Lưu ý quan trọng: SQLite không bảo đảm thứ tự bản ghi khi đọc bảng nếu không có `ORDER BY`.

Vì vậy, không được chỉ dựa vào thứ tự vật lý của dòng trong database. Hãy:

- Lưu cột ghi chú trực tiếp trong bảng đích.
- Tạo truy vấn hoặc VIEW kết quả có `ORDER BY` theo các cột khóa tăng dần.
- Nếu cần bảo đảm thứ tự hiển thị mặc định trong công cụ, tất cả truy vấn hiển thị kết quả phải dùng thứ tự này.
- Nếu cần xuất dữ liệu từ database sang Excel, phải truy vấn theo thứ tự đã cấu hình trước khi ghi file.

Không yêu cầu SQLite phải lưu thứ tự vật lý của các dòng.

Nếu bảng đã có cột `GhiChuTrung`, cập nhật giá trị của cột này theo kết quả kiểm tra mới, không tạo thêm cột trùng tên.

### B. Đầu ra Excel `.xlsx`

Sau khi merge:

1. Đóng tài nguyên ghi workbook.
2. Mở lại file đầu ra.
3. Đọc sheet đích.
4. Kiểm tra trùng theo khóa.
5. Bổ sung hoặc cập nhật cột `GhiChuTrung`.
6. Sắp xếp dữ liệu tăng dần theo khóa.
7. Ghi lại workbook an toàn.

Yêu cầu:

- Giữ lại các sheet khác không liên quan.
- Giữ tiêu đề cột.
- Không ghi đè nhầm sheet khác.
- Giữ kiểu dữ liệu ngày tháng, số và chuỗi.
- Hạn chế làm mất định dạng hiện có.
- Nếu file không thể ghi lại, thông báo lỗi và giữ bản sao lưu hoặc file tạm phù hợp để tránh mất dữ liệu.

### C. Đầu ra Excel `.xls`

Thực hiện cùng quy trình với `.xlsx`, nhưng sử dụng thư viện tương thích `.xls`.

Phải kiểm tra giới hạn 65.536 dòng mỗi sheet, bao gồm dòng tiêu đề.

Nếu vượt giới hạn, thông báo cho người dùng và đề xuất xuất `.xlsx` hoặc chia dữ liệu thành nhiều sheet/file theo cấu hình. Không được âm thầm cắt bỏ dữ liệu.

## 5. Bổ sung thông tin giao diện

Không thay đổi bố cục chính của Excel Merger Pro trong ảnh tham chiếu.

Bổ sung vào khu vực cấu hình nâng cao:

- Checkbox hoặc ComboBox `Kiểm tra trùng theo cột khóa`.
- Danh sách chọn một hoặc nhiều cột khóa.
- Checkbox `Đánh dấu tất cả dòng thuộc nhóm trùng`.
- TextBox `Tên cột ghi chú`, mặc định `GhiChuTrung`.
- Checkbox `Sắp xếp tăng dần theo cột khóa`, mặc định được chọn khi chế độ kiểm tra trùng theo khóa được bật.
- Tùy chọn `Giữ thứ tự ổn định khi khóa bằng nhau`.
- Tùy chọn xử lý khóa rỗng.
- Tùy chọn phân biệt chữ hoa/chữ thường.

Các control chỉ được kích hoạt khi chức năng kiểm tra trùng theo khóa được chọn.

Nếu cửa sổ cấu hình hiện tại không đủ diện tích, tạo cửa sổ cấu hình nâng cao riêng, không làm xáo trộn bố cục giao diện chính.

## 6. Bổ sung tiến trình và nhật ký

Thêm các bước vào tiến trình xử lý:

1. Đang merge dữ liệu.
2. Đang mở lại dữ liệu đầu ra.
3. Đang phân tích các cột khóa.
4. Đang xác định nhóm trùng.
5. Đang bổ sung cột ghi chú.
6. Đang sắp xếp dữ liệu tăng dần.
7. Đang lưu kết quả hậu kiểm.
8. Hoàn thành.

Nhật ký phải ghi nhận:

- Cột khóa đã chọn.
- Số lượng bản ghi được kiểm tra.
- Số nhóm khóa trùng.
- Số dòng thuộc nhóm trùng.
- Số dòng không trùng.
- Số dòng có khóa rỗng.
- Tên cột ghi chú.
- Các cột sắp xếp.
- Thời gian kiểm tra trùng.
- Kết quả lưu dữ liệu.
- Lỗi phát sinh nếu có.

Không sử dụng số liệu giả. Mọi thống kê phải lấy từ kết quả kiểm tra thực tế.

## 7. Bổ sung thống kê kết quả

Trong khu vực `Thông tin kết quả`, bổ sung:

- Số dòng đã kiểm tra trùng.
- Số nhóm trùng.
- Số dòng thuộc nhóm trùng.
- Số dòng không trùng.
- Số dòng khóa rỗng.
- Cột khóa được sử dụng.
- Cột ghi chú được tạo hoặc cập nhật.
- Trạng thái sắp xếp.
- Thời gian hậu kiểm.

Phân biệt rõ:

- `Số nhóm trùng`: số giá trị khóa hoặc tổ hợp khóa khác nhau xuất hiện từ hai lần trở lên.
- `Số dòng thuộc nhóm trùng`: tổng số dòng nằm trong tất cả nhóm trùng.
- `Số dòng trùng vượt lần xuất hiện đầu tiên`: tổng số dòng dư nếu mỗi nhóm chỉ giữ lại một dòng để tính thống kê; chỉ hiển thị chỉ số này nếu cần.

Ví dụ: một khóa xuất hiện 3 lần thì có 1 nhóm trùng, 3 dòng thuộc nhóm trùng và 2 dòng dư so với một bản ghi đại diện.

Không nhầm lẫn ba chỉ số trên.

## 8. Yêu cầu hiệu năng

- Với SQLite, ưu tiên truy vấn SQL, index hoặc bảng tạm phù hợp.
- Với Excel, tránh nạp dữ liệu nhiều lần không cần thiết.
- Không làm treo giao diện.
- Cập nhật tiến trình trong suốt bước hậu kiểm.
- Sử dụng thuật toán có độ phức tạp phù hợp, ưu tiên cách tiếp cận gần O(n log n) hoặc tốt hơn khi phù hợp.
- Với dữ liệu lớn, cân nhắc xử lý theo lô hoặc cấu trúc dữ liệu tối ưu.
- Không sử dụng so sánh chuỗi cho tất cả các kiểu dữ liệu nếu việc đó làm sai kết quả sắp xếp.

## 9. Tiêu chí nghiệm thu bổ sung

Chức năng chỉ được xem là hoàn thành khi:

1. Chỉ chạy khi người dùng chọn kiểm tra trùng theo cột khóa.
2. Đọc lại dữ liệu từ đích sau khi merge.
3. Đánh dấu tất cả dòng thuộc nhóm trùng.
4. Tạo hoặc cập nhật cột `GhiChuTrung`.
5. Không xóa các dòng trùng.
6. Sắp xếp tăng dần theo khóa đã chọn.
7. SQLite hiển thị đúng thứ tự thông qua `ORDER BY` hoặc VIEW.
8. Excel được ghi lại theo đúng thứ tự khóa.
9. Các cột dữ liệu gốc được bảo toàn.
10. Không làm mất sheet khác trong workbook.
11. Số liệu thống kê khớp với dữ liệu thực tế.
12. Xử lý đúng khóa đơn, khóa ghép, khóa rỗng và các kiểu dữ liệu khác nhau.
13. Có log lỗi và xử lý an toàn nếu bước hậu kiểm thất bại.

Hãy tích hợp chức năng này vào kiến trúc hiện tại của Excel Merger Pro. Không viết lại toàn bộ ứng dụng theo cách làm mất các chức năng đã triển khai. Bảo đảm mã nguồn biên dịch được và kiểm thử với dữ liệu thực tế trước khi xác nhận hoàn thành.
