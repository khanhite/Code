# PROMPT XÂY DỰNG PHẦN MỀM QUẢN LÝ VÀ GIÁM SÁT MẠNG NỘI BỘ

## 1. Vai trò và mục tiêu dự án

Bạn là chuyên gia lập trình phần mềm Windows Desktop, có kinh nghiệm về quản trị mạng LAN, TCP/IP, ICMP Ping, DNS, DHCP, ARP, Windows Networking, quản lý thiết bị đầu cuối và thiết kế giao diện phần mềm quản trị hệ thống CNTT.

Hãy phân tích hình ảnh giao diện tham chiếu do tôi cung cấp và xây dựng một phần mềm hoàn chỉnh có tên:

**LAN Network Manager – Công cụ quản lý và giám sát mạng nội bộ**

Mục tiêu là xây dựng một công cụ dành cho cán bộ CNTT sử dụng trong mạng LAN nội bộ tại các cơ quan, đơn vị, có khả năng:

- Quản lý danh sách máy tính, máy chủ và thiết bị mạng thông qua địa chỉ IP.
- Kiểm tra trạng thái kết nối mạng theo thời gian thực.
- Quét, phát hiện và thu thập thông tin thiết bị trong mạng nội bộ.
- Hiển thị kết quả Ping, thông tin IP, Gateway, DNS và các thông số mạng liên quan.
- Theo dõi trạng thái thiết bị, các dịch vụ và tiến trình chẩn đoán mạng.
- Cung cấp các cửa sổ chức năng riêng biệt để theo dõi kết quả thực thi.
- Ghi nhật ký hoạt động và hỗ trợ cán bộ CNTT xác định nhanh các sự cố kết nối.

Phần mềm phải hoạt động ổn định trên Windows 10 và Windows 11, ưu tiên sử dụng ít tài nguyên, phản hồi nhanh và không bị treo giao diện khi thực hiện nhiều tác vụ đồng thời.

## 2. Công nghệ lập trình đề xuất

Sử dụng các công nghệ sau:

- **Ngôn ngữ:** C#.
- **Framework:** .NET 8 LTS hoặc phiên bản .NET LTS còn được hỗ trợ phù hợp với môi trường triển khai.
- **Giao diện:** Windows Forms (WinForms).
- **IDE:** Visual Studio 2022 hoặc phiên bản tương thích.
- **Kiểm tra kết nối:** `System.Net.NetworkInformation.Ping`.
- **Thông tin mạng:** Các API có sẵn trong .NET, DNS, NetworkInterface và các công cụ Windows phù hợp.
- **Xử lý tác vụ:** `async/await`, `Task`, `CancellationToken`, `PeriodicTimer` hoặc cơ chế tương đương.
- **Lưu cấu hình:** JSON hoặc CSV.
- **Lưu nhật ký:** File log UTF-8 có ngày giờ.
- **Biểu tượng giao diện:** SVG, icon hoặc bộ biểu tượng hiện đại phù hợp với ứng dụng quản trị mạng.

Ưu tiên các thư viện có sẵn trong .NET. Chỉ sử dụng thư viện bên ngoài khi thực sự cần thiết; phải nêu rõ mục đích và giấy phép sử dụng.

Ứng dụng phải có thể biên dịch, chạy độc lập trên máy Windows theo cấu hình triển khai đã lựa chọn. Ưu tiên hỗ trợ xuất bản dạng self-contained, không yêu cầu người dùng cài đặt môi trường lập trình.

## 3. Phân tích và thiết kế giao diện theo hình ảnh tham chiếu

Sử dụng hình ảnh tôi cung cấp làm cơ sở thiết kế giao diện. Giữ phong cách của phần mềm quản trị mạng Windows truyền thống: cửa sổ nhỏ gọn, nhiều bảng dữ liệu, thanh công cụ màu xanh, các nút chức năng rõ ràng và các cửa sổ kết quả chẩn đoán riêng biệt.

Không sao chép máy móc các chi tiết không thể đọc được từ hình ảnh có độ phân giải thấp. Hãy xây dựng giao diện tương đương về bố cục, chức năng và trải nghiệm sử dụng, đồng thời nâng cấp tính thẩm mỹ.

### 3.1. Cửa sổ chính – Main Dashboard

Cửa sổ chính gồm các thành phần:

- Thanh tiêu đề hiển thị tên ứng dụng và phiên bản.
- Thanh công cụ gồm các chức năng quản lý thiết bị, quét mạng, kiểm tra kết nối, cấu hình và nhật ký.
- Khu vực tổng quan số lượng thiết bị, thiết bị đang hoạt động, thiết bị mất kết nối và tác vụ đang thực hiện.
- Khu vực điều khiển nhanh để bắt đầu, tạm dừng hoặc dừng các tác vụ kiểm tra.
- Thanh trạng thái hiển thị trạng thái ứng dụng, thời điểm cập nhật dữ liệu và số lượng thiết bị đang được giám sát.

Bố cục cần có khả năng co giãn theo kích thước cửa sổ. Ưu tiên sử dụng `SplitContainer`, `TableLayoutPanel`, `Panel` và `DataGridView` để tổ chức giao diện.

### 3.2. Cửa sổ quản lý danh sách IP

Thiết kế cửa sổ tương tự khu vực phía trên bên trái của ảnh tham chiếu.

Các thành phần:

- Ô nhập địa chỉ IP hoặc tên máy tính.
- Ô tìm kiếm, lọc danh sách.
- Nút thêm, sửa, xóa thiết bị.
- Nút Ping, kiểm tra kết nối và làm mới.
- Bảng danh sách thiết bị.

Bảng dữ liệu gồm các cột:

| Tên cột | Mô tả |
|---|---|
| STT | Số thứ tự |
| Loại thiết bị | Máy trạm, máy chủ, switch, router hoặc thiết bị khác |
| Tên thiết bị | Tên máy tính hoặc tên do quản trị viên đặt |
| Địa chỉ IP | Địa chỉ IPv4 hoặc IPv6 phù hợp với chức năng |
| Trạng thái | Online, Offline, Đang kiểm tra hoặc Lỗi |
| Thời gian phản hồi | Độ trễ Ping gần nhất |
| Thao tác | Ping, xem chi tiết, chỉnh sửa |

Sử dụng màu sắc để nhận diện trạng thái: xanh lá cho kết nối thành công, đỏ cho mất kết nối, vàng cho đang kiểm tra và xám cho trạng thái chưa xác định.

Cho phép nhập danh sách thiết bị từ file CSV có cấu trúc:

`LOAI,TEN,IP`

Hỗ trợ lưu và tải danh sách thiết bị để không phải nhập lại sau mỗi lần khởi động.

### 3.3. Cửa sổ quét mạng và phát hiện thiết bị

Thiết kế cửa sổ tương tự khu vực phía trên bên phải ảnh tham chiếu.

Chức năng:

- Nhập dải IP hoặc subnet cần kiểm tra.
- Chọn số lượng tác vụ đồng thời.
- Chọn thời gian chờ phản hồi.
- Bắt đầu, tạm dừng và dừng quá trình quét.
- Hiển thị tiến độ thực hiện.
- Lọc thiết bị đang hoạt động và xuất kết quả.

Các trường thông tin:

- Địa chỉ IP.
- Tên máy chủ DNS nếu phân giải được.
- MAC address nếu có thể thu thập hợp lệ từ mạng cục bộ.
- Trạng thái phản hồi.
- Thời gian phản hồi.
- Thông tin nhà sản xuất theo OUI nếu có dữ liệu tra cứu cục bộ.
- Thời điểm phát hiện gần nhất.

Chỉ thực hiện quét trên các dải địa chỉ được quản trị viên cho phép. Không mặc định quét toàn bộ mạng khi chưa có cấu hình.

Lưu ý kỹ thuật: Ping không thành công không đồng nghĩa thiết bị chắc chắn đã tắt. Một số thiết bị có thể chặn ICMP hoặc áp dụng chính sách tường lửa. Phải thể hiện trạng thái theo đúng kết quả thu thập được.

### 3.4. Cửa sổ Ping Monitor

Thiết kế một cửa sổ riêng để hiển thị kết quả kiểm tra kết nối, tương tự cửa sổ phía dưới bên trái của ảnh.

Nội dung gồm:

- Danh sách IP được kiểm tra.
- Thời điểm gửi gói tin.
- Trạng thái phản hồi.
- Thời gian phản hồi tính bằng ms.
- TTL nếu hệ điều hành cung cấp.
- Số lần thành công, thất bại và tỷ lệ mất gói.
- Tổng thời gian giám sát.

Các nút điều khiển:

- Bắt đầu Ping.
- Tạm dừng.
- Dừng.
- Xóa kết quả.
- Xuất nhật ký.

Yêu cầu kỹ thuật:

- Mỗi lần kiểm tra mặc định gửi tối đa 2 gói ICMP cho mỗi thiết bị.
- Cho phép thay đổi số gói và khoảng thời gian kiểm tra trong phần cấu hình.
- Không gửi Ping liên tục không giới hạn với tần suất quá cao.
- Có thể giám sát nhiều IP đồng thời nhưng phải giới hạn số tác vụ song song.
- Không sử dụng vòng lặp đồng bộ làm treo giao diện.
- Hiển thị kết quả ngay khi có phản hồi.
- Khi bấm Dừng, phải hủy các tác vụ đang chờ thông qua `CancellationToken` hoặc cơ chế hủy tương đương.
- Không ghi đè dữ liệu của thiết bị này lên thiết bị khác khi nhiều tác vụ trả kết quả đồng thời.

Mỗi bản ghi log cần có thời gian, địa chỉ IP, kết quả, độ trễ và thông báo lỗi nếu có.

### 3.5. Cửa sổ thông tin mạng của máy tính

Thiết kế cửa sổ hiển thị thông tin mạng tương tự khu vực bên phải của hình ảnh.

Tự động thu thập các thông tin có thể truy xuất hợp lệ từ máy đang chạy ứng dụng:

- Hostname.
- Tên người dùng Windows hiện tại.
- Danh sách card mạng.
- Địa chỉ IPv4 và IPv6.
- Subnet mask hoặc prefix length.
- Default Gateway.
- DNS Server.
- Địa chỉ MAC.
- DHCP Enabled.
- Trạng thái kết nối card mạng.
- Địa chỉ IP nguồn được hệ điều hành lựa chọn khi kết nối đến một đích cụ thể.
- Thông tin route và bảng ARP nếu người dùng có đủ quyền truy cập.

Bố trí thông tin theo từng nhóm, dễ đọc, có nút sao chép và làm mới.

Đối với dữ liệu cần quyền quản trị, hãy hiển thị thông báo yêu cầu quyền phù hợp thay vì làm ứng dụng lỗi.

### 3.6. Cửa sổ danh sách dịch vụ và tiến trình chẩn đoán

Thiết kế cửa sổ tương tự khu vực trung tâm phía dưới ảnh tham chiếu.

Bảng gồm các cột:

- STT.
- Tên chức năng.
- Trạng thái.
- Nội dung kết quả.
- Thời gian thực hiện.
- Nút thao tác.

Các chức năng có thể bao gồm:

1. Kiểm tra kết nối Gateway.
2. Kiểm tra kết nối DNS.
3. Kiểm tra phân giải tên miền.
4. Kiểm tra kết nối đến máy chủ nội bộ.
5. Kiểm tra một cổng TCP được quản trị viên cấu hình.
6. Thu thập thông tin card mạng.
7. Thu thập thông tin route.
8. Kiểm tra trạng thái các dịch vụ Windows được cấu hình.
9. Kiểm tra khả năng truy cập tài nguyên mạng được phép.
10. Tổng hợp kết quả chẩn đoán.

Mỗi chức năng phải có trạng thái riêng: Chưa chạy, Đang chạy, Thành công, Cảnh báo, Thất bại hoặc Đã hủy.

Không được mặc định mọi dịch vụ Windows đều cần chạy. Chỉ kiểm tra các dịch vụ liên quan đến chức năng đã được cấu hình.

Không tự động dừng dịch vụ, sửa cấu hình IP, thay đổi DNS, sửa firewall hoặc thực hiện thao tác ảnh hưởng đến hệ thống. Các thao tác thay đổi cấu hình phải có chức năng riêng, kiểm tra quyền, cảnh báo và yêu cầu xác nhận của người dùng.

### 3.7. Cửa sổ kết quả lệnh và nhật ký

Thiết kế cửa sổ tương tự khu vực phía dưới bên phải ảnh tham chiếu.

Yêu cầu:

- Hiển thị kết quả chẩn đoán theo thời gian thực.
- Có thời gian bắt đầu và kết thúc tác vụ.
- Hiển thị lệnh hoặc tên thao tác đã thực hiện.
- Phân biệt thông tin, cảnh báo và lỗi.
- Có nút Xóa log, Sao chép và Xuất file.
- Tự động cuộn đến nội dung mới nhất nhưng cho phép người dùng tắt tự động cuộn.
- Giới hạn số dòng hiển thị nhằm tránh sử dụng quá nhiều RAM.
- Lưu log xuống file UTF-8 theo ngày.

Nếu sử dụng các công cụ Windows như `ipconfig`, `route`, `arp`, `nslookup` hoặc `tracert`, hãy gọi thông qua danh sách thao tác được định nghĩa trước, không ghép chuỗi đầu vào của người dùng thành lệnh hệ thống tùy ý.

Phải đọc stdout và stderr không gây deadlock, có timeout, hỗ trợ hủy và xử lý tiến trình con khi người dùng dừng tác vụ.

## 4. Thiết kế kiến trúc phần mềm

Tổ chức dự án theo cấu trúc dễ bảo trì, không viết toàn bộ chức năng vào một lớp Form duy nhất.

Đề xuất cấu trúc:

```text
LANNetworkManager/
├── Program.cs
├── Forms/
│   ├── MainForm.cs
│   ├── DeviceManagerForm.cs
│   ├── NetworkScannerForm.cs
│   ├── PingMonitorForm.cs
│   ├── NetworkInfoForm.cs
│   ├── DiagnosticsForm.cs
│   └── LogViewerForm.cs
├── Models/
│   ├── NetworkDevice.cs
│   ├── PingResult.cs
│   ├── NetworkAdapterInfo.cs
│   ├── DiagnosticResult.cs
│   └── AppSettings.cs
├── Services/
│   ├── PingService.cs
│   ├── NetworkScanService.cs
│   ├── NetworkInfoService.cs
│   ├── DnsService.cs
│   ├── TcpCheckService.cs
│   └── DiagnosticService.cs
├── Data/
│   ├── DeviceRepository.cs
│   ├── CsvService.cs
│   └── SettingsService.cs
├── Logging/
│   └── AppLogger.cs
├── Resources/
├── Config/
└── README.md
```

Có thể điều chỉnh cấu trúc nếu cần, nhưng phải đảm bảo tách biệt giao diện, xử lý nghiệp vụ, dữ liệu và nhật ký.

## 5. Yêu cầu về hiệu năng và độ ổn định

Đây là yêu cầu bắt buộc:

1. Giao diện luôn phản hồi khi quét mạng hoặc Ping nhiều thiết bị.
2. Không gọi các thao tác mạng đồng bộ trực tiếp trên UI thread.
3. Giới hạn concurrency để tránh quá tải máy tính và mạng.
4. Hủy tác vụ đúng cách khi đóng cửa sổ hoặc thoát chương trình.
5. Không tạo hàng nghìn tác vụ cùng lúc khi quét subnet lớn.
6. Có timeout cho các thao tác mạng và tiến trình ngoài.
7. Xử lý ngoại lệ ở từng lớp, ghi log và hiển thị thông báo thân thiện.
8. Không sử dụng vòng lặp kiểm tra liên tục không có khoảng nghỉ.
9. Không làm mất danh sách thiết bị hoặc cấu hình khi khởi động lại.
10. Có thể chạy ứng dụng trong môi trường mạng không có Internet.
11. Hỗ trợ tiếng Việt có dấu, không lỗi font hoặc mã hóa CSV.
12. Tự giải phóng tài nguyên, timer, socket và tiến trình khi đóng ứng dụng.

Không tuyên bố thiết bị Online chỉ dựa vào dữ liệu cũ. Mỗi kết quả cần có thời điểm kiểm tra cuối cùng và trạng thái hết hạn nếu không được cập nhật trong khoảng thời gian cấu hình.

## 6. Yêu cầu về giao diện và trải nghiệm người dùng

- Giao diện Windows Desktop hiện đại, gọn nhẹ, phù hợp cho cán bộ CNTT.
- Có thể mở các cửa sổ chức năng đồng thời.
- Không để các cửa sổ con che khuất hoàn toàn cửa sổ chính.
- Có thể thay đổi kích thước cửa sổ.
- Các bảng hỗ trợ sắp xếp, lọc và tự động điều chỉnh cột.
- Có trạng thái trực quan bằng màu sắc và biểu tượng.
- Các nút thao tác phải có chức năng thực tế.
- Có hộp thoại xác nhận đối với thao tác xóa hoặc thay đổi dữ liệu.
- Lưu lại vị trí và kích thước cửa sổ nếu phù hợp.
- Hỗ trợ chế độ sáng/tối nếu triển khai được mà không làm phức tạp hoặc giảm hiệu năng ứng dụng.
- Hiển thị phiên bản phần mềm và thông tin tác giả ở khu vực phù hợp.

Ưu tiên sử dụng `DataGridView` được cấu hình tốt, tránh hiệu ứng đồ họa nặng và tránh cập nhật toàn bộ bảng mỗi khi có một kết quả Ping mới.

## 7. Yêu cầu về bảo mật

- Chỉ sử dụng công cụ trong mạng nội bộ được phép quản lý.
- Không thu thập mật khẩu, thông tin xác thực hoặc dữ liệu cá nhân không cần thiết.
- Không gửi thông tin thiết bị đến dịch vụ bên ngoài.
- Không tự động tải hoặc thực thi mã từ Internet.
- Không lưu thông tin nhạy cảm trong log.
- Không yêu cầu quyền Administrator nếu chức năng hiện tại không cần.
- Kiểm tra và xác thực địa chỉ IP, subnet, cổng TCP, đường dẫn file và dữ liệu nhập vào.
- Giới hạn các lệnh hệ thống trong danh sách được kiểm soát.
- Có thể cấu hình dải mạng được phép kiểm tra và số tác vụ đồng thời.
- Không thực hiện quét cổng diện rộng hoặc thay đổi cấu hình thiết bị ngoài phạm vi được cấp phép.

## 8. Yêu cầu kiểm thử

Tạo dữ liệu thử nghiệm và kiểm tra tối thiểu các trường hợp sau:

- IP hợp lệ, IP không hợp lệ.
- Thiết bị phản hồi Ping.
- Thiết bị không phản hồi Ping.
- Thiết bị chặn ICMP nhưng vẫn có thể truy cập qua TCP.
- Không có card mạng hoặc card mạng bị ngắt kết nối.
- DNS phân giải thành công và thất bại.
- Gateway không phản hồi.
- File CSV thiếu cột, sai định dạng hoặc có ký tự tiếng Việt.
- Người dùng nhấn Dừng trong lúc quét mạng.
- Người dùng đóng cửa sổ trong lúc tác vụ đang chạy.
- Nhiều tác vụ hoàn thành cùng lúc.
- Không có quyền quản trị.
- Tiến trình hệ thống bị timeout hoặc trả về lỗi.
- Chạy ứng dụng liên tục trong thời gian dài.

Mỗi trường hợp phải có kết quả mong đợi, thông báo lỗi phù hợp và không làm ứng dụng bị treo.

## 9. Sản phẩm đầu ra bắt buộc

Hãy cung cấp đầy đủ các sản phẩm sau:

1. Mã nguồn C# hoàn chỉnh của dự án.
2. File `.csproj` với các cấu hình cần thiết.
3. Mã nguồn tất cả Form, Model, Service và Repository.
4. Giao diện được tạo bằng mã hoặc Designer, đảm bảo các thành phần hiển thị đúng.
5. File cấu hình mẫu và dữ liệu thiết bị mẫu.
6. File README hướng dẫn cài đặt, biên dịch, chạy và sử dụng.
7. Hướng dẫn xuất bản ứng dụng thành file EXE hoặc thư mục triển khai độc lập.
8. Bộ kiểm thử cho các chức năng mạng quan trọng.
9. Danh sách các giới hạn kỹ thuật và các chức năng cần quyền quản trị.
10. Hướng dẫn khắc phục các lỗi thường gặp.

Không được chỉ tạo giao diện minh họa, mã giả, các hàm rỗng hoặc nút bấm không có chức năng.

Không được bỏ qua những lớp quan trọng bằng các chú thích như `TODO`, “triển khai sau” hoặc “viết tương tự”.

Nếu không thể xuất trực tiếp file dự án, hãy cung cấp mã nguồn đầy đủ của từng file, ghi rõ đường dẫn và nội dung để có thể tạo lại dự án và biên dịch thành công.

## 10. Quy trình thực hiện

Thực hiện dự án theo các giai đoạn:

**Giai đoạn 1 – Phân tích:** Phân tích ảnh tham chiếu, xác định các cửa sổ, thành phần giao diện, chức năng và luồng dữ liệu.

**Giai đoạn 2 – Xây dựng giao diện:** Tạo cửa sổ chính, các cửa sổ con và bảng dữ liệu theo bố cục yêu cầu.

**Giai đoạn 3 – Xây dựng nghiệp vụ:** Triển khai quản lý IP, Ping, quét mạng, thu thập thông tin card mạng, DNS, kiểm tra TCP và nhật ký.

**Giai đoạn 4 – Tích hợp:** Kết nối giao diện với các dịch vụ, xử lý tác vụ bất đồng bộ, đồng bộ trạng thái và hỗ trợ hủy.

**Giai đoạn 5 – Kiểm thử:** Kiểm tra các trường hợp thành công, thất bại, timeout, lỗi quyền và khả năng phản hồi của giao diện.

**Giai đoạn 6 – Đóng gói:** Hoàn thiện tài liệu, cấu hình triển khai và hướng dẫn biên dịch thành ứng dụng Windows.

Sau mỗi giai đoạn, hãy kiểm tra tính nhất quán của mã nguồn trước khi tiếp tục.

## 11. Tiêu chí nghiệm thu

Phần mềm chỉ được xem là hoàn thành khi:

- Giao diện tương đương về bố cục và chức năng với hình ảnh tham chiếu.
- Tất cả các cửa sổ chức năng có thể mở, sử dụng và đóng bình thường.
- Danh sách IP được lưu và tải lại thành công.
- Ping và quét mạng hiển thị kết quả thực tế.
- Các tác vụ không làm treo giao diện.
- Chức năng Dừng có hiệu lực.
- Nhật ký ghi nhận đúng hoạt động và lỗi.
- Dữ liệu tiếng Việt được lưu và đọc chính xác.
- Ứng dụng có thể biên dịch thành công trên môi trường đã chọn.
- Không có lỗi nghiêm trọng làm mất dữ liệu hoặc khiến ứng dụng không thể sử dụng.

**Yêu cầu cuối cùng:** Hãy bắt đầu bằng việc phân tích ảnh giao diện, đề xuất sơ đồ các cửa sổ và cấu trúc dự án. Sau đó triển khai lần lượt từng thành phần thành mã nguồn thực tế, có thể biên dịch, chạy và kiểm thử. Ưu tiên tính ổn định, hiệu năng và khả năng sử dụng thực tế trong mạng nội bộ cơ quan hơn các hiệu ứng đồ họa không cần thiết.
