# Case Study 01 – Windows Endpoint Management

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Phase:** 01 – Endpoint Deployment & Administration  
**Platform:** VMware Workstation, Windows 11 Pro, Active Directory, Group Policy, PowerShell, Winget

---

## 1. Project Challenge

Trong môi trường doanh nghiệp, bộ phận IT chịu trách nhiệm triển khai và duy trì các máy tính phục vụ người dùng cuối.

Một máy trạm sau khi cài đặt Windows cần được kết nối vào mạng nội bộ, tích hợp với hệ thống quản lý danh tính tập trung, nhận các chính sách bảo mật và có khả năng truy cập tài nguyên doanh nghiệp.

Nếu không có quy trình quản lý thống nhất, việc vận hành máy trạm có thể gặp các vấn đề:

- Tài khoản người dùng được quản lý phân tán trên từng thiết bị.
- Cấu hình bảo mật giữa các máy không đồng nhất.
- Khó triển khai và cập nhật phần mềm.
- Khó kiểm tra nguyên nhân khi người dùng không truy cập được tài nguyên nội bộ.
- Khó tích hợp thiết bị vào các hệ thống Asset Management và Monitoring.

Để mô phỏng quy trình quản trị Endpoint, Phase 1 triển khai Windows 11 Pro trong môi trường Active Directory đã được xây dựng từ Project 1.

### Objectives

- Triển khai Windows 11 Pro trên VMware Workstation.
- Kết nối máy trạm vào mạng USER.
- Gia nhập Active Directory Domain `corp.local`.
- Xác minh Domain Authentication và Group Policy.
- Triển khai một số phần mềm thông dụng bằng Winget.
- Kiểm tra kết nối DNS, SMB và các dịch vụ nội bộ.
- Chuẩn bị máy trạm cho các Phase ITSM, Asset Management, VPN và Monitoring.

---

## 2. Technical Concepts

### 2.1. Windows Endpoint Management

**Endpoint Management** là quá trình triển khai, cấu hình, quản lý và bảo trì các thiết bị đầu cuối được sử dụng trong hệ thống CNTT.

Đối với môi trường Windows Domain, công tác quản lý Endpoint bao gồm:

- Operating System Deployment.
- User & Device Identity Management.
- Configuration & Policy Enforcement.
- Software Deployment.
- Security Configuration.
- Connectivity Troubleshooting.
- Device Inventory & Monitoring.

Trong Project 2, máy trạm WIN11-01 được sử dụng làm Endpoint chính để kiểm thử các quy trình trên.

### 2.2. Active Directory Domain Services

**Active Directory Domain Services (AD DS)** là dịch vụ thư mục cho phép quản lý tập trung các đối tượng như User, Computer, Group và Organizational Unit.

Việc đưa máy trạm vào Domain giúp:

- Sử dụng tài khoản Domain để đăng nhập.
- Quản lý Computer Object trong Active Directory.
- Áp dụng Group Policy.
- Sử dụng các dịch vụ hỗ trợ xác thực trong môi trường Domain.
- Truy cập tài nguyên nội bộ theo quyền được cấp.

Trong hệ thống:

| Thành phần | Giá trị |
|---|---|
| Active Directory Domain | `corp.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `10.10.10.10` |
| Domain DNS Server | `10.10.10.10` |
| Domain-Joined Endpoint | `WIN11-01` |

### 2.3. DNS trong môi trường Active Directory

DNS có vai trò quan trọng trong việc tìm Domain Controller và phân giải tên các dịch vụ nội bộ.

Windows sử dụng các bản ghi DNS, bao gồm SRV Records, để xác định dịch vụ phù hợp khi cần liên hệ với Domain Controller.

Ví dụ:

`dc01.corp.local` → `10.10.10.10`

`fs01.corp.local` → `10.10.10.20`

Nếu máy trạm sử dụng DNS không phù hợp, quá trình Domain Join, tìm Domain Controller hoặc áp dụng Group Policy có thể gặp lỗi.

Do đó, việc xác minh DNS luôn là một bước quan trọng trong Endpoint Troubleshooting.

### 2.4. Group Policy

**Group Policy Object (GPO)** là cơ chế quản lý cấu hình tập trung trong Active Directory.

GPO có thể áp dụng cho User hoặc Computer tùy theo phạm vi liên kết, Security Filtering và các điều kiện xử lý chính sách.

Một số mục đích sử dụng:

- Cấu hình chính sách bảo mật máy trạm.
- Quản lý các thiết lập hệ thống.
- Hạn chế những chức năng không phù hợp với người dùng.
- Chuẩn hóa môi trường làm việc.
- Hỗ trợ quản trị tập trung nhiều Endpoint.

Trong Project 2, WIN11-01 nhận chính sách `GPO_Workstation_Baseline`.

### 2.5. Windows Package Manager – Winget

**Winget** là công cụ quản lý gói phần mềm trên Windows, hỗ trợ tìm kiếm, cài đặt và nâng cấp ứng dụng bằng dòng lệnh.

Công cụ này giúp kỹ thuật viên IT giảm các thao tác cài đặt thủ công.

Trong Phase 1, Winget được sử dụng để cài đặt:

- Google Chrome.
- 7-Zip.

Đồng thời, quá trình thực hiện đã phát sinh một vấn đề liên quan đến việc sử dụng Winget giữa các tài khoản Windows khác nhau. Sự cố được phân tích trong phần Troubleshooting.

---

## 3. Solution Architecture

### 3.1. Endpoint Deployment Architecture

Máy WIN11-01 được triển khai dưới dạng Virtual Machine trong VMware Workstation.

Endpoint nằm ở mạng USER và sử dụng pfSense làm Default Gateway để truy cập những dịch vụ nằm trong SERVER Network.

```text
                  VMware Workstation
                          |
                    USER Network
                    10.10.20.0/24
                          |
                    +-----------+
                    | WIN11-01  |
                    | Win 11 Pro|
                    | .20.104   |
                    +-----------+
                          |
                          |
                    +-----------+
                    | pfSense   |
                    | Gateway   |
                    | .20.1     |
                    +-----------+
                          |
                    SERVER Network
                    10.10.10.0/24
                          |
               +----------+----------+
               |                     |
          +---------+           +---------+
          | DC01    |           | FS01    |
          | AD/DNS  |           | File    |
          | .10.10  |           | .10.20  |
          +---------+           +---------+
```

Các địa chỉ rút gọn trong sơ đồ thuộc các subnet tương ứng.

### 3.2. Endpoint Configuration

| Property | Configuration |
|---|---|
| Hostname | `WIN11-01` |
| Operating System | Windows 11 Pro |
| Virtualization | VMware Workstation |
| Network Adapter | VMnet4 |
| IPv4 Address | `10.10.20.104` |
| Subnet | `10.10.20.0/24` |
| Default Gateway | `10.10.20.1` |
| DNS Server | `10.10.10.10` |
| Domain | `corp.local` |
| Computer OU | `CORP_Computers` |
| Applied GPO | `GPO_Workstation_Baseline` |

### 3.3. Design Decisions

| Quyết định | Lý do |
|---|---|
| Sử dụng Windows 11 Pro | Hỗ trợ Domain Join và Remote Desktop Host |
| Đặt Endpoint trong mạng USER | Phân tách Workstation khỏi Server Infrastructure |
| Sử dụng DNS nội bộ | Hỗ trợ Active Directory Domain Discovery |
| Tái sử dụng Domain `corp.local` | Tập trung vào Endpoint Operations thay vì triển khai lại AD |
| Áp dụng Workstation Baseline GPO | Chuẩn hóa chính sách quản trị máy trạm |
| Sử dụng Winget | Thực hành Software Deployment bằng dòng lệnh |
| Dùng PowerShell để xác minh cấu hình | Tạo kết quả kiểm thử có thể lặp lại |

---

## 4. Implementation

### 4.1. Windows Endpoint Deployment

Windows 11 Pro được triển khai dưới dạng máy ảo trên VMware Workstation.

Máy được kết nối tới mạng USER thông qua VMnet4.

Sau khi cài đặt Windows, thực hiện kiểm tra:

```powershell
hostname
```

```powershell
Get-CimInstance Win32_OperatingSystem |
Select-Object Caption, Version, BuildNumber
```

```powershell
Get-NetIPConfiguration
```

Mục đích là xác nhận danh tính máy, hệ điều hành và cấu hình mạng trước khi tích hợp với Domain.

### 4.2. Active Directory Domain Integration

WIN11-01 được gia nhập Domain:

`corp.local`

Sau khi Domain Join, máy được xác nhận thuộc môi trường Active Directory.

Kiểm tra bằng PowerShell:

```powershell
Get-CimInstance Win32_ComputerSystem |
Select-Object Name, Domain, PartOfDomain
```

Kết quả đã xác minh:

- Computer Name: `WIN11-01`.
- Domain: `corp.local`.
- PartOfDomain: `True`.

Kiểm tra Domain Controller Discovery:

```powershell
nltest /dsgetdc:corp.local
```

Kết quả cho thấy WIN11-01 có thể xác định Domain Controller `DC01.corp.local`.

### 4.3. Computer Object Management

Trong Active Directory, máy WIN11-01 được quản lý tại Organizational Unit:

`OU=CORP_Computers,DC=corp,DC=local`

OU cho phép tổ chức các đối tượng và hỗ trợ xác định phạm vi áp dụng Group Policy.

Việc đặt Workstation vào đúng OU giúp thuận tiện cho quản lý tập trung và phân biệt với những nhóm thiết bị khác.

### 4.4. Group Policy Validation

Sau khi máy gia nhập Domain, tiến hành kiểm tra Group Policy:

```powershell
gpupdate /force
```

Lệnh trên yêu cầu Windows cập nhật Group Policy.

Sau đó sử dụng:

```powershell
gpresult /r
```

Hoặc:

```powershell
gpresult /scope computer /r
```

Trong quá trình kiểm thử, WIN11-01 nhận được các chính sách:

- `Default Domain Policy`.
- `GPO_Workstation_Baseline`.

Điều này xác nhận quá trình xử lý Group Policy trên Workstation hoạt động và máy nhận được GPO Baseline đã cấu hình.

### 4.5. Software Deployment Using Winget

Phase 1 sử dụng Winget để cài đặt các phần mềm cơ bản trên Endpoint.

Kiểm tra Winget:

```powershell
winget --version
```

Cập nhật package sources:

```powershell
winget source update
```

Cài đặt Google Chrome:

```powershell
winget install --id Google.Chrome -e --source winget
```

Cài đặt 7-Zip:

```powershell
winget install --id 7zip.7zip -e --source winget
```

Kiểm tra phần mềm:

```powershell
winget list
```

Trong lab, Google Chrome và 7-Zip đã được cài đặt và kiểm tra đường dẫn thực thi thành công.

Cách tiếp cận này minh họa khả năng triển khai phần mềm thông qua Command-Line Package Management.

Đây là thao tác triển khai phần mềm trên một Endpoint, chưa phải hệ thống Software Deployment tự động ở quy mô nhiều máy tính.

### 4.6. DNS & File Server Connectivity

Kiểm tra Domain DNS:

```powershell
Resolve-DnsName dc01.corp.local
```

Kiểm tra File Server DNS:

```powershell
Resolve-DnsName fs01.corp.local
```

Địa chỉ được xác minh:

`fs01.corp.local → 10.10.10.20`

Kiểm tra TCP Connectivity:

```powershell
Test-NetConnection fs01.corp.local -Port 445
```

Kết quả thực tế:

```text
RemoteAddress    : 10.10.10.20
RemotePort       : 445
TcpTestSucceeded : True
```

Kiểm tra đường dẫn File Share:

```powershell
Test-Path "\\fs01.corp.local\CORP-SHARE"
```

Kết quả:

```text
True
```

Kết quả chứng minh WIN11-01 có thể truy cập đường dẫn SMB Share từ mạng USER đến FS01 thuộc SERVER Network, trong ngữ cảnh tài khoản đang thực hiện kiểm tra.

---

## 5. Troubleshooting – Winget Permission & Package Registration

### 5.1. Problem Description

Trong quá trình triển khai phần mềm, Winget hoạt động với tài khoản Domain User nhưng gặp lỗi khi sử dụng dưới tài khoản Domain Administrator.

Các hiện tượng đã ghi nhận gồm:

- Lệnh `winget` không được nhận diện.
- Lỗi quyền truy cập `0x80070005`.
- Lỗi mở Package Source `0x8a15000f`.

Vấn đề này cho thấy quyền Administrator không đồng nghĩa mọi ứng dụng dạng MSIX/AppX đều được đăng ký và hoạt động trong từng User Profile.

### 5.2. Root Cause Analysis

Winget được phân phối thông qua Windows App Installer.

Các Package AppX/MSIX có thể có trạng thái đăng ký khác nhau giữa các tài khoản Windows.

Vì vậy, một tài khoản có thể sử dụng Winget bình thường, trong khi tài khoản khác gặp vấn đề về Package Registration hoặc Source Configuration.

Trong lab, vấn đề được xử lý bằng cách đăng ký lại những package cần thiết và cập nhật Winget Sources.

### 5.3. Resolution

Các thành phần được kiểm tra và đăng ký lại:

- `Microsoft.DesktopAppInstaller`.
- `Microsoft.Winget.Source`.

Sau đó thực hiện kiểm tra:

```powershell
winget --version
```

```powershell
winget source list
```

```powershell
winget source update
```

Việc đăng ký lại package và cập nhật sources đã khắc phục sự cố trong môi trường lab.

### 5.4. Verification

Kết quả sau khi xử lý:

- `winget --version` trả về `v1.11.510`.
- Winget Sources gồm `msstore` và `winget`.
- Google Chrome được cài đặt thành công.
- 7-Zip được cài đặt thành công.
- Các đường dẫn phần mềm được kiểm tra.

### 5.5. Lessons Learned

Qua sự cố này, có thể rút ra các điểm kỹ thuật:

1. Quyền Administrator và trạng thái Package Registration là hai yếu tố khác nhau.
2. Khi một Command-Line Tool hoạt động với tài khoản này nhưng lỗi với tài khoản khác, cần kiểm tra User Context.
3. Việc xử lý lỗi Software Deployment không nên chỉ tập trung vào cài đặt lại phần mềm.
4. Package Sources và App Installer Registration là những thành phần cần kiểm tra khi Winget gặp lỗi.

---

## 6. Validation & Test Results

| Test ID | Test Case | Expected Result | Actual Result |
|---|---|---|---|
| END-01 | Windows 11 Deployment | OS hoạt động | PASS |
| END-02 | Network Connectivity | Có kết nối mạng USER | PASS |
| END-03 | Domain Membership | PartOfDomain = True | PASS |
| END-04 | DC Discovery | Tìm được DC01 | PASS |
| END-05 | Computer OU | Thuộc CORP_Computers | PASS |
| END-06 | Group Policy | Baseline GPO được áp dụng | PASS |
| END-07 | Winget Recovery | Winget hoạt động | PASS |
| END-08 | Chrome / 7-Zip Deployment | Cài đặt thành công | PASS |
| END-09 | DNS Resolution | Phân giải được FS01 | PASS |
| END-10 | SMB Connectivity | TCP 445 thành công | PASS |
| END-11 | File Share Accessibility | CORP-SHARE truy cập được | PASS |

### Evidence

Các bằng chứng dự kiến lưu tại:

`evidence/endpoint/`

| Evidence | Nội dung |
|---|---|
| `01-windows-system.png` | Windows 11 System Information |
| `02-domain-membership.png` | Domain Membership |
| `03-domain-controller-discovery.png` | Kết quả `nltest` |
| `04-group-policy.png` | `gpresult` |
| `05-winget-deployment.png` | Chrome và 7-Zip |
| `06-file-server-connectivity.png` | SMB TCP 445 và File Share |

Các tên file trên là quy ước đề xuất, cần đối chiếu với ảnh thực tế trước khi tạo liên kết trong Markdown.

---

## 7. Security Considerations

### Domain-Based Administration

Sử dụng Active Directory để hỗ trợ quản lý danh tính tập trung thay vì chỉ sử dụng Local Accounts.

### Group Policy Enforcement

Áp dụng GPO Baseline nhằm duy trì chính sách cấu hình thống nhất trên Workstation.

### Network Segmentation

WIN11-01 được đặt tại USER Network, tách biệt với các Server Infrastructure.

### Least Privilege

Các thao tác quản trị và cài đặt phần mềm cần được thực hiện bằng tài khoản có quyền phù hợp với nhiệm vụ.

### Remote Desktop Access

Ở Phase 4, Windows Firewall của WIN11-01 được bổ sung giới hạn chỉ cho phép Remote Desktop từ VPN Subnet.

Chính sách RDP không được xem là thành phần đã triển khai ngay từ đầu Phase 1, mà là phần tăng cường bảo mật được thực hiện sau này.

---

## 8. Technical Outcomes

Sau Phase 1, hệ thống có một Windows 11 Workstation được quản lý trong Active Directory và có thể truy cập các dịch vụ nội bộ cần thiết.

Các kết quả chính:

- Hoàn thành Domain Join.
- Xác minh Domain Controller Discovery.
- Áp dụng Workstation Baseline GPO.
- Cài đặt phần mềm qua Winget.
- Khắc phục lỗi Winget theo User Context.
- Xác minh DNS và File Server Connectivity.
- Chuẩn bị Endpoint cho các hoạt động ITSM, Inventory, Remote Support và Monitoring.

### Skills Demonstrated

- Windows Endpoint Administration.
- Active Directory Domain Integration.
- Group Policy Verification.
- DNS & SMB Troubleshooting.
- PowerShell Administration.
- Package Management with Winget.
- Network Connectivity Validation.
- Root Cause Analysis.

---

## 9. Scope & Limitations

Case Study này tập trung vào một máy trạm Windows 11 Pro trong môi trường lab.

Những hạng mục chưa nằm trong phạm vi:

- Triển khai Windows tự động hàng loạt bằng MDT, WDS hoặc Autopilot.
- Quản lý Endpoint qua Microsoft Intune.
- Software Deployment tập trung cho nhiều Workstations.
- Windows Update Management tập trung.
- Enterprise EDR hoặc Centralized Endpoint Security Management.

Do đó, kết quả Phase 1 phản ánh năng lực triển khai và quản trị một Domain-Joined Workstation, không đại diện cho hệ thống Endpoint Management triển khai ở quy mô doanh nghiệp lớn.

---

## 10. Conclusion

Phase 1 đã xây dựng thành công Endpoint nền tảng cho toàn bộ Project 2.

WIN11-01 được tích hợp vào môi trường Active Directory, nhận GPO, triển khai phần mềm bằng Winget và truy cập các dịch vụ nội bộ thông qua hạ tầng mạng phân đoạn.

Ngoài các thao tác cấu hình, việc xử lý lỗi Winget giữa những tài khoản Windows khác nhau cung cấp một tình huống Troubleshooting thực tế liên quan đến User Context, Package Registration và Software Deployment.

Máy trạm WIN11-01 tiếp tục được sử dụng trong Phase 2 để mô phỏng hoạt động IT Helpdesk và Incident Management bằng GLPI.