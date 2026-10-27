# PROMPT XÂY DỰNG CÔNG CỤ KSS CHECKER BẰNG POWERSHELL + WINFORMS

## 1. Vai trò

Bạn là chuyên gia phát triển ứng dụng Windows bằng PowerShell, Windows Forms (WinForms), quản trị hệ thống, giám sát mạng và tự động hóa nghiệp vụ CNTT.
- Windows PowerShell 5.1.
- Windows Forms thông qua `System.Windows.Forms`.
- System.Drawing để thiết kế giao diện.
- Các API .NET và Windows có sẵn để truy vấn mạng, thiết bị và hệ thống.
- CSV làm nguồn dữ liệu đầu vào khi phù hợp với mã nguồn gốc.

Sản phẩm chính phải là một file PowerShell `.ps1` có thể chạy trực tiếp trên Windows. Có thể tạo thêm file cấu hình, CSV mẫu, thư viện hỗ trợ hoặc script triển khai nếu thực sự cần thiết, nhưng không được bắt buộc người dùng phải chuyển sang một nền tảng lập trình khác.

## 2. Nguyên tắc xây dựng công cụ

Trước khi viết mã, hãy lập danh sách đầy đủ:

- Các hàm và chức năng hiện có.
- Cấu trúc giao diện chính.
- Các cửa sổ chức năng.
- Biến script-scope và trạng thái dùng chung.
- Timer, event handler, background job và tác vụ bất đồng bộ.
- Cách đọc dữ liệu CSV.
- Các đường dẫn dữ liệu và phụ thuộc bên ngoài.
- Thuật toán Ping, Tracert, nhận diện mạng, sao lưu và phục hồi.
- Những lỗi tiềm ẩn hoặc điểm có thể gây treo giao diện, rò rỉ tài nguyên và sai kết quả.

Đối chiếu từng chức năng của phiên bản mới với phiên bản gốc.

Không tự ý bỏ chức năng, thay đổi quy tắc nghiệp vụ, đổi tên trường dữ liệu hoặc thay đổi cấu trúc thư mục sao lưu nếu chưa xác minh ảnh hưởng.

Nếu không có đủ dữ liệu để xác định chính xác một chức năng, hãy ghi rõ điểm chưa xác minh và tiếp tục triển khai các phần đã xác định được.

## 3. Xây dựng giao diện WinForms

Tạo giao diện desktop theo phong cách công cụ quản trị CNTT, bố cục gọn gàng, dễ thao tác, sử dụng font Segoe UI và màu sắc thống nhất.
Giao diện chính phải cung cấp các nút chức năng:

1. Thông tin card mạng.
2. Kiểm tra kết nối mạng.
3. Quản lý cổng COM và USB.
4. Quản lý máy in, photo và scanner.
5. Quản lý ứng dụng đang chạy.
6. Thông tin cấu hình máy tính.
7. Sao lưu profile người dùng.
8. Phục hồi profile người dùng.

Mỗi chức năng mở cửa sổ riêng tương tự cách tổ chức trong mã nguồn gốc.

Yêu cầu:

- Các nút có tên rõ ràng, có thể thêm biểu tượng phù hợp.
- Hiển thị trạng thái hoạt động và thông báo lỗi.
- Không để giao diện bị treo khi đang truy vấn hệ thống hoặc mạng.
- Xử lý sự kiện đóng cửa sổ đúng cách.
- Dừng timer và giải phóng tài nguyên khi không còn sử dụng.
- Hỗ trợ màn hình có độ phân giải khác nhau.
- Tương thích với môi trường Windows PowerShell 5.1.

## 4. Chức năng thông tin card mạng

Có thể có các hàm chức năng `Get-NetworkInfo`, `Convert-PrefixToSubnetMask` và `Show-NetworkCardDialog`.

Thu thập các thông tin:

- Tên card mạng.
- Trạng thái kết nối.
- Tốc độ liên kết.
- IPv4.
- Subnet mask.
- Default gateway.
- DNS.
- DHCP hoặc IP tĩnh.
- Interface index nếu cần.

Tạo một mảng và biến để lưu các dải mạng `10.56.208.*`, `10.56.240.*`, `10.56.2.*`,... đã được định nghĩa.

Phải sao chép đầy đủ bảng ánh xạ mạng và tên đơn vị từ mã nguồn gốc. Không tự suy đoán các dải IP còn thiếu.

Cập nhật thông tin tự động mỗi 3 giây hoặc theo đúng khoảng thời gian hiện tại của bản gốc.

Xử lý đúng máy có nhiều card mạng, card VPN, card ảo và card không có gateway. Không mặc định card đầu tiên là card có kết nối Internet.

## 5. Chức năng kiểm tra kết nối mạng

Định nghĩa hàm `Load-DeviceIPData` và `Show-NetworkCheckDialog`.

Đọc danh sách thiết bị từ `DeviceIP.csv` với các trường chính `DEVICE` và `IP`, sau khi xác minh cấu trúc file thực tế.

Hiển thị bảng gồm:

- Tên thiết bị.
- IP.
- Kết quả Ping.
- Độ trễ.
- Nút Tracert.
- Trạng thái và thời điểm kiểm tra.

### Ping tự động

Giữ nguyên nghiệp vụ cơ bản của công cụ gốc:

- Đọc danh sách thiết bị từ CSV.
- Ping mỗi thiết bị tối đa 2 lần trong một chu kỳ.
- Nếu có phản hồi thành công, hiển thị độ trễ tốt nhất.
- Nếu không có phản hồi, hiển thị trạng thái thất bại.
- Tự động lặp lại quá trình kiểm tra theo chu kỳ hiện có của mã nguồn.
- Có thể dừng khi đóng cửa sổ.

Không để các chu kỳ Ping chồng lấn hoặc tạo quá nhiều tác vụ đồng thời.

Sử dụng `System.Net.NetworkInformation.Ping` hoặc cơ chế tương đương trong .NET. Có timeout rõ ràng, xử lý ngoại lệ và giải phóng đối tượng Ping.

Không kết luận thiết bị chắc chắn đã tắt chỉ vì không phản hồi ICMP.

### Ping thủ công

Định nghĩa `Show-PingDialog`:

- Cho phép nhập IP hoặc hostname.
- Thực hiện Ping liên tục.
- Hiển thị thời gian phản hồi, kết quả và lỗi.
- Có nút dừng/đóng.
- Giải phóng Timer và đối tượng Ping khi kết thúc.

Cho phép mở nhiều cửa sổ Ping độc lập nếu bản gốc hỗ trợ, không để trạng thái của cửa sổ này ảnh hưởng cửa sổ khác.

### Tracert

Định nghĩa hàm `Show-TracertDialog` bằng PowerShell và .NET.

- Dò đường đi qua các chặng bằng ICMP TTL.
- Hiển thị số thứ tự chặng, IP và thời gian phản hồi.
- Hiển thị `*` khi timeout.
- Có giới hạn số chặng và timeout.
- Cho phép hủy thao tác.
- Hiển thị kết quả theo thời gian thực nếu có thể.
- Không làm treo giao diện.
- Xử lý trường hợp router không phản hồi ICMP TTL Expired.

## 6. Chức năng quản lý COM và USB

Định nghĩa hàm `Get-PortsAndUSB` và `Show-PortsDialog`.

Hiển thị danh sách:

- Cổng COM.
- Tên và mô tả thiết bị.
- Trạng thái thiết bị.
- Thông tin bổ sung mà Windows cung cấp.

Không tự ý chiếm dụng cổng COM đang được ứng dụng khác sử dụng.

Nếu kiểm tra khả năng truy cập cổng, phải phân biệt rõ cổng không tồn tại, cổng đang bận và lỗi quyền truy cập.

## 7. Chức năng máy in, photo và scanner

Định nghĩa hàm `Show-PrinterDialog`.

Thu thập dữ liệu máy in từ Windows, bao gồm:

- Tên máy in.
- Trạng thái.
- Máy in mặc định.
- Cổng kết nối.
- Driver.
- Thông tin chia sẻ nếu có.

Bổ sung khả năng nhận diện scanner nếu hệ thống hỗ trợ.

Không hiển thị thông tin giả hoặc khẳng định scanner đã được nhận diện khi API được sử dụng không cung cấp dữ liệu đó.

## 8. Chức năng quản lý ứng dụng đang chạy

Định nghĩa hàm `Show-AppProcessDialog`.

Đọc `AppProcess.csv` và giữ đúng cấu trúc cột của file gốc sau khi kiểm tra dữ liệu thực tế.

Hiển thị:

- Tên tiến trình.
- Tên đầy đủ.
- Trạng thái đang chạy hoặc không chạy.
- Nút tắt.
- Nút khởi động lại.

Trước khi tắt tiến trình, yêu cầu người dùng xác nhận.

Khi khởi động lại, phải xác định được đường dẫn thực thi hợp lệ. Không tự ý tắt tiến trình hệ thống quan trọng, không kết thúc toàn bộ tiến trình trùng tên và không truyền dữ liệu tùy ý vào lệnh shell.

Nếu không đủ quyền hoặc không tìm được đường dẫn ứng dụng, phải hiển thị thông báo rõ ràng.

## 9. Chức năng thông tin hệ thống

Định nghĩa hàm `Show-SystemInfoDialog`.

Hiển thị thông tin máy tính, hệ điều hành, CPU, RAM, ổ đĩa và các thông tin hệ thống có trong mã nguồn gốc.

Sử dụng CIM/WMI hoặc API Windows phù hợp với PowerShell 5.1.

Không thực hiện truy vấn kéo dài trực tiếp trên UI thread. Có xử lý lỗi nếu không đọc được một trường thông tin.

## 10. Chức năng sao lưu profile

Định nghĩa hàm đầy đủ `Show-BackupDialog`.

Giữ lại các nhóm dữ liệu sao lưu trong mã nguồn gốc, bao gồm:

- Desktop.
- Documents.
- Downloads.
- Chrome Profile.
- Edge Profile.
- Sticky Notes.
- Tệp cấu hình ứng dụng KSS.

Giữ nguyên các đường dẫn, cấu trúc thư mục và quy tắc đặt tên bản sao lưu sau khi đối chiếu mã nguồn thực tế.

Giao diện cần có:

- Checkbox lựa chọn dữ liệu.
- Chọn thư mục đích.
- Thống kê dung lượng.
- Thanh tiến trình.
- Nhật ký sao lưu.
- Thống kê thành công và thất bại.
- Nút bắt đầu và đóng.

Giữ cơ chế ưu tiên 7-Zip nếu có và ZIP nếu không có 7-Zip, tương thích với cách triển khai trong mã nguồn gốc.

Không báo thành công khi có file bị bỏ qua do lỗi. Phải thông báo rõ dữ liệu nào đã sao lưu, dữ liệu nào thất bại và nguyên nhân nếu xác định được.

## 11. Chức năng phục hồi profile

Định nghĩa hàm đầy đủ `Show-RestoreDialog`.

Hỗ trợ nguồn phục hồi theo những định dạng thực sự được mã nguồn gốc hỗ trợ, bao gồm thư mục sao lưu, ZIP và 7z nếu có.

Cho phép:

- Chọn nguồn.
- Chọn các mục cần phục hồi.
- Xác nhận trước khi ghi đè.
- Theo dõi tiến trình.
- Xem nhật ký kết quả.

Bắt buộc kiểm tra đường dẫn khi giải nén, ngăn path traversal, không tự ý xóa toàn bộ profile hiện tại và không ghi đè dữ liệu nếu chưa được xác nhận.

Khi phát sinh lỗi giữa chừng, hiển thị trạng thái phục hồi thực tế và thông tin cần thiết để xử lý tiếp.

## 12. Yêu cầu kỹ thuật riêng cho PowerShell

Tất cả mã nguồn phải tương thích Windows PowerShell 5.1.

Tuân thủ các nguyên tắc:

- Sử dụng `Add-Type -AssemblyName System.Windows.Forms` và `System.Drawing` khi cần.
- Dùng biến `$script:` hoặc đối tượng trạng thái dùng chung có kiểm soát.
- Không để các event handler vô tình mất trạng thái do phạm vi biến.
- Không cập nhật WinForms control trực tiếp từ thread nền.
- Quản lý Timer, Job, Runspace và các đối tượng .NET đúng vòng đời.
- Hạn chế dùng `Start-Job` cho những thao tác ngắn cần cập nhật giao diện thường xuyên nếu có phương án phù hợp hơn.
- Dùng `try/catch/finally` tại các vị trí có khả năng phát sinh lỗi.
- Giải phóng các đối tượng IDisposable đúng cách.
- Không dùng `Invoke-Expression` cho dữ liệu đầu vào không đáng tin cậy.
- Không yêu cầu quyền Administrator nếu chức năng không cần quyền đó.
- Không tự động thay đổi IP, DNS, firewall hoặc cấu hình hệ thống ngoài phạm vi nghiệp vụ gốc.
- Không sử dụng mã hóa hoặc làm rối mã nguồn để che giấu hành vi của ứng dụng.

Ưu tiên cấu trúc hàm rõ ràng, tên hàm nhất quán và chú thích tiếng Việt.

## 13. Cấu trúc và khả năng chạy độc lập

Ưu tiên đọc dữ liệu từ thư mục chứa script bằng `$PSScriptRoot`, không phụ thuộc thư mục làm việc hiện tại.

Khi chạy từ shortcut hoặc một thư mục khác, ứng dụng vẫn phải tìm đúng file CSV.

Nếu thiếu file dữ liệu, thông báo cụ thể và cung cấp hướng dẫn khắc phục; không âm thầm tạo dữ liệu giả.

Có thể cung cấp script đóng gói EXE nếu cần, nhưng file `.ps1` vẫn phải là mã nguồn chính và có thể chạy độc lập trên máy Windows đáp ứng yêu cầu.

## 14. Kiểm thử và xử lý lỗi

Kiểm thử các trường hợp:

- Thiếu hoặc sai định dạng CSV.
- IP không hợp lệ.
- Mất kết nối mạng.
- Ping timeout.
- Nhiều card mạng hoặc VPN.
- Đóng cửa sổ trong lúc chạy tác vụ.
- Không có thiết bị COM/USB.
- Không có máy in.
- Tiến trình không tồn tại hoặc không đủ quyền.
- Sao lưu bị lỗi truy cập file.
- Không có 7-Zip.
- File phục hồi lỗi hoặc đường dẫn không an toàn.
- Các lỗi Timer và event handler.
- Ứng dụng hoạt động lâu mà không rò rỉ tài nguyên.

Không chỉ kiểm tra bằng cách đọc code. Hãy chạy thử trên Windows khi môi trường cho phép, sửa lỗi thực tế và báo cáo trung thực các trường hợp chưa thể kiểm thử.

## 15. Quy trình thực hiện

**Giai đoạn 1:** Đọc toàn bộ mã nguồn gốc và lập bảng chức năng.

**Giai đoạn 2:** Xác định các hàm, biến dùng chung, event handler, Timer và phụ thuộc.

**Giai đoạn 3:** Xây dựng phiên bản PowerShell mới, giữ các chức năng gốc và cải thiện những điểm yếu.

**Giai đoạn 4:** Kiểm tra từng chức năng riêng biệt.

**Giai đoạn 5:** Tích hợp toàn bộ giao diện, xử lý lỗi và quản lý tài nguyên.

**Giai đoạn 6:** Chạy thử, sửa lỗi và kiểm tra hồi quy.

**Giai đoạn 7:** Bàn giao mã nguồn, file dữ liệu mẫu cần thiết và hướng dẫn sử dụng.

Không dừng lại ở việc phân tích, đề xuất hoặc viết các đoạn code minh họa. Hãy trực tiếp tạo và hoàn thiện file mã nguồn trong môi trường làm việc được cung cấp.

## 16. Tiêu chí nghiệm thu

Sản phẩm được xem là đạt yêu cầu khi:

1. Sử dụng PowerShell + WinForms làm công nghệ chính.
2. Có đầy đủ các chức năng trọng yếu của mã nguồn gốc.
3. Giao diện và cách sử dụng tương tự công cụ tham chiếu.
4. Ping và Tracert hoạt động với dữ liệu mạng thực tế.
5. Các file CSV được đọc đúng.
6. Sao lưu và phục hồi có kiểm soát lỗi.
7. Giao diện không bị treo khi thực hiện tác vụ nền.
8. Có cơ chế dừng tác vụ và giải phóng tài nguyên.
9. Không có nút chức năng chỉ mang tính minh họa.
10. Có báo cáo những chức năng đã kiểm thử và các giới hạn còn lại.
