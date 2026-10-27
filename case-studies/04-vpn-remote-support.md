# Case Study 04 – Secure Remote IT Support

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Phase:** 04 – VPN & Remote Technical Support  
**Technologies:** pfSense CE, OpenVPN, Windows 11 Pro, Remote Desktop Protocol, Windows Firewall, PowerShell

---

## 1. Project Challenge

Trong môi trường doanh nghiệp, bộ phận IT thường cần truy cập máy tính của nhân viên để kiểm tra, chẩn đoán và xử lý sự cố từ xa.

Tuy nhiên, việc cho phép truy cập Remote Desktop trực tiếp từ Internet có thể làm gia tăng rủi ro bảo mật, đặc biệt nếu dịch vụ RDP được công khai mà không có các biện pháp kiểm soát truy cập phù hợp.

Những yêu cầu đặt ra:

- Kỹ thuật viên có thể kết nối từ xa đến mạng nội bộ.
- Không công khai trực tiếp TCP 3389 của máy trạm ra WAN.
- Yêu cầu xác thực trước khi truy cập tài nguyên nội bộ.
- Chỉ cho phép kết nối đến những dịch vụ phục vụ công việc hỗ trợ.
- Giới hạn phạm vi truy cập bằng Firewall Rules.
- Có khả năng kiểm tra và ghi nhận kết quả phiên hỗ trợ.

Phase 4 giải quyết bài toán bằng cách triển khai **OpenVPN trên pfSense**, sau đó cho phép kỹ thuật viên sử dụng Remote Desktop để truy cập WIN11-01 thông qua VPN Tunnel.

### Objectives

- Triển khai OpenVPN Remote Access trên pfSense.
- Cấu hình xác thực bằng tài khoản và Certificate.
- Cấp phát địa chỉ IP trong VPN Tunnel Network.
- Thiết lập định tuyến Split Tunneling.
- Cấu hình Firewall Rules giới hạn truy cập.
- Cho phép RDP từ VPN Subnet đến WIN11-01.
- Kích hoạt Network Level Authentication.
- Giới hạn Windows Firewall RDP chỉ cho phép VPN Subnet.
- Kiểm thử kết nối VPN và Remote Desktop.
- Ghi nhận hoạt động hỗ trợ trong GLPI.

---

## 2. Technical Concepts

### 2.1. Remote IT Support

Remote IT Support là hình thức hỗ trợ kỹ thuật mà nhân viên IT có thể truy cập hoặc điều khiển thiết bị từ một vị trí khác.

Các hoạt động phổ biến:

- Chẩn đoán sự cố hệ điều hành.
- Kiểm tra Network Configuration.
- Xử lý lỗi phần mềm.
- Cấu hình tài khoản và quyền truy cập.
- Kiểm tra dịch vụ đang hoạt động.
- Hỗ trợ người dùng mà không cần tiếp xúc trực tiếp với thiết bị.

Trong Project 2, Remote Support được thực hiện qua Windows Remote Desktop Protocol.

### 2.2. Virtual Private Network

**VPN (Virtual Private Network)** cho phép thiết lập một kết nối logic qua mạng trung gian để trao đổi dữ liệu với mạng đích.

Một Remote Access VPN thường cung cấp:

- Mã hóa lưu lượng qua VPN Tunnel.
- Xác thực danh tính trước khi thiết lập phiên VPN.
- Cấp phát địa chỉ mạng cho Client.
- Khả năng định tuyến đến các mạng nội bộ được cho phép.

VPN không tự động cấp toàn quyền truy cập tất cả tài nguyên nội bộ. Phạm vi truy cập vẫn phụ thuộc vào Routing, Firewall và chính sách của dịch vụ đích.

### 2.3. OpenVPN

OpenVPN là giải pháp VPN sử dụng TLS để hỗ trợ xác thực và bảo vệ kết nối.

Các thành phần chính trong mô hình:

| Thành phần | Chức năng |
|---|---|
| OpenVPN Server | Tiếp nhận và xác thực kết nối VPN |
| OpenVPN Client | Thiết lập kết nối từ máy kỹ thuật viên |
| Certificate | Tham gia quá trình xác thực TLS |
| User Credentials | Xác thực tài khoản người dùng |
| Tunnel Network | Cấp phát IP cho Client |
| Firewall Rules | Kiểm soát lưu lượng sau khi VPN kết nối |

Trong Project 2, OpenVPN Server được triển khai trực tiếp trên pfSense.

### 2.4. Certificate-Based Authentication

Certificate là chứng thư số chứa thông tin khóa công khai và các thuộc tính định danh được xác nhận bằng chữ ký số.

Trong mô hình VPN, Certificate có thể được sử dụng để xác thực Server, Client hoặc cả hai, tùy cấu hình.

Hệ thống lab sử dụng cơ chế kết hợp:

- Xác thực người dùng.
- Xác thực bằng Certificate.

Cơ chế này bổ sung một lớp kiểm soát so với việc chỉ sử dụng tài khoản và mật khẩu.

Certificate và Private Key phải được bảo vệ, không đưa lên GitHub.

### 2.5. Split Tunneling

**Split Tunneling** là cơ chế định tuyến chỉ một phần lưu lượng mạng qua VPN, trong khi các lưu lượng khác vẫn sử dụng đường truyền mạng thông thường.

Ví dụ:

```text
Traffic to Internal Network
          |
          v
      VPN Tunnel

Traffic to Other Networks
          |
          v
    Local Network Route
```

Ưu điểm:

- Giảm lượng dữ liệu phải truyền qua VPN.
- Phù hợp với nhu cầu truy cập một số subnet hoặc dịch vụ nội bộ.
- Không bắt buộc toàn bộ lưu lượng Internet đi qua VPN Server.

Tuy nhiên, Split Tunneling cần được thiết kế cẩn thận vì máy Client đồng thời có kết nối với mạng nội bộ VPN và các mạng bên ngoài.

Trong Project 2, Split Tunneling được sử dụng để hỗ trợ truy cập đến những địa chỉ mạng phục vụ công việc Remote Support.

### 2.6. Remote Desktop Protocol

**RDP (Remote Desktop Protocol)** là giao thức cho phép truy cập phiên Desktop từ xa trên Windows.

Trong cấu hình phổ biến, Windows Remote Desktop sử dụng:

- TCP 3389.
- UDP 3389 khi được hỗ trợ và cho phép.

Windows 11 Pro có khả năng hoạt động như Remote Desktop Host.

Trong Project 2, RDP được triển khai trên WIN11-01 và chỉ được cho phép truy cập từ VPN Subnet.

### 2.7. Network Level Authentication

**Network Level Authentication (NLA)** yêu cầu xác thực trước khi thiết lập đầy đủ phiên Remote Desktop.

NLA giúp giảm phạm vi tài nguyên của Remote Desktop cần được cung cấp trước quá trình xác thực.

NLA là một biện pháp bảo vệ bổ sung, không thay thế VPN, Firewall hoặc chính sách quản lý tài khoản.

---

## 3. Solution Architecture

### 3.1. Remote Access Architecture

Máy Windows Host đóng vai trò Remote IT Technician và chạy OpenVPN Client.

OpenVPN Server được triển khai trên pfSense.

Sau khi kết nối VPN thành công, Client nhận địa chỉ IP thuộc VPN Tunnel Network `10.10.40.0/24`.

Kỹ thuật viên có thể truy cập WIN11-01 theo chính sách Firewall đã thiết lập.

```text
             WINDOWS PHYSICAL HOST
               Remote Technician
                       |
                 OpenVPN Client
                       |
                VMware NAT Network
                192.168.10.0/24
                       |
                       | UDP 1194
                       v
              +------------------+
              |     pfSense      |
              |   OpenVPN Server |
              | 192.168.10.129   |
              +------------------+
                       |
                 VPN Tunnel
                 10.10.40.0/24
                       |
                       | Firewall Rules
                       v
                  USER Network
                  10.10.20.0/24
                       |
                +--------------+
                |   WIN11-01   |
                | Windows 11   |
                | 10.10.20.104 |
                | RDP TCP 3389 |
                +--------------+
```

### 3.2. Network Configuration

| Component | Configuration |
|---|---|
| VPN Server | pfSense CE 2.5.2 |
| pfSense WAN IP | `192.168.10.129` |
| VMware NAT Gateway | `192.168.10.2` |
| VPN Protocol | UDP |
| VPN Server Port | 1194 |
| VPN Tunnel Network | `10.10.40.0/24` |
| VPN Client IP (verified session) | `10.10.40.2` |
| Target Workstation | WIN11-01 |
| Target IP | `10.10.20.104` |
| RDP Port | TCP 3389 |
| Domain | `corp.local` |

Địa chỉ `192.168.10.129` là địa chỉ WAN của pfSense trong VMware NAT Network, không phải IP công cộng được định tuyến trực tiếp trên Internet.

### 3.3. Design Decisions

| Quyết định | Lý do |
|---|---|
| Triển khai VPN trước RDP | Không mở trực tiếp RDP ra WAN |
| Sử dụng OpenVPN trên pfSense | Tập trung Firewall và Remote Access |
| Kết hợp Account và Certificate | Bổ sung kiểm soát xác thực |
| VPN Subnet riêng | Dễ quản lý và áp dụng Firewall Rules |
| Split Tunneling | Chỉ định tuyến các mạng cần truy cập qua VPN |
| Firewall theo Source và Destination | Giới hạn quyền truy cập |
| Bật NLA | Tăng cường bảo vệ phiên RDP |
| Giới hạn Windows Firewall | Bổ sung bảo vệ ngay tại Endpoint |
| Ghi nhận trong GLPI | Liên kết hoạt động Remote Support với ITSM |

---

## 4. Implementation

### 4.1. OpenVPN Server Deployment

OpenVPN Server được cấu hình trên pfSense.

Các thành phần đã triển khai gồm:

- OpenVPN Server Listener.
- Authentication Configuration.
- Certificate-Based Authentication.
- VPN Tunnel Network.
- Firewall Rules.
- Client Configuration.

Thông tin chính:

```text
Server Address : 192.168.10.129
Protocol       : UDP
Port           : 1194
Tunnel Network : 10.10.40.0/24
```

OpenVPN Client được cài đặt trên Windows Host.

Sau khi cung cấp thông tin xác thực phù hợp, Client kết nối thành công đến pfSense và nhận địa chỉ thuộc VPN Tunnel Network.

### 4.2. VPN Client Verification

Trong bài kiểm thử, OpenVPN Client nhận địa chỉ:

`10.10.40.2`

Trên Windows Host, sử dụng PowerShell để kiểm tra:

```powershell
Get-NetIPAddress -AddressFamily IPv4 |
Where-Object {
    $_.IPAddress -like "10.10.40.*"
} |
Select-Object InterfaceAlias, IPAddress
```

Kết quả xác minh cho thấy địa chỉ VPN được gán cho interface OpenVPN TAP-Windows6.

Thông tin này chứng minh VPN Client đã có địa chỉ IPv4 trên Tunnel Interface.

Tuy nhiên, để xác nhận kết nối thực sự phục vụ Remote Support, cần tiếp tục kiểm tra Routing và TCP Connectivity đến máy đích.

### 4.3. Split Tunnel Routing

VPN Client được cấu hình các tuyến đến mạng nội bộ phục vụ Remote Support.

Một tuyến đã được sử dụng:

```text
Destination: 10.10.20.0/24
Next Hop   : 10.10.40.1
```

Ngoài ra, hệ thống có tuyến đến Domain Controller nhằm phục vụ những hoạt động liên quan đến dịch vụ nội bộ.

Có thể kiểm tra bằng PowerShell:

```powershell
route print -4
```

Hoặc:

```powershell
Get-NetRoute -AddressFamily IPv4 |
Where-Object {
    $_.DestinationPrefix -like "10.10.*"
}
```

Việc kiểm tra Routing Table giúp xác định lưu lượng đến USER Network được đưa qua VPN Tunnel thay vì đi trực tiếp qua mạng cục bộ.

### 4.4. pfSense WAN Firewall

pfSense được cấu hình cho phép OpenVPN Server tiếp nhận kết nối trên WAN.

Rule chính:

| Parameter | Value |
|---|---|
| Interface | WAN |
| Action | Pass |
| Address Family | IPv4 |
| Protocol | UDP |
| Source | Any |
| Destination | WAN Address |
| Destination Port | 1194 |

Rule này cho phép kết nối tới dịch vụ OpenVPN.

Không có NAT Port Forward trực tiếp từ WAN đến RDP TCP 3389 trong kết quả kiểm tra cấu hình.

Điều này giảm nguy cơ công khai trực tiếp Remote Desktop trên WAN.

### 4.5. OpenVPN Firewall Rules

Sau khi VPN Client kết nối, lưu lượng từ VPN Tunnel được kiểm soát bằng các rule trên OpenVPN Interface của pfSense.

Các rule chính:

**Rule 1 – VPN to WIN11 RDP**

| Property | Value |
|---|---|
| Action | Pass |
| Interface | OpenVPN |
| Protocol | TCP |
| Source | `10.10.40.0/24` |
| Destination | `10.10.20.104` |
| Port | 3389 |
| Description | `VPN_TO_WIN11_RDP` |

Rule này cho phép VPN Client truy cập RDP đến WIN11-01, thay vì cấp quyền truy cập không giới hạn toàn bộ USER Network.

**Rule 2 – VPN to DC01 DNS TCP**

| Property | Value |
|---|---|
| Interface | OpenVPN |
| Protocol | TCP |
| Source | `10.10.40.0/24` |
| Destination | `10.10.10.10` |
| Port | 53 |
| Description | `VPN_TO_DC01_DNS_TCP` |

**Rule 3 – VPN to DC01 DNS UDP**

| Property | Value |
|---|---|
| Interface | OpenVPN |
| Protocol | UDP |
| Source | `10.10.40.0/24` |
| Destination | `10.10.10.10` |
| Port | 53 |
| Description | `VPN_TO_DC01_DNS_UDP` |

Trong giai đoạn Security Review, hai rule DNS từng được phát hiện cấu hình trùng giao thức TCP/UDP. Việc tách rõ TCP và UDP là cấu hình mục tiêu được đề xuất để loại bỏ sự trùng lặp.

**Trạng thái xác minh:** Rule RDP đã được kiểm tra từ ảnh cấu hình và kết nối thực tế. Việc chỉnh sửa cuối cùng hai DNS rule thành các giao thức riêng biệt chưa có output xác nhận sau khi Save/Apply.

### 4.6. Windows Remote Desktop Configuration

WIN11-01 được cấu hình cho phép Remote Desktop.

Dịch vụ Remote Desktop Services được xác minh đang hoạt động.

Kiểm tra trên Windows:

```powershell
Get-Service TermService
```

Kiểm tra TCP Listener:

```powershell
Get-NetTCPConnection -LocalPort 3389 -State Listen
```

Trong quá trình triển khai, Windows RDP được xác nhận lắng nghe trên cổng TCP 3389.

### 4.7. Windows Firewall Hardening

Để tránh cho phép kết nối RDP từ mọi nguồn, Windows Firewall được giới hạn về VPN Subnet.

Kết quả PowerShell thực tế:

```text
Name                           Enabled  Action  RemoteAddress

RemoteDesktop-In-TCP-WSS        False    Allow   Any
RemoteDesktop-Shadow-In-TCP     False    Allow   Any
RemoteDesktop-UserMode-In-TCP   True     Allow   10.10.40.0/255.255.255.0
RemoteDesktop-UserMode-In-UDP   True     Allow   10.10.40.0/255.255.255.0
```

Hai rule Remote Desktop User Mode được bật và giới hạn nguồn thuộc:

`10.10.40.0/24`

Các rule WSS và Shadowing trong kết quả kiểm tra ở trạng thái Disabled.

**Ý nghĩa bảo mật:**

Ngay cả khi lưu lượng có thể đi qua pfSense, Windows Firewall vẫn đóng vai trò là lớp kiểm soát bổ sung tại Endpoint.

Đây là mô hình bảo vệ nhiều lớp (Defense in Depth).

### 4.8. Network Level Authentication

NLA được xác minh thông qua Windows Registry:

```powershell
Get-ItemProperty `
  -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
  -Name UserAuthentication |
Select-Object UserAuthentication
```

Kết quả thực tế:

```text
UserAuthentication
------------------
1
```

Giá trị `1` cho biết Network Level Authentication đã được bật.

### 4.9. Remote Desktop Connectivity Verification

Sau khi OpenVPN kết nối, Windows Host thực hiện kiểm tra:

```powershell
Test-NetConnection 10.10.20.104 -Port 3389 -InformationLevel Detailed
```

Kết quả thực tế:

```text
SourceAddress    : 10.10.40.2
InterfaceAlias   : OpenVPN TAP-Windows6
RemoteAddress    : 10.10.20.104
RemotePort       : 3389
TcpTestSucceeded : True
```

Điều này xác nhận kết nối TCP 3389 từ VPN Client đến WIN11-01 thành công qua Tunnel Interface.

Ngoài phép thử TCP, phiên Remote Desktop thực tế đã được thiết lập bằng tài khoản Domain được cấp quyền.

---

## 5. Troubleshooting – Firewall Rule Placement

### 5.1. Problem Description

Trong quá trình cấu hình Remote Access, có tình huống Firewall Rule được tạo hoặc kiểm tra tại interface không phù hợp với luồng lưu lượng VPN.

Điều này gây khó khăn khi xác định tại sao một kết nối được cho phép hoặc bị chặn.

### 5.2. Technical Analysis

Trong pfSense, Firewall Rules thường được đánh giá trên interface mà lưu lượng đi vào theo chính sách lọc tương ứng.

Đối với mô hình này:

- Kết nối OpenVPN từ Client đến Server đi vào qua WAN.
- Lưu lượng đã đi qua VPN Tunnel cần được kiểm soát bằng những rule áp dụng cho OpenVPN.
- Firewall Rules của Windows tiếp tục kiểm soát lưu lượng khi đến Endpoint.

Do đó, cần phân biệt:

**WAN Rule:** Cho phép thiết lập kết nối đến OpenVPN Server.

**OpenVPN Rule:** Cho phép những truy cập nội bộ cụ thể từ VPN Client.

**Windows Firewall Rule:** Quy định nguồn được phép truy cập RDP tại WIN11-01.

### 5.3. Resolution

Cấu hình được rà soát và tổ chức theo đúng vị trí:

- WAN: OpenVPN UDP 1194.
- OpenVPN: RDP TCP 3389 đến WIN11-01 và DNS đến DC01.
- Windows Firewall: chỉ cho phép RDP từ VPN Subnet.

Ngoài ra, NAT Port Forward được kiểm tra để xác nhận không tồn tại rule công khai trực tiếp RDP qua WAN.

### 5.4. Lessons Learned

Sự cố này giúp củng cố các kiến thức:

1. Cần hiểu luồng gói tin trước khi tạo Firewall Rule.
2. WAN Access và VPN Tunnel Access là hai lớp lưu lượng khác nhau.
3. Không nên sử dụng Any-to-Any Rules chỉ để xử lý nhanh lỗi kết nối.
4. Cần kiểm tra đồng thời Routing, Firewall và Endpoint Service.
5. Một kết nối TCP thành công cần được đối chiếu với đúng Source Interface và Source Address.

---

## 6. Security Validation

### 6.1. WAN Exposure

Qua kết quả kiểm tra pfSense:

- OpenVPN UDP 1194 được cho phép trên WAN.
- Không phát hiện NAT Port Forward trực tiếp tới TCP 3389.
- Không triển khai WAN RDP Port Forward trong kiến trúc.

### 6.2. VPN Access Control

Rule RDP giới hạn:

```text
Source      : 10.10.40.0/24
Destination : 10.10.20.104
Protocol    : TCP
Port        : 3389
```

Điều này phù hợp với nguyên tắc Least Privilege ở cấp Firewall.

### 6.3. Endpoint Access Control

Windows Firewall giới hạn RDP TCP và UDP về VPN Subnet.

NLA được bật.

### 6.4. Remaining Security Considerations

Các giới hạn bảo mật cần ghi nhận:

- pfSense CE 2.5.2 là phiên bản cũ.
- Chưa kiểm thử toàn diện tất cả Floating Rules, Interface Group Rules và IPv6 Policies.
- Chưa thực hiện kiểm thử xâm nhập hoặc đánh giá cấu hình bảo mật toàn diện.
- Chưa triển khai MFA cho OpenVPN trong phạm vi đã xác minh.
- Môi trường sử dụng VMware NAT thay vì WAN Internet công cộng.

Vì vậy, kết quả bảo mật phản ánh chính sách Remote Access đã kiểm thử trong lab, không phải chứng nhận hệ thống đạt mức Production Security.

---

## 7. Integration with GLPI Incident Management

Trong Phase 4, hoạt động Remote Support được ghi nhận bằng GLPI Ticket #6.

Quy trình thực hành:

```text
Support Requirement
        |
        v
GLPI Ticket
        |
        v
Connect OpenVPN
        |
        v
Verify VPN Routing
        |
        v
Connect RDP to WIN11-01
        |
        v
Perform Technical Checks
        |
        v
Document Support Activity
        |
        v
Record Resolution
```

Việc kết hợp GLPI với VPN/RDP giúp thể hiện quy trình hỗ trợ kỹ thuật có ghi nhận và truy vết công việc.

Đây là hoạt động quản lý Incident được thực hiện thủ công; chưa có cơ chế điều khiển RDP trực tiếp từ giao diện GLPI.

---

## 8. Validation & Test Results

| Test ID | Test Case | Result |
|---|---|---|
| VPN-01 | OpenVPN Server hoạt động | PASS |
| VPN-02 | VPN User Authentication | PASS |
| VPN-03 | Certificate-Based Connection | PASS |
| VPN-04 | VPN Tunnel IP Assignment | PASS |
| VPN-05 | Split Tunnel Route | PASS |
| VPN-06 | VPN-to-WIN11 TCP 3389 | PASS |
| VPN-07 | Remote Desktop Session | PASS |
| VPN-08 | Windows RDP TCP/UDP Firewall Scope | PASS |
| VPN-09 | RDP Network Level Authentication | PASS |
| VPN-10 | No RDP WAN Port Forward | PASS |
| VPN-11 | pfSense WAN/OpenVPN Rule Review | PASS – cơ bản |

---

## 9. Technical Outcomes

Phase 4 hoàn thành một quy trình hỗ trợ Windows Endpoint từ xa thông qua VPN.

Những kết quả chính:

- Triển khai OpenVPN Remote Access trên pfSense.
- Xác thực VPN Client.
- Cấu hình Split Tunnel Routing.
- Thiết lập Firewall Rules theo Source, Destination và Port.
- Cho phép Remote Desktop qua VPN.
- Giới hạn Windows Firewall về VPN Subnet.
- Kích hoạt Network Level Authentication.
- Xác minh kết nối RDP thành công.
- Ghi nhận hoạt động hỗ trợ trong GLPI.

### Skills Demonstrated

- Network Administration.
- pfSense Firewall Configuration.
- Remote Access VPN.
- OpenVPN Authentication.
- VPN Routing & Split Tunneling.
- Firewall Rule Analysis.
- Windows Remote Desktop Administration.
- Network Level Authentication.
- PowerShell Connectivity Testing.
- Defense in Depth.
- Technical Troubleshooting.

---

## 10. Scope & Limitations

Phase 4 tập trung vào Remote Support cho một Windows Endpoint trong môi trường lab.

Các hạng mục chưa triển khai hoặc chưa kiểm thử đầy đủ:

- Multi-Factor Authentication.
- VPN User Group Access Policies ở quy mô lớn.
- High Availability cho VPN Gateway.
- Centralized VPN Session Auditing.
- VPN Performance Benchmarking.
- Internet-facing Production Deployment.
- Penetration Testing.
- Comprehensive Firewall Policy Audit.

Những giới hạn này phản ánh phạm vi thực hành và không làm thay đổi kết quả kiểm thử kết nối VPN/RDP đã đạt được.

---

## 11. Conclusion

Phase 4 xây dựng được giải pháp Remote IT Support dựa trên OpenVPN và Windows Remote Desktop.

Kỹ thuật viên có thể xác thực vào VPN, nhận địa chỉ Tunnel, truy cập mạng USER qua các tuyến được cấu hình và thiết lập RDP đến WIN11-01.

Hệ thống sử dụng các lớp kiểm soát gồm pfSense Firewall, Windows Firewall và Network Level Authentication nhằm giới hạn quyền truy cập Remote Desktop.

Các kết quả `TcpTestSucceeded=True`, SourceAddress thuộc VPN Subnet và phiên Remote Desktop thực tế xác nhận khả năng hỗ trợ từ xa đã hoạt động trong môi trường lab.

Phase 4 cung cấp nền tảng Remote Support phục vụ hoạt động xử lý Incident và quản trị Endpoint trong Project 2. 