# Enterprise IT Support & Infrastructure Operations Lab

**Mô phỏng hạ tầng CNTT doanh nghiệp với Windows Server, Active Directory, pfSense, GLPI, OpenVPN và Zabbix**

## 1. Project Overview

Dự án xây dựng một môi trường IT doanh nghiệp mô phỏng trên VMware Workstation, tập trung vào các nghiệp vụ quản trị hạ tầng, hỗ trợ người dùng, quản lý tài sản CNTT, hỗ trợ từ xa và giám sát hệ thống.

Project kế thừa hạ tầng mạng, Active Directory, DNS và File Server từ Project 1, sau đó mở rộng để mô phỏng các hoạt động thường gặp của bộ phận IT Operations.

Hệ thống được tổ chức thành năm nhóm chức năng:

1. **Endpoint Management:** Triển khai và quản lý Windows Workstation trong môi trường Active Directory.
2. **IT Service Management:** Tiếp nhận và xử lý yêu cầu hỗ trợ thông qua GLPI.
3. **IT Asset Management:** Thu thập và quản lý thông tin thiết bị bằng GLPI Agent.
4. **Remote IT Support:** Thiết lập VPN và hỗ trợ máy trạm từ xa qua Remote Desktop.
5. **Infrastructure Monitoring & Recovery:** Giám sát hệ thống bằng Zabbix, xử lý cảnh báo và kiểm thử phục hồi dữ liệu.

### Project Objectives

- Tích hợp Windows Endpoint vào Active Directory Domain.
- Quản lý chính sách hệ thống thông qua Group Policy.
- Xây dựng quy trình IT Helpdesk và Incident Management.
- Quản lý Inventory và vòng đời tài sản CNTT.
- Thiết lập Remote Support qua VPN được kiểm soát bằng Firewall.
- Giám sát tài nguyên máy trạm và mô phỏng sự cố hiệu năng.
- Kiểm thử Backup, Database Restore và Data Integrity.
- Áp dụng Network Segmentation và Least Privilege.

---

## 2. Technology Stack

| Nhóm công nghệ | Công nghệ | Vai trò |
|---|---|---|
| Virtualization | VMware Workstation | Mô phỏng hạ tầng |
| Firewall & Routing | pfSense CE | Routing, Firewall, VPN |
| Identity Management | Windows Server AD DS | Domain và Authentication |
| DNS | Windows DNS Server | Phân giải tên miền nội bộ |
| Endpoint | Windows 11 Pro | Máy trạm người dùng |
| Policy Management | Group Policy | Chính sách quản lý Workstation |
| ITSM | GLPI | Helpdesk và Incident Management |
| Asset Inventory | GLPI Agent | Thu thập thông tin thiết bị |
| Database | MariaDB | GLPI Database |
| Monitoring | Zabbix 7.0 | Metrics, Triggers và Events |
| Monitoring Agent | Zabbix Agent 2 | Windows Monitoring |
| Remote Access | OpenVPN | Kết nối VPN |
| Remote Support | RDP | Remote Desktop |
| Scripting | PowerShell | Administration và Verification |
| Software Deployment | Winget | Cài đặt phần mềm |
| Backup & Recovery | MariaDB Dump, TAR, SHA-256 | Backup, Restore, Integrity Verification |

---

## 3. Lab Architecture

### 3.1. Infrastructure Overview

Hệ thống được triển khai trên một máy tính vật lý sử dụng VMware Workstation để vận hành nhiều máy ảo.

Các máy ảo được phân chia thành ba mạng nội bộ: SERVER, USER và MGMT.

pfSense đảm nhiệm vai trò Gateway, Inter-network Routing, Firewall và OpenVPN Server.

Windows Server cung cấp các dịch vụ Active Directory và DNS. Ubuntu Server đảm nhiệm GLPI Ticketing và Zabbix Monitoring.

Windows 11 Pro đóng vai trò máy trạm doanh nghiệp, đồng thời là đối tượng được quản lý thông qua AD, GLPI, VPN/RDP và Zabbix.

### 3.2. Network Topology

```text
                     PHYSICAL WINDOWS HOST
                      VMware Workstation
                               |
             +-----------------+------------------+
             |                                    |
       VMware NAT Network                   OpenVPN Client
       VMnet8: 192.168.10.0/24              VPN: 10.10.40.0/24
             |                                    |
             |                                    |
             +-----------------+------------------+
                               |
                      +-----------------+
                      |     pfSense     |
                      | CE 2.5.2        |
                      | Firewall/Router |
                      | OpenVPN Server  |
                      +-----------------+
                               |
              +----------------+----------------+
              |                |                |
        SERVER NETWORK    USER NETWORK     MGMT NETWORK
        10.10.10.0/24     10.10.20.0/24    10.10.30.0/24
           VMnet3            VMnet4           VMnet5
              |                |                |
       +------+------+    +----------+    +-----------+
       |             |    | WIN11-01 |    |  MGMT01   |
       |             |    | Windows  |    |10.10.30.10|
       |             |    | 11 Pro   |    +-----------+
       |             |    |.20.104   |
       |             |    +----------+
       |             |
  +----+-------------+------------------------+
  |                   |                       |
+--------+         +--------+              +----------+
| DC01   |         | FS01   |              | LINUX01  |
| AD/DNS |         | SMB    |              | Zabbix   |
| .10.10 |         | .10.20 |              | .10.30   |
+--------+         +--------+              +----------+
                                             |
                                        +-----------+
                                        | TICKETING |
                                        | GLPI      |
                                        | .10.40    |
                                        +-----------+
```

**Ghi chú:** Trong sơ đồ, địa chỉ `.10.10`, `.10.20`, `.10.30`, `.10.40` là các địa chỉ rút gọn thuộc subnet `10.10.10.0/24`.

Mạng VPN là một mạng tunnel riêng. Người dùng VPN truy cập vào các mạng nội bộ thông qua chính sách định tuyến và Firewall Rules trên pfSense.

Các mạng SERVER, USER và MGMT được mô phỏng bằng các VMware VMnet riêng biệt, không phải VLAN tagging IEEE 802.1Q trên switch vật lý.

### 3.3. Network Segmentation

| Network | Subnet | Gateway | VMware Network | Mục đích |
|---|---|---|---|---|
| WAN | `192.168.10.0/24` | `192.168.10.2` | VMnet8 | VMware NAT / Upstream |
| SERVER | `10.10.10.0/24` | `10.10.10.1` | VMnet3 | Hạ tầng máy chủ |
| USER | `10.10.20.0/24` | `10.10.20.1` | VMnet4 | Máy trạm người dùng |
| MGMT | `10.10.30.0/24` | `10.10.30.1` | VMnet5 | Máy trạm quản trị |
| VPN | `10.10.40.0/24` | VPN Tunnel | OpenVPN | Remote IT Support |

pfSense sử dụng địa chỉ WAN `192.168.10.129` trong mạng NAT của VMware.

Các interface SERVER, USER và MGMT sử dụng những subnet riêng biệt nhằm mô phỏng một môi trường doanh nghiệp có phân đoạn mạng.

**Mục đích thiết kế:**

- Tách máy chủ khỏi máy trạm người dùng.
- Tạo phân đoạn riêng dành cho quản trị.
- Kiểm soát truy cập giữa các mạng thông qua pfSense.
- Giảm phạm vi truy cập không cần thiết.
- Hỗ trợ kiểm thử Firewall Rules theo từng nguồn và đích.

### 3.4. Server & Endpoint Inventory

| Hostname | IP Address | Hệ điều hành / Nền tảng | Vai trò |
|---|---|---|---|
| pfSense | `10.10.10.1` | pfSense CE 2.5.2 | Firewall, Routing, VPN |
| DC01 | `10.10.10.10` | Windows Server | Active Directory, DNS |
| FS01 | `10.10.10.20` | Windows Server | File Server |
| LINUX01 | `10.10.10.30` | Ubuntu Server | Zabbix Server |
| TICKETING | `10.10.10.40` | Ubuntu | GLPI, Apache, MariaDB |
| WIN11-01 | `10.10.20.104` | Windows 11 Pro | Domain Workstation |
| MGMT01 | `10.10.30.10` | Management Workstation | Quản trị hệ thống |

**Domain:** `corp.local`

**Domain Controller:** `DC01`

**DNS Server:** `10.10.10.10`

**GLPI Server:** `10.10.10.40`

**Zabbix Server:** `10.10.10.30`

**Managed Endpoint:** `WIN11-01`

### 3.5. Network Service Flows

| Source | Destination | Protocol / Port | Chức năng |
|---|---|---|---|
| WIN11-01 | DC01 | DNS TCP/UDP 53 | Domain DNS Resolution |
| WIN11-01 | DC01 | AD authentication ports | Domain Authentication |
| WIN11-01 | FS01 | SMB TCP 445 | File Sharing |
| WIN11-01 | TICKETING | HTTP TCP 80 | GLPI Web |
| WIN11-01 | LINUX01 | TCP 10051 | Zabbix Active Checks |
| Windows Host | pfSense WAN | OpenVPN UDP 1194 | VPN Connection |
| VPN Client | WIN11-01 | RDP TCP 3389 | Remote Support |
| Windows Host | TICKETING | SSH TCP 22 | Administration / Backup |

**Lưu ý:** Bảng này mô tả các luồng dịch vụ chính, không phải danh sách đầy đủ tất cả Active Directory ports hoặc toàn bộ Firewall Rules.

### 3.6. Security Architecture

Hệ thống áp dụng nhiều lớp kiểm soát:

**Network Layer**

pfSense kiểm soát lưu lượng giữa các mạng, bao gồm các kết nối từ VPN vào USER và SERVER Network.

**Identity Layer**

Active Directory quản lý tài khoản và Computer Objects trong Domain.

**Endpoint Layer**

Windows Firewall giới hạn RDP chỉ cho phép nguồn thuộc VPN Subnet `10.10.40.0/24`.

Network Level Authentication được kích hoạt cho Remote Desktop.

**Server Layer**

Ubuntu Ticketing sử dụng UFW với chính sách mặc định từ chối kết nối đến.

SSH chỉ cho phép nguồn quản trị được xác định.

MariaDB được cấu hình chỉ lắng nghe trên Loopback `127.0.0.1:3306`.

**Backup Layer**

Backup được lưu với quyền truy cập hạn chế và xác minh tính toàn vẹn bằng SHA-256.

Một số bản backup được sao chép sang Windows Host để giảm phụ thuộc vào VM nguồn.

### 3.7. Architecture Design Decisions

| Quyết định | Lý do |
|---|---|
| Tái sử dụng Active Directory từ Project "Enterprise-IT-Infrastructure" | Tập trung Project vào Operations và IT Support |
| Tách SERVER, USER, MGMT | Mô phỏng Network Segmentation |
| Sử dụng pfSense làm Gateway | Tập trung quản lý Routing, Firewall và VPN |
| Sử dụng GLPI | Kết hợp Helpdesk với Asset Management |
| Sử dụng Zabbix | Thực hành Agent Monitoring, Trigger và Event |
| OpenVPN trước khi sử dụng RDP | Tránh mở trực tiếp RDP ra WAN |
| Database Restore trên DB riêng | Kiểm thử Backup mà không ghi đè Production Lab Database |
| Sao chép Backup ra Windows Host | Giảm rủi ro phụ thuộc vào ổ đĩa VM |

Sơ đồ chi tiết và phân tích các quyết định kỹ thuật được trình bày trong [Architecture Documentation](architecture/architecture.md).

---

## 4. Key Implementations

### Phase 1 – Windows Endpoint Management

**Mục tiêu:** Triển khai và quản lý Windows Workstation trong môi trường Active Directory.

Các nội dung đã thực hiện:

- Triển khai Windows 11 Pro trên VMware Workstation.
- Kết nối WIN11-01 vào mạng USER.
- Gia nhập Domain `corp.local`.
- Xác minh khả năng phát hiện Domain Controller.
- Áp dụng Workstation Baseline Group Policy.
- Cài đặt Google Chrome và 7-Zip bằng Winget.
- Kiểm tra DNS, Domain Authentication và SMB File Sharing.

**Kết quả:** WIN11-01 hoạt động trong Domain, nhận GPO và truy cập được các dịch vụ nội bộ.

[Case Study 01 – Endpoint Management](case-studies/01-endpoint-management.md)

### Phase 2 – IT Service Management

**Mục tiêu:** Xây dựng quy trình IT Helpdesk và Incident Management bằng GLPI.

Các nội dung đã thực hiện:

- Triển khai GLPI trên Ubuntu.
- Sử dụng MariaDB làm Database.
- Tích hợp LDAP với Active Directory.
- Cấu hình tài khoản và vai trò xử lý Ticket.
- Xây dựng quy trình Incident Management.
- Cấu hình SLA với TTO 1 giờ và TTR 4 giờ.

**Kết quả:** GLPI hỗ trợ tiếp nhận, phân loại, phân công, xử lý và đóng Ticket.

[Case Study 02 – ITSM & Incident Management](case-studies/02-itsm-incident-management.md)

### Phase 3 – IT Asset Management

**Mục tiêu:** Quản lý thông tin thiết bị và liên kết Asset với Ticket hỗ trợ.

Các nội dung đã thực hiện:

- Triển khai GLPI Agent trên WIN11-01.
- Thu thập và đồng bộ Inventory.
- Quản lý Computer Name và Inventory Number.
- Liên kết tài sản với Incident.
- Thực hành Asset Lifecycle.

**Kết quả:** WIN11-01 được ghi nhận trong GLPI với Inventory Number `CORP-WS-001`.

[Case Study 03 – IT Asset Management](case-studies/03-it-asset-management.md)

### Phase 4 – Secure Remote IT Support

**Mục tiêu:** Hỗ trợ Windows Workstation từ xa qua VPN với các chính sách Firewall giới hạn quyền truy cập.

Các nội dung đã thực hiện:

- Cấu hình OpenVPN Server trên pfSense.
- Xác thực bằng tài khoản và Certificate.
- Cấu hình Split Tunneling.
- Thiết lập Firewall Rules cho VPN.
- Kết nối RDP đến WIN11-01.
- Kích hoạt Network Level Authentication.
- Giới hạn Windows Firewall RDP về VPN Subnet.

**Kết quả kiểm thử:**

```text
VPN Client: 10.10.40.2
RDP Target: 10.10.20.104
Port: TCP 3389
TcpTestSucceeded: True
```

[Case Study 04 – Secure Remote Support](case-studies/04-vpn-remote-support.md)

### Phase 5 – Monitoring, Incident Response & Backup

**Mục tiêu:** Giám sát Endpoint, phát hiện sự cố và kiểm thử khả năng phục hồi dữ liệu.

Các nội dung đã thực hiện:

- Triển khai Zabbix Server và Zabbix Agent 2.
- Cấu hình Windows Monitoring bằng Active Checks.
- Theo dõi CPU và các Metrics hệ thống.
- Tạo Trigger CPU High Usage.
- Mô phỏng CPU Load.
- Ghi nhận Problem và Recovery Event.
- Tạo GLPI Ticket #7 để quản lý Monitoring Incident.
- Backup MariaDB và GLPI Files.
- Restore Database sang cơ sở dữ liệu thử nghiệm.
- Xác minh SHA-256 và thực hiện Off-VM Backup.
- Kiểm tra cấu hình Firewall và RDP Security.

**Kết quả nổi bật:**

- Zabbix phát hiện CPU High Usage.
- Problem được chuyển sang trạng thái Recovery.
- GLPI Ticket #7 được sử dụng để xử lý Incident.
- Phục hồi thành công 447 bảng MariaDB.
- Xác minh tính toàn vẹn của các bản backup.

[Case Study 05 – Monitoring & Recovery](case-studies/05-monitoring-backup-recovery.md)

---

## 5. Technical Challenges & Troubleshooting

| Sự cố | Cách tiếp cận |
|---|---|
| Winget không hoạt động với tài khoản Administrator | Kiểm tra App Installer và đăng ký lại package |
| Firewall Rules khiến dịch vụ nội bộ không truy cập được | Phân tích interface, source, destination và port |
| Zabbix Active Checks gặp vấn đề kết nối | Kiểm tra Agent, Server Port và Firewall |
| CPU Trigger không tạo Event như mong đợi | Kiểm tra Trigger Expression và dữ liệu Metrics |
| RDP cần giới hạn phạm vi truy cập | Cấu hình Firewall theo VPN Subnet |
| Cần kiểm tra khả năng phục hồi GLPI | Database Restore và xác minh dữ liệu |

Mỗi Case Study sẽ phân tích cụ thể vấn đề kỹ thuật, quá trình xử lý và kết quả kiểm chứng.

---

## 6. Validation Results

| Test Case | Result |
|---|---|
| Windows Domain Integration | PASS |
| Group Policy Application | PASS |
| GLPI LDAP Integration | PASS |
| GLPI Inventory | PASS |
| OpenVPN Authentication | PASS |
| VPN-to-Windows RDP | PASS |
| RDP Network Level Authentication | PASS |
| Zabbix Active Monitoring | PASS |
| CPU Alert & Recovery | PASS |
| GLPI Incident Documentation | PASS |
| MariaDB Restore – 447 Tables | PASS |
| Backup SHA-256 Verification | PASS |
| Ubuntu UFW Security | PASS |
| Full GLPI Application Restore | NOT TESTED |

Chi tiết kết quả: [Validation Summary](results/validation-summary.md).

---

## 7. Security Considerations

- Network Segmentation cho SERVER, USER và MGMT.
- pfSense kiểm soát lưu lượng liên mạng.
- Không mở trực tiếp RDP ra WAN qua NAT Port Forward.
- OpenVPN sử dụng tài khoản và Certificate.
- Windows Firewall chỉ cho phép RDP từ VPN Subnet.
- RDP Network Level Authentication được kích hoạt.
- Ubuntu Ticketing sử dụng UFW Default Deny Incoming.
- SSH được giới hạn theo địa chỉ nguồn.
- MariaDB chỉ lắng nghe trên Loopback.
- Backup được bảo vệ bằng File Permissions và Integrity Checks.

Dự án sử dụng pfSense CE 2.5.2 trong mạng lab. Đây là phiên bản cũ và không được xem là cấu hình sẵn sàng triển khai trên Internet công cộng.

---

## 8. Project Outcomes

Dự án cung cấp kinh nghiệm thực hành trong các lĩnh vực:

**Endpoint Administration:** Windows Deployment, Domain Join, GPO, DNS, SMB và Software Deployment.

**IT Service Management:** Ticket Lifecycle, Incident Management, SLA và Troubleshooting.

**IT Asset Management:** Device Inventory, Asset Identification và Asset-to-Ticket Association.

**Remote Infrastructure Support:** OpenVPN, pfSense Firewall, Split Tunneling và RDP.

**Monitoring & Incident Response:** Zabbix Agent, Trigger, Problem/Recovery Event và Incident Documentation.

**Backup & Recovery:** Database Backup, Restore Verification, Off-VM Backup và SHA-256 Integrity Validation.

---

## 9. Limitations & Future Improvements

Dự án được triển khai trong môi trường lab ảo hóa, chưa phải Production.

Các giới hạn hiện tại:

- Phụ thuộc tài nguyên máy Host.
- Chưa triển khai High Availability.
- pfSense sử dụng phiên bản cũ.
- GLPI sử dụng HTTP trong mạng lab.
- Chưa kiểm thử Full Application Disaster Recovery.
- Chưa triển khai tích hợp API tự động từ Zabbix sang GLPI.
- Chưa triển khai cơ chế Backup Scheduling và Restore Automation hoàn chỉnh.

Những hướng cải tiến tiềm năng:

- Nâng cấp và chuẩn hóa phiên bản phần mềm.
- HTTPS và Certificate Management.
- Monitoring-to-Ticket Automation.
- Automated Backup & Recovery Validation.
- Infrastructure as Code.
- Centralized Logging và Security Monitoring.

---

## 10. Documentation

| Tài liệu | Nội dung |
|---|---|
| [System Architecture](architecture/architecture.md) | Kiến trúc và các luồng giao tiếp |
| [Endpoint Management](case-studies/01-endpoint-management.md) | Windows Endpoint và Domain |
| [ITSM & Incident Management](case-studies/02-itsm-incident-management.md) | GLPI Helpdesk |
| [IT Asset Management](case-studies/03-it-asset-management.md) | GLPI Inventory |
| [Remote IT Support](case-studies/04-vpn-remote-support.md) | OpenVPN và RDP |
| [Monitoring & Recovery](case-studies/05-monitoring-backup-recovery.md) | Zabbix, Incident và Backup |
| [Validation Summary](results/validation-summary.md) | Test Cases và kết quả |

---

## 11. Project Scope

Đây là dự án thực hành cá nhân nhằm mô phỏng môi trường IT Operations doanh nghiệp.

Repository phục vụ mục đích trình bày kiến trúc, quá trình triển khai, xử lý sự cố và kết quả kiểm thử.