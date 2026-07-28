# Yêu cầu chỉnh sửa mã nguồn PowerShell

Trong file **`getPCInfo-v5.4.ps1`** đã tải lên, hãy chỉnh sửa mã nguồn theo các yêu cầu dưới đây.

> **Quan trọng:** Giữ nguyên toàn bộ chức năng hiện có của chương trình. Chỉ bổ sung và sửa đổi các phần liên quan đến việc ghi dữ liệu vào **`AppInstalled.csv`**.

---

# Mục tiêu

Thay đổi cơ chế ghi dữ liệu của file **`AppInstalled.csv`** nhằm chỉ ghi nhận các phần mềm có thay đổi thông tin, tránh ghi lặp toàn bộ danh sách phần mềm sau mỗi lần script được thực thi.

---

# 1. Thay đổi cấu trúc file AppInstalled.csv

Bổ sung **02 cột mới** đặt **trước cột `AppName`** theo đúng thứ tự sau:

| STT | Tên cột |
|-----|----------|
|1|Ngày cập nhật|
|2|HostName|
|3|AppName|
|4|Version|
|5|Publisher|
|6|InstallDate|
|7|Architecture|
|8|UninstallString|
|9|UninstallSilent|
|10|EstimatedSize|

Trong đó:

### Cột "Ngày cập nhật"

Lưu thời điểm dữ liệu được ghi vào file với định dạng:

```text
yyyy-MM-dd HH:mm
```

Ví dụ:

```text
2026-07-28 09:45
```

### Cột "HostName"

Lưu tên máy tính đang thực thi script `getPCInfo-v5.4.ps1`.

---

# 2. Thay đổi cơ chế ghi dữ liệu

Trước khi ghi dữ liệu vào **`AppInstalled.csv`**, chương trình phải thực hiện các bước sau.

## Bước 1

Đọc toàn bộ dữ liệu phần mềm đang cài đặt trên máy tính hiện tại.

Các trường dùng để so sánh gồm:

- AppName
- Version
- Publisher
- InstallDate
- Architecture

---

## Bước 2

Đọc toàn bộ dữ liệu hiện có trong `AppInstalled.csv`.

Chỉ so sánh các bản ghi có cùng:

- HostName
- AppName

---

## Bước 3

Đối với từng **AppName** của máy đang kiểm tra:

- Nếu chưa tồn tại trong `AppInstalled.csv` → ghi mới.
- Nếu đã tồn tại thì so sánh:
  - Version
  - Publisher
  - InstallDate
  - Architecture

Nếu có ít nhất một trường thay đổi:

- append thêm một dòng mới;
- cập nhật **Ngày cập nhật** theo định dạng `yyyy-MM-dd HH:mm`;
- ghi **HostName** của máy hiện tại.

Nếu không có thay đổi:

- không ghi dữ liệu.

---

# Các trường KHÔNG dùng để xác định thay đổi

- UninstallString
- UninstallSilent
- EstimatedSize

Các trường này chỉ ghi kèm khi phát sinh bản ghi mới hoặc thay đổi.

---

# Thuật toán

```text
Lấy danh sách phần mềm

↓

Đọc AppInstalled.csv

↓

So sánh theo

HostName + AppName

↓

Nếu chưa tồn tại
    → Ghi mới

Nếu tồn tại

    So sánh

    Version
    Publisher
    InstallDate
    Architecture

    Nếu khác
        → Append
        → Cập nhật Ngày cập nhật

    Nếu giống
        → Bỏ qua
```

---

# Yêu cầu hiệu năng

- Chỉ đọc CSV một lần.
- Sử dụng Dictionary hoặc Hashtable để tra cứu.
- Không đọc lại file cho từng phần mềm.
- Chỉ append các bản ghi mới hoặc thay đổi.

---

# Yêu cầu mã nguồn

- Giữ nguyên toàn bộ chức năng hiện có.
- Không ảnh hưởng đến `PCInfo.csv`.
- Tương thích Windows PowerShell 5.1.
- Không dùng module ngoài.
- Chỉ dùng thư viện .NET Framework có sẵn.

---

# Kết quả mong muốn

1. Trả về toàn bộ file `getPCInfo-v5.4.ps1` đã chỉnh sửa.
2. Giải thích các thay đổi.
3. Liệt kê các hàm được sửa hoặc bổ sung.
4. Đảm bảo tương thích môi trường GPO/Logon Script.
