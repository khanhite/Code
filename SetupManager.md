# YÊU CẦU PHÁT TRIỂN CÔNG CỤ POWERSELL WINFORMS QUẢN LÝ PHẦN MỀM

## 1. Mục tiêu

Tôi có một thư mục gốc tên **`Resource`**, bên trong chứa các file cài đặt/thực thi phần mềm được đánh số thứ tự như hình ảnh mẫu đã cung cấp, ví dụ:

* `1.DSKhanhHoa_Agent_Patch.exe`
* `2.Kaspersky-056.exe`
* `3.SafeNetAuthenticationClient-x64-10.9-....`
* `4.bit4id_xpki_1.4.10.824-ng-bit4id-user-...`
* `5.vss-internal_2.0.8.50.exe`
* `6.Webisolate` — thư mục
* `7.PulseSecure.x64.msi`
* `8.ZaloSetup-25.4.2.exe`
* `9.tsetup-x64.7.0.9.exe`
* `10.WPSOffice-Online.exe`
* `11.accessdatabaseengine_X64.exe`
* `12.XoaRac-chung.exe`

Hãy xây dựng một **ứng dụng PowerShell sử dụng Windows Forms (WinForms)** để quản lý việc kiểm tra, cài đặt và gỡ cài đặt các phần mềm trong thư mục `Resource`.

Ứng dụng phải có giao diện trực quan, dễ sử dụng, chạy ổn định trên Windows 10/11 và ưu tiên khả năng chạy độc lập mà không yêu cầu người dùng phải chỉnh sửa mã nguồn.

---

# 2. Công nghệ và yêu cầu chung

* Ngôn ngữ: **PowerShell**.
* Giao diện: **Windows Forms (`System.Windows.Forms`)**.
* Có thể sử dụng `System.Drawing` để thiết kế giao diện.
* Toàn bộ chương trình nằm trong **một file `.ps1` duy nhất**, nếu không có lý do bắt buộc phải tách file.
* Không sử dụng WPF.
* Không sử dụng thư viện bên ngoài nếu không thật sự cần thiết.
* Code phải được tổ chức thành các hàm riêng biệt, dễ bảo trì.
* Có xử lý exception bằng `try/catch`.
* Không để lỗi PowerShell làm ứng dụng tự động thoát.
* Các thao tác cài đặt/gỡ cài đặt phải chạy bất đồng bộ hoặc thông qua cơ chế phù hợp để **không làm treo giao diện WinForms**.
* Các thao tác yêu cầu quyền Administrator phải tự động kiểm tra quyền và có cơ chế yêu cầu chạy dưới quyền Administrator khi cần.

---

# 3. Cấu trúc thư mục Resource

Mặc định chương trình tìm thư mục:

```text
.\Resource
```

theo thư mục chứa file `.ps1`.

Ví dụ:

```text
Tool\
│
├── SoftwareManager.ps1
│
├── Resource\
│   ├── 1.DSKhanhHoa_Agent_Patch.exe
│   ├── 2.Kaspersky-056.exe
│   ├── 3.SafeNetAuthenticationClient-x64-10.9-....exe
│   ├── 4.bit4id_xpki_1.4.10.824-ng-bit4id-user-....exe
│   ├── 5.vss-internal_2.0.8.50.exe
│   ├── 6.Webisolate\
│   │   └── ...
│   ├── 7.PulseSecure.x64.msi
│   ├── 8.ZaloSetup-25.4.2.exe
│   ├── 9.tsetup-x64.7.0.9.exe
│   ├── 10.WPSOffice-Online.exe
│   ├── 11.accessdatabaseengine_X64.exe
│   └── 12.XoaRac-chung.exe
│
└── AppProcess.csv
```

Nếu không tìm thấy `Resource`, phải hiển thị thông báo lỗi rõ ràng trên giao diện thay vì để chương trình crash.

---

# 4. Giai đoạn khởi động và Loading

Khi chương trình khởi động:

1. Hiển thị một form loading riêng.
2. Hiển thị thông tin, ví dụ:

```text
Đang tải danh sách phần mềm...
Vui lòng chờ...
```

3. Có `ProgressBar` chạy trong quá trình:

   * Kiểm tra thư mục `Resource`.
   * Đọc danh sách file.
   * Đọc các subfolder.
   * Phân tích tên file.
   * Đọc `AppProcess.csv`.
   * Kiểm tra process đang chạy.
   * Xác định trạng thái từng phần mềm.

4. Không được làm treo giao diện trong quá trình loading.

5. Sau khi hoàn tất, đóng Loading Form và hiển thị Main Form.

---

# 5. Đọc file trong Resource

Chương trình phải quét toàn bộ thư mục `Resource`, bao gồm cả các thư mục con.

## 5.1. Ưu tiên file thực thi

Ưu tiên phát hiện các file có extension:

```text
.exe
.msi
.cmd
.bat
.ps1
.com
.scr
```

Có thể bổ sung các extension thực thi khác nếu phù hợp.

## 5.2. Xử lý Subfolder

Nếu trong `Resource` có subfolder, ví dụ:

```text
Resource\
└── 6.Webisolate\
    ├── setup.exe
    ├── uninstall.exe
    └── ...
```

thì phải tiếp tục quét các file bên trong.

Ưu tiên lấy file cài đặt/thực thi chính trong subfolder.

Không được coi mỗi file phụ trợ DLL, TXT, XML... là một phần mềm độc lập.

## 5.3. Xác định STT

STT phải được lấy từ phần số ở đầu tên file/thư mục.

Ví dụ:

```text
1.DSKhanhHoa_Agent_Patch.exe
```

→ STT = `1`

```text
10.WPSOffice-Online.exe
```

→ STT = `10`

Không được sắp xếp theo thứ tự alphabet khiến `10` đứng trước `2`.

Phải sắp xếp theo số:

```text
1
2
3
...
9
10
11
12
```

---

# 6. Giao diện Main Form

Sau khi loading hoàn tất, hiển thị Main Form.

Giao diện phải có bố cục từ trên xuống dưới như sau:

## 6.1. Danh sách phần mềm

Mỗi phần mềm hiển thị thành một dòng gồm:

```text
[STT] - [Tên file/phần mềm] - [Trạng thái] - [Button]
```

Ví dụ:

```text
1 - DSKhanhHoa_Agent_Patch.exe - Đang chạy     [Uninstall]
2 - Kaspersky-056.exe             - Chưa cài    [Setup]
3 - SafeNetAuthenticationClient   - Đang chạy   [Uninstall]
```

Có thể sử dụng `TableLayoutPanel`, `FlowLayoutPanel` hoặc một control phù hợp để tạo danh sách động.

Khuyến nghị sử dụng `TableLayoutPanel` để căn chỉnh các cột đồng đều.

Các cột gồm:

```text
STT | Tên phần mềm | Trạng thái | Thao tác
```

---

# 7. Trạng thái phần mềm

Thống nhất sử dụng hai trạng thái chính:

### Trạng thái 1: `Đang chạy`

Phần mềm được xem là đang chạy khi:

1. Tìm thấy process tương ứng trong Windows.
2. Lấy được đường dẫn executable thực tế của process.
3. Đường dẫn executable này tồn tại trong `AppProcess.csv`.

Khi trạng thái là:

```text
Đang chạy
```

thì hiển thị:

```text
[Uninstall]
```

### Trạng thái 2: `Chưa cài`

Nếu không tìm thấy process tương ứng hoặc không xác định được executable tương ứng trong `AppProcess.csv`, hiển thị:

```text
Chưa cài
```

và button:

```text
[Setup]
```

---

# 8. Đọc và sử dụng AppProcess.csv

Chương trình phải đọc file:

```text
AppProcess.csv
```

nằm cùng cấp với file `.ps1`, hoặc nếu thiết kế hợp lý hơn thì hỗ trợ tìm trong `Resource`.

File này dùng để ánh xạ:

```text
Tên phần mềm
→
Tên process / đường dẫn executable
```

Ví dụ có thể có cấu trúc:

```csv
AppName,ProcessName,ExePath
Kaspersky,Kaspersky.exe,C:\Program Files\Kaspersky\Kaspersky.exe
Zalo,Zalo.exe,C:\Users\...\Zalo.exe
WPS Office,wps.exe,C:\Program Files\WPS Office\...
```

Nếu cấu trúc `AppProcess.csv` thực tế khác, hãy thiết kế hàm đọc CSV theo hướng dễ chỉnh sửa và ghi rõ trong comment vị trí cần thay đổi.

---

# 9. Kiểm tra process đang chạy

Không được chỉ kiểm tra tên process.

Phải ưu tiên lấy **đường dẫn executable thực tế** của process đang chạy.

Có thể sử dụng:

```powershell
Get-CimInstance Win32_Process
```

hoặc phương pháp tương đương.

Ví dụ:

```text
ProcessName = Zalo.exe
ExecutablePath = C:\Users\User\AppData\Local\Zalo\Zalo.exe
```

Sau đó so sánh `ExecutablePath` với đường dẫn được khai báo trong `AppProcess.csv`.

Phải chuẩn hóa đường dẫn trước khi so sánh:

* Không phân biệt chữ hoa/chữ thường.
* Xử lý dấu `\` cuối đường dẫn.
* Có thể xử lý đường dẫn có quotation mark.
* Có thể xử lý biến môi trường nếu cần.
* Có thể xử lý đường dẫn tương đối thành absolute path.

---

# 10. Quy tắc xác định trạng thái

Logic phải tương đương:

```text
Đọc danh sách phần mềm
        ↓
Đọc AppProcess.csv
        ↓
Lấy toàn bộ process đang chạy
        ↓
Lấy ExecutablePath của process
        ↓
So sánh với AppProcess.csv
        ↓
Có khớp?
   ┌───────┴────────┐
   │                │
  YES               NO
   │                │
Đang chạy         Chưa cài
   │                │
[Uninstall]       [Setup]
```

Nếu không thể lấy `ExecutablePath` do quyền truy cập hoặc process đặc biệt, phải ghi rõ nguyên nhân vào log và không được làm crash ứng dụng.

---

# 11. Chức năng Setup

Khi người dùng nhấn:

```text
[Setup]
```

chương trình phải:

1. Xác định file thực thi tương ứng.
2. Xác định đường dẫn đầy đủ.
3. Hiển thị log:

```text
Đang cài đặt: <Tên file>
```

4. Chạy file cài đặt.
5. Đối với `.exe`:

   * Cho phép truyền tham số silent nếu có cấu hình.
   * Nếu chưa biết tham số silent, sử dụng chế độ chạy thông thường.
6. Đối với `.msi`:

   * Có thể sử dụng `msiexec.exe`.
7. Đối với `.cmd/.bat`:

   * Chạy thông qua `cmd.exe /c`.
8. Không làm treo Main Form.
9. Theo dõi process cài đặt nếu có thể.
10. Sau khi cài đặt hoàn tất, tự động refresh trạng thái.

Ví dụ:

```text
[14:25:01] Bắt đầu cài đặt ZaloSetup-25.4.2.exe
[14:25:15] Process cài đặt đã hoàn tất.
[14:25:16] Đang kiểm tra lại trạng thái...
[14:25:17] Trạng thái: Đang chạy
```

---

# 12. Chức năng Uninstall

Khi phần mềm có trạng thái:

```text
Đang chạy
```

thì hiển thị:

```text
[Uninstall]
```

Khi nhấn Uninstall:

## Bước 1 – Xác định process

Tìm process dựa trên:

```text
AppProcess.csv
```

và lấy:

```text
ProcessName
ExecutablePath
ProcessId
```

## Bước 2 – Đề xuất uninstall command

Chương trình phải cố gắng xác định lệnh uninstall từ:

```text
HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*
HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*
HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*
```

Đọc các trường:

```text
DisplayName
DisplayVersion
UninstallString
QuietUninstallString
InstallLocation
```

Ưu tiên:

```text
QuietUninstallString
```

nếu có.

Nếu không có thì sử dụng:

```text
UninstallString
```

và đề xuất tham số silent phù hợp nếu có thể xác định an toàn.

## Bước 3 – Đóng process

Trước khi uninstall:

1. Ghi log process đang chạy.
2. Hiển thị xác nhận:

```text
Bạn có chắc chắn muốn gỡ cài đặt phần mềm này không?
```

3. Nếu người dùng đồng ý:

   * Thử đóng process một cách bình thường.
   * Nếu không đóng được, có thể sử dụng `Stop-Process`.
   * Chỉ sử dụng `taskkill /F` khi cần thiết.

Không được tự động kill các process hệ thống không liên quan.

## Bước 4 – Chạy uninstall

Sau khi process đã được đóng:

```text
Thực hiện Uninstall
```

theo `QuietUninstallString` hoặc `UninstallString`.

Sau khi hoàn tất:

```text
Refresh trạng thái
```

Nếu phần mềm không còn tồn tại:

```text
Chưa cài
[Setup]
```

---

# 13. Log TextBox

Phía dưới danh sách phần mềm phải có một:

```text
Multi-line TextBox
```

với:

```text
ReadOnly = True
Multiline = True
ScrollBars = Vertical
```

Dùng để hiển thị log realtime.

Mỗi log nên có timestamp:

```text
[14:25:01] Đang kiểm tra Resource...
[14:25:02] Đã tìm thấy 12 phần mềm.
[14:25:03] Đang kiểm tra process...
[14:25:04] Kaspersky - Đang chạy.
[14:25:04] Zalo - Chưa cài.
```

Phải tự động scroll xuống dòng mới nhất.

Có thể bổ sung button:

```text
[Clear Log]
```

nếu phù hợp.

---

# 14. Footer

Ở góc dưới cùng bên trái Main Form hiển thị:

```text
R&D by KSS
```

Font nhỏ, giao diện gọn gàng.

---

# 15. Refresh trạng thái

Sau mỗi thao tác:

* Setup
* Uninstall
* Đóng process
* Cài đặt hoàn tất
* Gỡ cài đặt hoàn tất

phải tự động kiểm tra lại trạng thái.

Không yêu cầu người dùng đóng và mở lại chương trình.

Có thể bổ sung button:

```text
[Refresh]
```

ở phía trên giao diện để người dùng chủ động kiểm tra lại.

---

# 16. Yêu cầu về giao diện

Thiết kế giao diện theo phong cách:

* Hiện đại.
* Gọn gàng.
* Dễ sử dụng.
* Font dễ đọc.
* Các dòng có chiều cao phù hợp.
* Tên file dài phải được hiển thị đầy đủ hoặc có tooltip.
* Button Setup và Uninstall phải dễ phân biệt.
* Trạng thái phải dễ nhận biết bằng màu sắc hoặc icon.
* Không để giao diện bị vỡ khi tên file rất dài.

Kích thước Main Form có thể khoảng:

```text
Width: 900–1100
Height: 650–750
```

và cho phép resize.

Khi resize form, danh sách phần mềm và Log TextBox phải tự động giãn theo cửa sổ.

---

# 17. Xử lý lỗi

Tất cả các tình huống sau phải được xử lý:

* Không tìm thấy Resource.
* Resource rỗng.
* Không tìm thấy AppProcess.csv.
* CSV lỗi định dạng.
* Không có quyền đọc process.
* Không lấy được ExecutablePath.
* File cài đặt không tồn tại.
* File cài đặt bị khóa.
* User không có quyền Administrator.
* Process không thể đóng.
* UninstallString không tồn tại.
* Uninstall thất bại.
* File `.msi` lỗi.
* Process cài đặt trả về ExitCode khác 0.

Không được để ứng dụng tự động thoát.

Mọi lỗi phải được ghi vào Log TextBox.

---

# 18. Kiến trúc code

Hãy tổ chức mã nguồn thành các hàm rõ ràng, ví dụ:

```powershell
Initialize-Application
Initialize-UI
Show-LoadingForm
Get-ResourceFiles
Get-ApplicationProcessMap
Get-RunningProcesses
Get-ProcessExecutablePath
Get-ApplicationStatus
Get-UninstallInformation
Start-ApplicationSetup
Start-ApplicationUninstall
Stop-ApplicationProcess
Write-Log
Refresh-ApplicationStatus
Update-ApplicationRow
Test-Administrator
```

Không viết toàn bộ logic vào một block code duy nhất.

Các hàm phải có comment giải thích những đoạn logic quan trọng.

---

# 19. Yêu cầu về tính ổn định

Đặc biệt chú ý:

* Không sử dụng vòng lặp vô hạn làm treo UI.
* Không gọi thao tác nặng trực tiếp trên UI thread.
* Không block Main Form trong quá trình kiểm tra process.
* Khi chạy Setup/Uninstall phải khóa button tương ứng trong thời gian thao tác để tránh người dùng click nhiều lần.
* Sau khi hoàn tất phải enable lại button.
* Không tạo nhiều process cài đặt trùng nhau do click liên tục.
* Phải dispose các Form/Control/Process không còn sử dụng.
* Không để các event handler bị đăng ký nhiều lần sau mỗi lần refresh.

---

# 20. Kết quả đầu ra cần cung cấp

Hãy cung cấp **mã nguồn PowerShell hoàn chỉnh**, có thể lưu trực tiếp thành:

```text
SoftwareManager.ps1
```

và chạy được.

Không chỉ cung cấp pseudocode hoặc code minh họa.

Mã nguồn phải bao gồm đầy đủ:

1. Import các assembly WinForms cần thiết.
2. Kiểm tra quyền Administrator.
3. Xác định thư mục Resource.
4. Loading Form + ProgressBar.
5. Quét Resource và Subfolder.
6. Phân tích STT.
7. Đọc AppProcess.csv.
8. Kiểm tra process đang chạy.
9. Xác định trạng thái.
10. Tạo Main Form.
11. Tạo danh sách phần mềm động.
12. Button Setup.
13. Button Uninstall.
14. Tìm UninstallString/QuietUninstallString.
15. Stop process khi cần.
16. Log realtime.
17. Refresh trạng thái.
18. Xử lý exception.
19. Footer `R&D by KSS`.
20. Code comment rõ ràng.

---

# 21. Yêu cầu quan trọng về logic

Không được đơn giản hóa yêu cầu thành việc chỉ kiểm tra xem **tên file cài đặt có tồn tại hay không**.

Ví dụ:

```text
Resource\8.ZaloSetup-25.4.2.exe
```

chỉ là **bộ cài đặt**, không có nghĩa Zalo đã được cài.

Trạng thái phải được xác định dựa trên:

```text
Process đang chạy
        +
ExecutablePath
        +
AppProcess.csv
```

Đồng thời khi uninstall phải ưu tiên sử dụng thông tin uninstall thực tế của Windows Registry thay vì giả định rằng tên file trong Resource chính là file uninstall.

---

# 22. Kỳ vọng cuối cùng

Hãy xây dựng công cụ theo hướng đây là một **Software Deployment / Software Management Tool nội bộ**, có khả năng:

```text
Quét Resource
      ↓
Nhận diện phần mềm
      ↓
Kiểm tra process
      ↓
Đối chiếu AppProcess.csv
      ↓
Hiển thị trạng thái
      ↓
┌───────────────┐
│               │
▼               ▼
Chưa cài      Đang chạy
│               │
Setup         Uninstall
│               │
└───────┬───────┘
        ▼
Refresh trạng thái
```

Ưu tiên **tính ổn định, khả năng bảo trì, giao diện không bị treo và xử lý lỗi đầy đủ**.

Nếu có điểm nào trong yêu cầu chưa đủ thông tin, hãy **tự lựa chọn phương án kỹ thuật hợp lý nhất và ghi chú rõ trong comment của mã nguồn**, không được bỏ qua chức năng hoặc chỉ đưa ra pseudocode.
