# System Architecture – Enterprise IT Support & Infrastructure Operations Lab

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Environment:** VMware Workstation, Windows Server, Windows 11, Ubuntu, pfSense  
**Architecture Type:** Virtualized Enterprise Infrastructure Lab

---

## 1. Architecture Overview

Project mô phỏng hệ thống IT Operations của một doanh nghiệp quy mô nhỏ, kết hợp các dịch vụ quản lý danh tính, máy trạm, Service Desk, quản lý tài sản, hỗ trợ từ xa và giám sát hạ tầng.

Hệ thống kế thừa các thành phần nền tảng từ Project "Enterprise-IT-Infrastructure-Lab", bao gồm:

- pfSense Firewall và Routing.
- Active Directory Domain Services.
- DNS Server.
- File Server.
- Các phân đoạn mạng SERVER, USER và MGMT.

Project mở rộng hạ tầng bằng cách bổ sung:

- Windows 11 Endpoint.
- GLPI IT Service Management.
- GLPI Agent Inventory.
- OpenVPN Remote Access.
- Remote Desktop Support.
- Zabbix Infrastructure Monitoring.
- Backup & Restore Verification.

### Architectural Objectives

Kiến trúc được thiết kế nhằm đáp ứng các yêu cầu:

1. Quản lý danh tính và thiết bị tập trung.
2. Phân tách Workstation và Server Infrastructure.
3. Kiểm soát lưu lượng giữa các mạng.
4. Cung cấp hệ thống Helpdesk tập trung.
5. Thu thập thông tin tài sản CNTT.
6. Hỗ trợ Endpoint từ xa qua VPN.
7. Giám sát tài nguyên và phát hiện sự cố.
8. Bảo vệ dữ liệu cấu hình và Database.
9. Cho phép kiểm thử, chẩn đoán và khôi phục từng thành phần mà không ảnh hưởng không cần thiết đến các thành phần khác.

---

## 2. Infrastructure Topology

### 2.1. Physical & Virtualization Layer

Toàn bộ môi trường được triển khai trên một Windows Physical Host chạy VMware Workstation.

VMware cung cấp các Virtual Network riêng biệt cho những nhóm hệ thống.

| VMware Network | Network | Purpose |
|---|---|---|
| VMnet8 | `192.168.10.0/24` | VMware NAT / pfSense WAN |
| VMnet3 | `10.10.10.0/24` | SERVER Network |
| VMnet4 | `10.10.20.0/24` | USER Network |
| VMnet5 | `10.10.30.0/24` | MGMT Network |

Việc sử dụng các VMnet riêng tạo ra những miền Layer 2 độc lập phục vụ việc mô phỏng Network Segmentation.

Trong lab, thuật ngữ VLAN10, VLAN20 và VLAN30 được sử dụng theo chức năng mạng. Tuy nhiên, các mạng này được mô phỏng thông qua VMware Virtual Networks, không phải mô hình switch trunk gắn tag IEEE 802.1Q.

### 2.2. High-Level Topology

```text
                     WINDOWS PHYSICAL HOST
                       VMware Workstation
                               |
                 +-------------+-------------+
                 |                           |
            VMnet8 / NAT                OpenVPN Client
          192.168.10.0/24              Remote Technician
                 |                           |
                 +-------------+-------------+
                               |
                      +----------------+
                      |    pfSense     |
                      | CE 2.5.2       |
                      | Firewall       |
                      | Routing        |
                      | OpenVPN        |
                      +----------------+
                               |
              +----------------+----------------+
              |                |                |
           VMnet3           VMnet4           VMnet5
           SERVER            USER             MGMT
        10.10.10.0/24    10.10.20.0/24    10.10.30.0/24
              |                |                |
       +------+------+    +-----------+    +-----------+
       |      |      |    | WIN11-01  |    |  MGMT01   |
       |      |      |    | Windows11 |    | 10.10.30.10
       |      |      |    | .20.104   |    +-----------+
       |      |      |    +-----------+
       |      |      |
       |      |      +------------------+
       |      |                         |
    +------+ +------+               +----------+
    | DC01 | | FS01 |               | LINUX01  |
    | AD   | | SMB  |               | Zabbix   |
    | DNS  | |      |               | .10.30   |
    |.10.10| |.10.20|               +----------+
    +------+ +------+                     |
                                          |
                                      +-----------+
                                      | TICKETING |
                                      | GLPI      |
                                      | .10.40    |
                                      +-----------+

                   OpenVPN Tunnel Network
                       10.10.40.0/24
```

**Chú thích:** DC01, FS01, LINUX01 và TICKETING đều nằm trên cùng SERVER Network. Các nhánh trong sơ đồ chỉ thể hiện nhóm thiết bị, không phải kết nối vật lý nối tiếp giữa các máy chủ.

OpenVPN Tunnel là một mạng Layer 3 riêng được quản lý bởi pfSense.

---

## 3. Network Architecture

### 3.1. Network Addressing Plan

| Network | Subnet | Gateway | Function |
|---|---|---|---|
| WAN | `192.168.10.0/24` | `192.168.10.2` | VMware NAT |
| SERVER | `10.10.10.0/24` | `10.10.10.1` | Infrastructure Services |
| USER | `10.10.20.0/24` | `10.10.20.1` | Endpoints |
| MGMT | `10.10.30.0/24` | `10.10.30.1` | Administration |
| VPN | `10.10.40.0/24` | OpenVPN Tunnel | Remote Access |

pfSense WAN:

`192.168.10.129`

OpenVPN Server:

`192.168.10.129:1194/UDP`

### 3.2. IP Inventory

| Device | Address | Role |
|---|---|---|
| DC01 | `10.10.10.10` | AD DS / DNS |
| FS01 | `10.10.10.20` | File Server |
| LINUX01 | `10.10.10.30` | Zabbix Server |
| TICKETING | `10.10.10.40` | GLPI Server |
| WIN11-01 | `10.10.20.104` | Windows Endpoint |
| MGMT01 | `10.10.30.10` | Management Workstation |
| Windows Host – VMnet3 | `10.10.10.254` | Lab Host Administration |

### 3.3. Inter-Network Routing

pfSense đảm nhiệm việc định tuyến giữa các mạng SERVER, USER và MGMT.

Ví dụ khi WIN11-01 truy cập GLPI:

```text
WIN11-01
10.10.20.104
      |
      v
Default Gateway
10.10.20.1
      |
      v
pfSense Firewall / Routing
      |
      v
TICKETING
10.10.10.40:80
```

Để một kết nối thành công, các điều kiện cần bao gồm:

- Client có địa chỉ IP và tuyến đường phù hợp.
- Gateway định tuyến được tới mạng đích.
- Firewall không chặn kết nối.
- Server có dịch vụ lắng nghe trên cổng tương ứng.
- Firewall cục bộ của Server cho phép kết nối.
- Các yêu cầu xác thực và phân quyền của dịch vụ được đáp ứng.

Vì vậy, khi xử lý lỗi kết nối, cần phân biệt giữa lỗi Network, Routing, Firewall và Application.

### 3.4. Network Segmentation Rationale

**SERVER Network**

Chứa những dịch vụ hạ tầng cần quản lý tập trung:

- Active Directory.
- DNS.
- File Server.
- GLPI.
- Zabbix.

**USER Network**

Chứa Workstations và Endpoints.

Các thiết bị trong mạng này chỉ nên truy cập những dịch vụ nội bộ cần thiết.

**MGMT Network**

Dành cho các thiết bị hoặc hoạt động quản trị.

Việc thiết kế mạng quản trị riêng giúp tạo cơ sở cho chính sách phân quyền và kiểm soát nguồn quản trị.

**VPN Network**

Dành cho những Client đã thiết lập kết nối OpenVPN thành công.

Lưu lượng từ VPN Client tới mạng nội bộ tiếp tục chịu kiểm soát bởi pfSense Firewall.

---

## 4. Identity & Access Architecture

### 4.1. Active Directory

Hạ tầng Identity sử dụng Active Directory Domain:

`corp.local`

Domain Controller:

`DC01 – 10.10.10.10`

Active Directory đảm nhiệm:

- Quản lý Domain Users.
- Quản lý Computer Objects.
- Cung cấp cơ chế xác thực cho Domain.
- Tổ chức đối tượng thông qua OU.
- Áp dụng Group Policy.

### 4.2. Endpoint Integration

WIN11-01 đã được gia nhập Domain và quản lý tại:

`OU=CORP_Computers,DC=corp,DC=local`

Các chính sách được xác minh trên Endpoint:

- Default Domain Policy.
- GPO_Workstation_Baseline.

Điều này cho phép triển khai các cấu hình quản trị tập trung trên máy trạm.

### 4.3. GLPI LDAP Integration

GLPI sử dụng Active Directory làm nguồn danh tính thông qua LDAP.

Luồng tích hợp:

```text
User / Technician
        |
        v
GLPI Login
        |
        v
LDAP Directory Configuration
        |
        v
Active Directory (DC01)
        |
        v
Authentication / User Attributes
        |
        v
GLPI Profile & Permissions
```

GLPI sử dụng dữ liệu tài khoản được tích hợp để phục vụ những hoạt động Helpdesk.

LDAP Directory Integration và Role-Based Permission của GLPI là hai lớp chức năng khác nhau.

---

## 5. IT Service Management Architecture

### 5.1. GLPI Application Stack

GLPI được triển khai trên Ubuntu Ticketing:

| Component | Configuration |
|---|---|
| Host | TICKETING |
| IP | `10.10.10.40` |
| Operating System | Ubuntu |
| Application | GLPI |
| Web Server | Apache |
| Database | MariaDB 10.11 |
| Database Name | `glpidb` |
| Application Path | `/var/www/glpi` |
| HTTP Port | TCP 80 |

GLPI và MariaDB được triển khai trên cùng một VM.

Đây là cấu trúc phù hợp với phạm vi lab nhỏ, nhưng chưa có tính dự phòng như một kiến trúc High Availability.

### 5.2. Service Desk Workflow

```text
User Reports Incident
          |
          v
      GLPI Ticket
          |
          v
Categorization & Priority
          |
          v
Technician Assignment
          |
          v
Investigation & Diagnosis
          |
          v
Task / Follow-up / Solution
          |
          v
Resolution & Closure
```

Trong môi trường lab, Ticket được sử dụng để ghi nhận các công việc hỗ trợ kỹ thuật.

Ví dụ:

- Ticket #6: Remote IT Support.
- Ticket #7: Monitoring Incident.

### 5.3. SLA Architecture

GLPI được cấu hình với mục tiêu:

| SLA Metric | Target |
|---|---|
| Time to Own | 1 giờ |
| Time to Resolve | 4 giờ |

SLA được sử dụng để thực hành quản lý thời hạn xử lý Incident.

Chưa có phép đo kiểm chứng năng lực đáp ứng SLA trên quy mô Ticket lớn.

---

## 6. IT Asset Management Architecture

### 6.1. Agent-Based Inventory

GLPI Agent được triển khai trên WIN11-01.

Agent thu thập dữ liệu Inventory từ hệ điều hành và gửi về GLPI Server.

```text
WIN11-01
   |
   v
GLPI Agent
   |
   | Inventory Data
   v
GLPI Server
   |
   v
MariaDB
   |
   v
Computer Asset
```

### 6.2. Asset Identification

WIN11-01 được ghi nhận với:

| Property | Value |
|---|---|
| Hostname | WIN11-01 |
| Inventory Number | CORP-WS-001 |
| Asset Type | Computer |
| Operating System | Windows 11 Pro |

Asset Inventory được sử dụng để hỗ trợ việc tra cứu thiết bị và quá trình xử lý Incident.

### 6.3. ITSM & ITAM Relationship

Trong hệ thống GLPI, Ticket có thể liên kết với Computer Asset.

Việc liên kết giúp:

- Xác định thiết bị bị ảnh hưởng.
- Tra cứu Hardware và Software Inventory.
- Theo dõi các Ticket liên quan đến cùng tài sản.
- Hỗ trợ quyết định bảo trì hoặc thay thế thiết bị.

Trong Project, WIN11-01 là tài sản được quản lý và đồng thời là đối tượng của các hoạt động Remote Support, Monitoring và Incident Management.

**Giới hạn xác minh:** Thông tin về WIN11-01 trong Ticket #7 đã được ghi nhận. Liên kết trực tiếp tại trường Associated Items cần được xác minh riêng trước khi công bố như một kết quả kiểm thử độc lập.

---

## 7. Remote Access Architecture

### 7.1. OpenVPN

OpenVPN Server được triển khai trên pfSense.

| Property | Value |
|---|---|
| Server WAN IP | `192.168.10.129` |
| Protocol | UDP |
| Port | 1194 |
| Tunnel Network | `10.10.40.0/24` |
| Authentication | User Credentials + Certificate |

Windows Host sử dụng OpenVPN Client để kết nối vào môi trường lab.

Địa chỉ đã nhận trong phiên kiểm thử:

`10.10.40.2`

### 7.2. Remote Desktop Flow

```text
Windows Host
OpenVPN Client
10.10.40.2
       |
       v
VPN Tunnel
       |
       v
pfSense OpenVPN Firewall
       |
       | TCP 3389
       v
WIN11-01
10.10.20.104
       |
       v
Windows Firewall + NLA
       |
       v
Remote Desktop Session
```

### 7.3. Remote Access Security

Các biện pháp đã xác minh:

- OpenVPN Authentication.
- Firewall Rule giới hạn RDP đến WIN11-01.
- Windows Firewall RDP giới hạn RemoteAddress.
- Network Level Authentication.
- Không có NAT Port Forward trực tiếp TCP 3389 từ WAN trong cấu hình đã kiểm tra.

Kết quả:

```text
SourceAddress    : 10.10.40.2
RemoteAddress    : 10.10.20.104
RemotePort       : 3389
TcpTestSucceeded : True
```

Đây là bằng chứng về khả năng kết nối đến RDP thông qua OpenVPN Tunnel.

---

## 8. Monitoring Architecture

### 8.1. Zabbix Server

Zabbix được triển khai trên LINUX01:

| Component | Configuration |
|---|---|
| IP | `10.10.10.30` |
| Zabbix Version | 7.0.31 |
| Database | PostgreSQL 14 |
| Web Server | Nginx |
| Monitored Host | WIN11-01 |
| Monitoring Type | Zabbix Agent 2 Active Checks |

### 8.2. Active Monitoring Flow

```text
WIN11-01
Zabbix Agent 2
      |
      | TCP 10051
      v
pfSense
      |
      v
LINUX01
Zabbix Server
      |
      v
Metrics / Trigger Evaluation
      |
      v
Problem / Recovery Events
```

Active Checks cho phép Agent liên hệ Zabbix Server để nhận danh sách kiểm tra và gửi dữ liệu giám sát.

### 8.3. CPU Incident Simulation

Trong Phase 5, một Trigger thử nghiệm được sử dụng:

```text
avg(/WIN11-01/system.cpu.util,1m)>20
```

CPU Load được tạo có kiểm soát trên WIN11-01.

Kết quả đã ghi nhận:

| Event | Time |
|---|---|
| Problem | 22:53:41 |
| Recovery | 22:57:41 |

Sau khi xác minh Monitoring Event, GLPI Ticket #7 được sử dụng để ghi nhận Incident.

Việc chuyển từ Monitoring sang Ticket được thực hiện thủ công.

---

## 9. Backup & Recovery Architecture

### 9.1. Backup Scope

GLPI Server sử dụng hai nhóm dữ liệu cần sao lưu:

**Database**

- MariaDB Database `glpidb`.

**Application Data**

- GLPI Configuration.
- GLPI Files.
- Plugins.
- Marketplace.
- Apache Configuration.
- Các cấu hình ứng dụng liên quan.

Ngoài ra, pfSense Configuration được sao lưu dưới dạng encrypted backup.

### 9.2. Backup Workflow

```text
GLPI Database + Application Files
                 |
                 v
        SQL Dump / TAR Archive
                 |
                 v
         SHA-256 Verification
                 |
                 v
          Local Backup Storage
                 |
                 v
             SCP Transfer
                 |
                 v
            Windows Host
           Off-VM Backup
```

### 9.3. Restore Verification

Database Backup được phục hồi sang:

`glpidb_restore_test`

Kết quả:

- Source Database: 447 bảng.
- Restored Database: 447 bảng.
- Ticket #7 được đối chiếu thành công.
- Asset WIN11-01 được đối chiếu thành công.
- Inventory Number CORP-WS-001 được xác minh.

**Result: PASS – Database Restore Verification**

### 9.4. Backup Limitations

- Chưa thực hiện Full GLPI Application Disaster Recovery.
- Chưa đo RPO hoặc RTO.
- Chưa thiết lập Backup Scheduling tự động hoàn chỉnh.
- Backup trên Windows Host chưa phải Offsite Backup độc lập.
- Không công khai Backup Archives hoặc Encrypted pfSense Configuration.

---

## 10. Security Architecture

### 10.1. Defense in Depth

Hệ thống sử dụng nhiều lớp kiểm soát thay vì phụ thuộc vào một cơ chế duy nhất.

| Layer | Security Control |
|---|---|
| Network | pfSense Firewall |
| Segmentation | SERVER / USER / MGMT |
| Identity | Active Directory |
| Application | GLPI Profiles |
| Remote Access | OpenVPN Authentication |
| Endpoint | Windows Firewall, RDP NLA |
| Linux Server | UFW |
| Database | MariaDB Loopback Binding |
| Backup | File Permissions, SHA-256 Verification |

### 10.2. Ubuntu Ticketing Firewall

UFW được cấu hình:

```text
Default incoming: deny
Default outgoing: allow
```

Các luồng truy cập được cho phép:

| Source | Destination | Service |
|---|---|---|
| `10.10.10.254` | `10.10.10.40` | SSH TCP 22 |
| `10.10.10.0/24` | `10.10.10.40` | HTTP TCP 80 |
| `10.10.20.0/24` | `10.10.10.40` | HTTP TCP 80 |
| `10.10.30.0/24` | `10.10.10.40` | HTTP TCP 80 |

Kết quả kiểm thử từ Windows Host:

- TCP 22: PASS.
- TCP 80: PASS.
- TCP 3306: Không kết nối được.

### 10.3. Windows RDP Security

Remote Desktop được giới hạn:

```text
Source Network : 10.10.40.0/24
Destination    : 10.10.20.104
Service        : RDP
```

Network Level Authentication:

`UserAuthentication = 1`

Hai lớp kiểm soát chính:

1. pfSense OpenVPN Firewall.
2. Windows Defender Firewall.

### 10.4. Remaining Security Risks

Các giới hạn hiện tại:

- pfSense CE 2.5.2 là phiên bản cũ.
- GLPI sử dụng HTTP trong lab.
- OpenVPN chưa có MFA.
- SSH Password Authentication vẫn được cho phép.
- Chưa thực hiện Penetration Testing.
- Chưa kiểm toán đầy đủ tất cả Firewall Rules, Floating Rules và IPv6.
- Chưa triển khai High Availability.

Vì vậy, kiến trúc này được đánh giá theo phạm vi thực hành lab, chưa được xác nhận Production-Ready.

---

## 11. Architecture Validation

Các kết quả kiểm thử chính:

| Test | Result |
|---|---|
| Windows Domain Membership | PASS |
| Domain Controller Discovery | PASS |
| Group Policy Application | PASS |
| File Server SMB Connectivity | PASS |
| GLPI LDAP Integration | PASS |
| GLPI Inventory | PASS |
| VPN Tunnel Establishment | PASS |
| VPN-to-WIN11 RDP | PASS |
| Windows RDP NLA | PASS |
| Zabbix Active Monitoring | PASS |
| CPU Problem / Recovery Event | PASS |
| GLPI Incident Documentation | PASS |
| MariaDB Restore – 447 Tables | PASS |
| Backup Integrity Verification | PASS |
| Ubuntu UFW Security | PASS |

---

## 12. Architecture Trade-offs

| Design Choice | Advantage | Limitation |
|---|---|---|
| VMware Workstation | Linh hoạt, dễ xây dựng lab | Phụ thuộc Host vật lý |
| pfSense | Routing, Firewall, VPN tập trung | Single Point of Failure |
| Một Domain Controller | Đơn giản, tiết kiệm tài nguyên | Không có dự phòng AD |
| GLPI + MariaDB cùng VM | Dễ triển khai và backup | Không có HA |
| Zabbix Agent Active Checks | Agent chủ động gửi dữ liệu | Phụ thuộc kết nối đến Zabbix Server |
| Split Tunneling | Giảm lưu lượng qua VPN | Cần quản lý chính sách Client |
| Off-VM Backup trên Host | Bảo vệ khỏi một số lỗi VM | Không bảo vệ khỏi mất toàn bộ Host |
| Manual Incident Documentation | Dễ kiểm chứng quy trình | Chưa tự động hóa Monitoring-to-ITSM |

Những lựa chọn trên được đưa ra theo giới hạn tài nguyên và mục tiêu học tập của dự án.

---

## 13. Technical Outcomes

Kiến trúc đã tích hợp các thành phần hạ tầng thành một hệ thống thống nhất:

- Active Directory quản lý danh tính người dùng và máy trạm.
- DNS hỗ trợ phân giải dịch vụ nội bộ.
- pfSense cung cấp Routing, Firewall và VPN.
- Windows 11 đóng vai trò Managed Endpoint.
- GLPI quản lý Helpdesk và Asset Inventory.
- OpenVPN kết hợp RDP cung cấp Remote Support.
- Zabbix giám sát trạng thái Endpoint.
- MariaDB Backup và Restore Verification hỗ trợ bảo vệ dữ liệu.

### Skills Demonstrated

- Infrastructure Architecture.
- VMware Virtual Networking.
- Network Segmentation.
- IPv4 Routing.
- Firewall Policy Design.
- Active Directory Integration.
- ITSM / ITAM Architecture.
- VPN Remote Access Design.
- Infrastructure Monitoring.
- Backup & Recovery Planning.
- Security Hardening.
- Cross-System Troubleshooting.

---

## 14. Tổng kết

Project sử dụng hạ tầng Active Directory và Network Services từ Project 1 để xây dựng thêm các nghiệp vụ IT Operations.

Thay vì triển khai những dịch vụ rời rạc, hệ thống kết hợp Endpoint Management, ITSM, Asset Management, Remote Support, Monitoring và Backup thành một môi trường thực hành thống nhất.

Kiến trúc sử dụng pfSense làm điểm kiểm soát mạng, Active Directory làm nền tảng danh tính, GLPI quản lý quy trình hỗ trợ và Zabbix cung cấp dữ liệu Monitoring.

Các kết quả Domain Join, VPN/RDP, CPU Alert, GLPI Ticket và MariaDB Restore đã được kiểm thử thực tế trong lab.

Dự án thể hiện khả năng thiết kế, tích hợp, vận hành và kiểm tra những thành phần hạ tầng CNTT cơ bản trong môi trường doanh nghiệp mô phỏng.