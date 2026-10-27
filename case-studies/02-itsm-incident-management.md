# Case Study 02 – IT Service Management & Incident Management

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Phase:** 02 – IT Service Management  
**Technologies:** GLPI, Ubuntu, Apache, MariaDB, Active Directory, LDAP, pfSense

---

## 1. Project Challenge

Trong môi trường doanh nghiệp, bộ phận IT tiếp nhận nhiều yêu cầu từ người dùng liên quan đến tài khoản, thiết bị, phần mềm, mạng và các dịch vụ nội bộ.

Nếu yêu cầu hỗ trợ chỉ được trao đổi qua tin nhắn, điện thoại hoặc email mà không có hệ thống quản lý tập trung, doanh nghiệp có thể gặp các vấn đề:

- Không theo dõi được trạng thái và người chịu trách nhiệm xử lý.
- Khó xác định thứ tự ưu tiên giữa các sự cố.
- Thiếu lịch sử xử lý và phương án khắc phục.
- Không có cơ sở đo lường thời gian phản hồi hoặc giải quyết.
- Khó liên kết sự cố với người dùng và tài sản CNTT.
- Thiếu dữ liệu để đánh giá hiệu quả hoạt động của bộ phận IT.

Phase 2 giải quyết bài toán này bằng cách triển khai GLPI làm nền tảng IT Service Management (ITSM) trong môi trường lab.

Hệ thống tận dụng Active Directory để quản lý danh tính người dùng và cung cấp quy trình xử lý Ticket từ lúc tiếp nhận đến khi hoàn thành.

### Objectives

- Triển khai GLPI trên Ubuntu Server.
- Xây dựng cơ sở dữ liệu MariaDB cho GLPI.
- Tích hợp hệ thống người dùng Active Directory thông qua LDAP.
- Phân quyền người yêu cầu hỗ trợ và nhân viên IT.
- Cấu hình danh mục Ticket và quy trình Incident Management.
- Xây dựng chính sách SLA với các chỉ tiêu TTO và TTR.
- Thực hành tiếp nhận, phân công, xử lý và đóng Ticket.
- Chuẩn bị nền tảng liên kết Ticket với Asset Management và Monitoring ở các Phase tiếp theo.

---

## 2. Technical Concepts

### 2.1. IT Service Management

**IT Service Management (ITSM)** là cách tiếp cận tổ chức và quản lý các dịch vụ CNTT nhằm đáp ứng nhu cầu của người dùng và hoạt động kinh doanh.

ITSM không chỉ bao gồm công việc sửa lỗi kỹ thuật mà còn liên quan đến quy trình, trách nhiệm, chất lượng dịch vụ và cải tiến hoạt động.

Một số quy trình thường gặp:

- Incident Management.
- Service Request Management.
- Problem Management.
- Change Management.
- IT Asset Management.
- Service Level Management.

Trong Project, phạm vi thực hành tập trung vào **Incident Management, Helpdesk Operations và Service Level Management**.

### 2.2. Incident Management

Incident là sự kiện gây gián đoạn hoặc suy giảm chất lượng của một dịch vụ CNTT.

Mục tiêu của Incident Management là khôi phục hoạt động dịch vụ trong thời gian phù hợp và giảm ảnh hưởng đến người dùng.

Ví dụ:

- Người dùng không truy cập được File Server.
- Workstation gặp lỗi kết nối mạng.
- Phần mềm không hoạt động.
- Máy trạm có hiệu năng bất thường.

Một Incident thường được xử lý theo quy trình:

```text
Incident Reported
       |
       v
Ticket Created
       |
       v
Categorization & Prioritization
       |
       v
Assignment to Technician
       |
       v
Investigation & Diagnosis
       |
       v
Resolution
       |
       v
User Confirmation / Closure
```

Quy trình này giúp đảm bảo sự cố được ghi nhận đầy đủ và có thể truy vết quá trình xử lý.

**Incident khác Problem:** Incident Management tập trung khôi phục dịch vụ, trong khi Problem Management tập trung xác định và quản lý nguyên nhân gốc của một hoặc nhiều Incident.

### 2.3. Incident và Service Request

Hai loại yêu cầu này có mục đích khác nhau.

| Tiêu chí | Incident | Service Request |
|---|---|---|
| Bản chất | Sự cố hoặc suy giảm dịch vụ | Yêu cầu cung cấp dịch vụ thông thường |
| Ví dụ | Không truy cập được thư mục chia sẻ | Yêu cầu cài đặt phần mềm |
| Mục tiêu | Khôi phục dịch vụ | Hoàn thành yêu cầu đã được phê duyệt |
| Đánh giá | Tác động, mức độ khẩn cấp | Quy trình và thời hạn cung cấp |

Việc phân loại đúng giúp Service Desk áp dụng quy trình và chính sách xử lý phù hợp.

### 2.4. Service Level Agreement

**SLA (Service Level Agreement)** xác định các cam kết hoặc mục tiêu về mức dịch vụ.

Trong quy trình Helpdesk, hai chỉ tiêu quan trọng là:

**TTO – Time to Own**

Thời gian mục tiêu để một Ticket được tiếp nhận hoặc nhận trách nhiệm xử lý theo cấu hình SLA.

**TTR – Time to Resolve**

Thời gian mục tiêu để giải quyết Ticket theo cấu hình SLA.

Trong lab, chính sách được thiết lập:

| Metric | Target |
|---|---|
| TTO | 1 giờ |
| TTR | 4 giờ |

Các mục tiêu này dùng để thực hành quản lý thời hạn xử lý Ticket; không đại diện cho một cam kết SLA đang áp dụng với khách hàng thực tế.

Việc tính hạn SLA còn phụ thuộc lịch làm việc, điều kiện áp dụng và cấu hình của GLPI.

### 2.5. LDAP & Active Directory Integration

**LDAP (Lightweight Directory Access Protocol)** là giao thức dùng để truy vấn và làm việc với dịch vụ thư mục.

Trong Project, GLPI tích hợp với Active Directory nhằm sử dụng nguồn danh tính tập trung thay vì phải tạo thủ công toàn bộ tài khoản người dùng.

Kiến trúc tích hợp:

```text
       GLPI
        |
        | LDAP Directory Integration
        v
       DC01
   Active Directory
        |
        +--- Domain Users
        +--- User Attributes
        +--- Groups
```

LDAP hỗ trợ truy vấn người dùng, đồng bộ thuộc tính và kiểm tra thông tin xác thực tùy theo cấu hình.

Cần phân biệt:

- **LDAP Integration:** Kết nối ứng dụng với Directory Service.
- **LDAP Authentication:** Xác thực người dùng thông qua Directory Service.
- **User Synchronization:** Nhập hoặc cập nhật dữ liệu người dùng trong ứng dụng.

---

## 3. Solution Architecture

### 3.1. Deployment Architecture

GLPI được triển khai trên một máy Ubuntu riêng trong SERVER Network.

| Component | Configuration |
|---|---|
| GLPI Server | TICKETING |
| IPv4 Address | `10.10.10.40` |
| Network | SERVER – `10.10.10.0/24` |
| Web Server | Apache |
| Database | MariaDB 10.11 |
| Database Name | `glpidb` |
| GLPI Directory | `/var/www/glpi` |
| Domain | `corp.local` |
| Active Directory Server | `10.10.10.10` |
| GLPI URL | `http://glpi.corp.local` |

### 3.2. System Communication

```text
                  USER Network
                  10.10.20.0/24
                         |
                    WIN11-01
                         |
                         | HTTP TCP 80
                         v
                  +--------------+
                  |    pfSense   |
                  | Firewall     |
                  +--------------+
                         |
                  SERVER Network
                  10.10.10.0/24
                         |
                  +--------------+
                  |  TICKETING   |
                  | 10.10.10.40  |
                  | GLPI/Apache  |
                  +--------------+
                    |          |
                    |          +---- MariaDB
                    |                localhost:3306
                    |
                    +--------------- DC01
                                      |
                                      +-- Active Directory
                                      +-- LDAP Authentication
```

Hệ thống có ba nhóm luồng giao tiếp chính:

1. Người dùng truy cập giao diện GLPI qua HTTP.
2. GLPI kết nối Active Directory để thực hiện các hoạt động liên quan đến tài khoản.
3. GLPI truy cập MariaDB cục bộ để lưu trữ thông tin ứng dụng.

MariaDB chỉ lắng nghe tại `127.0.0.1:3306`, hạn chế khả năng truy cập Database trực tiếp từ các máy khác trong mạng.

### 3.3. Design Decisions

| Quyết định | Lý do |
|---|---|
| GLPI chạy trên Ubuntu riêng | Tách chức năng ITSM khỏi Domain Controller |
| MariaDB trên cùng VM | Đơn giản hóa triển khai và quản lý Database trong lab |
| Tích hợp Active Directory | Sử dụng nguồn danh tính đã có |
| Phân vai người dùng và kỹ thuật viên | Mô phỏng quy trình Service Desk |
| SLA TTO/TTR | Thực hành quản lý thời hạn xử lý |
| Tổ chức Ticket theo danh mục | Hỗ trợ phân loại và báo cáo Incident |
| GLPI trong SERVER Network | Tập trung kiểm soát truy cập tới ứng dụng |

---

## 4. Implementation

### 4.1. GLPI Application Deployment

GLPI được triển khai trên Ubuntu với các thành phần:

- Apache Web Server.
- PHP.
- MariaDB.
- GLPI Application.

Thư mục ứng dụng:

`/var/www/glpi`

Database:

`glpidb`

Apache Virtual Host:

`/etc/apache2/sites-available/glpi.conf`

Website được phục vụ qua HTTP TCP 80.

Trong phạm vi lab, HTTPS chưa được triển khai. Đây là giới hạn bảo mật được ghi nhận thay vì xem cấu hình HTTP là giải pháp phù hợp cho Production.

### 4.2. Service Verification

Trên Ubuntu Ticketing, sử dụng:

```bash
systemctl is-active apache2 mariadb
```

Kết quả đã xác minh:

```text
active
active
```

Kiểm tra các dịch vụ lắng nghe:

```bash
sudo ss -lntp
```

Kết quả chính:

| Service | Listening Address | Port |
|---|---|---|
| Apache | All configured interfaces | TCP 80 |
| MariaDB | `127.0.0.1` | TCP 3306 |

Việc giới hạn MariaDB ở Loopback giúp ngăn truy cập Database trực tiếp qua IP của Ubuntu Server.

### 4.3. Active Directory Integration

GLPI được tích hợp với Domain `corp.local` thông qua LDAP.

Các công việc triển khai bao gồm:

- Khai báo LDAP Directory trong GLPI.
- Thiết lập kết nối tới Active Directory.
- Kiểm thử kết nối LDAP.
- Đồng bộ tài khoản người dùng.
- Kiểm tra thông tin User trong GLPI.
- Sử dụng tài khoản được tích hợp để thực hành quy trình hỗ trợ.

Việc tích hợp giúp GLPI sử dụng thông tin danh tính từ hệ thống Active Directory hiện có.

Thông tin Bind Account, mật khẩu và các dữ liệu xác thực không được công khai trong repository.

**Phạm vi xác minh:** Tích hợp LDAP và quá trình đồng bộ người dùng đã được thực hành. Không công khai các tham số nhạy cảm hoặc suy luận thêm những cơ chế xác thực chưa được kiểm thử.

### 4.4. Service Desk Roles

GLPI sử dụng hệ thống Profile và Permission để xác định các thao tác người dùng được phép thực hiện.

Trong lab, các vai trò được mô phỏng gồm:

| Vai trò | Trách nhiệm |
|---|---|
| Requester | Gửi yêu cầu hỗ trợ và theo dõi Ticket |
| Technician | Tiếp nhận, điều tra và xử lý Ticket |
| Administrator | Quản trị và cấu hình hệ thống ITSM |

Các vai trò này tạo nền tảng cho việc phân công trách nhiệm trong quy trình Incident Management.

### 4.5. Ticket Workflow

Quy trình xử lý Ticket được thực hành theo mô hình:

**Step 1 – Ticket Creation**

Người dùng hoặc kỹ thuật viên tạo Ticket khi có sự cố.

Thông tin Ticket có thể bao gồm:

- Title.
- Description.
- Requester.
- Category.
- Priority.
- Affected Asset.
- Assigned Technician.

**Step 2 – Classification**

Ticket được phân loại theo danh mục và mức độ ưu tiên.

Việc phân loại giúp kỹ thuật viên xác định nhóm vấn đề cần xử lý.

**Step 3 – Assignment**

Ticket được phân công cho kỹ thuật viên phụ trách.

**Step 4 – Investigation**

Kỹ thuật viên thực hiện phân tích sự cố, kiểm tra hệ thống và ghi lại kết quả.

**Step 5 – Follow-up & Task**

Thông tin trao đổi, công việc thực hiện và quá trình chẩn đoán được ghi nhận trên Ticket.

**Step 6 – Solution**

Sau khi xử lý, kỹ thuật viên ghi lại nguyên nhân hoặc phương án khắc phục theo kết quả xác minh.

**Step 7 – Resolution & Closure**

Ticket được chuyển sang trạng thái giải quyết và đóng theo quy trình áp dụng.

Quá trình xử lý trên giúp tạo lịch sử hoạt động phục vụ kiểm toán nội bộ và tham khảo khi sự cố tương tự xảy ra.

### 4.6. SLA Configuration

Trong môi trường lab, SLA được thiết lập với các mục tiêu:

| SLA Metric | Target |
|---|---|
| Time to Own | 1 giờ |
| Time to Resolve | 4 giờ |

Việc thiết lập SLA nhằm mô phỏng các yếu tố quản trị dịch vụ như:

- Thời hạn tiếp nhận Ticket.
- Thời hạn giải quyết.
- Theo dõi Ticket có nguy cơ quá hạn.
- Đánh giá tiến độ xử lý theo mục tiêu đặt ra.

Dự án chưa chứng minh việc đáp ứng SLA trên một khối lượng lớn Ticket thực tế.

---

## 5. Operational Example – Incident Management

Một trường hợp được ghi nhận trong quá trình triển khai Project là sử dụng GLPI để quản lý các công việc hỗ trợ máy trạm.

### Example 01 – Remote IT Support

Trong Phase 4, Ticket #6 được sử dụng để ghi nhận hoạt động hỗ trợ từ xa.

Các thành phần liên quan:

- Endpoint: WIN11-01.
- Remote Access: OpenVPN.
- Remote Support Protocol: RDP.
- ITSM Platform: GLPI.

Sau khi kết nối VPN và kiểm tra khả năng truy cập Remote Desktop, kỹ thuật viên ghi nhận hoạt động hỗ trợ trong GLPI.

Trường hợp này minh họa việc kết hợp **Remote Technical Support với Ticket Documentation**.

### Example 02 – Monitoring Incident

Trong Phase 5, Zabbix phát hiện tình trạng CPU Usage cao khi thực hiện bài kiểm thử tạo tải.

Kết quả Monitoring được sử dụng để lập Ticket:

**Ticket #7 – WIN11-01 – High CPU Utilization Detected by Zabbix**

Thông tin chính:

| Attribute | Value |
|---|---|
| Source | Zabbix Monitoring |
| Affected Endpoint | WIN11-01 |
| Incident Category | Infrastructure > Monitoring & Performance |
| Priority | Medium |
| Assigned Technician | Technician IT |

Các thành phần đã được thực hiện trong Ticket:

- Follow-up.
- Task.
- Solution.

Sau khi CPU Load kết thúc, Zabbix ghi nhận Recovery Event.

**Lưu ý:** Ticket được tạo và ghi nhận thủ công dựa trên kết quả Monitoring, chưa có tích hợp tự động Zabbix–GLPI thông qua API.

Hai ví dụ này chứng minh GLPI được sử dụng trong các quy trình vận hành thực tế của lab, không chỉ dừng lại ở bước cài đặt phần mềm.

---

## 6. Technical Challenges

### 6.1. Identity Integration

Việc tích hợp GLPI với Active Directory yêu cầu xác định chính xác nguồn danh tính, thông tin tài khoản, cách truy vấn người dùng và quyền sử dụng hệ thống.

Các bước xác minh cần phân biệt:

- LDAP Connection.
- User Synchronization.
- User Authentication.
- Profile Assignment.

Một kết nối LDAP thành công không tự động chứng minh mọi tài khoản đã được đồng bộ và phân quyền chính xác.

### 6.2. Network Access Control

Do GLPI nằm trong SERVER Network, người dùng từ USER Network cần được cho phép truy cập qua pfSense.

Sau khi áp dụng UFW, một phép kiểm thử đã được thực hiện từ WIN11-01:

```text
SourceAddress    : 10.10.20.104
RemoteAddress    : 10.10.10.40
RemotePort       : 80
TcpTestSucceeded : True
```

Điều này xác nhận WIN11-01 tiếp tục thiết lập kết nối TCP đến Apache sau khi UFW được kích hoạt.

### 6.3. Server Firewall Hardening

Ubuntu Ticketing ban đầu chưa bật UFW.

Ở giai đoạn Final Security Validation, UFW được cấu hình theo nguyên tắc **Default Deny Incoming**.

Chính sách được áp dụng:

| Source | Destination | Service |
|---|---|---|
| `10.10.10.254` | `10.10.10.40` | SSH TCP 22 |
| `10.10.10.0/24` | `10.10.10.40` | HTTP TCP 80 |
| `10.10.20.0/24` | `10.10.10.40` | HTTP TCP 80 |
| `10.10.30.0/24` | `10.10.10.40` | HTTP TCP 80 |

Sau khi kích hoạt UFW, Windows Host vẫn có thể kết nối SSH và HTTP.

Kết quả kiểm tra từ Windows Host:

```text
TCP 22   : True
TCP 80   : True
TCP 3306 : False
```

MariaDB cũng được xác nhận chỉ lắng nghe trên Loopback.

Đây là bước hardening được thực hiện sau khi triển khai ứng dụng, nhằm giảm bề mặt truy cập vào máy chủ GLPI.

---

## 7. Validation & Test Results

| Test ID | Test Case | Result |
|---|---|---|
| ITSM-01 | Apache Service Running | PASS |
| ITSM-02 | MariaDB Service Running | PASS |
| ITSM-03 | GLPI Application Deployment | PASS |
| ITSM-04 | Active Directory LDAP Integration | PASS |
| ITSM-05 | User Synchronization | PASS |
| ITSM-06 | Ticket Creation | PASS |
| ITSM-07 | Ticket Assignment | PASS |
| ITSM-08 | Follow-up, Task & Solution | PASS |
| ITSM-09 | SLA TTO/TTR Configuration | PASS |
| ITSM-10 | HTTP TCP 80 from WIN11-01 | PASS |
| ITSM-11 | Ubuntu UFW Configuration | PASS |
| ITSM-12 | MariaDB Remote TCP Access Restriction | PASS |

Các trạng thái PASS phản ánh các chức năng đã được triển khai và kiểm tra trong lab, không đồng nghĩa với một cuộc kiểm thử hiệu năng, tải hoặc bảo mật toàn diện.

---

## 8. Security Considerations

### 8.1. Identity Management

Active Directory được sử dụng làm nguồn quản lý danh tính, giảm nhu cầu duy trì các danh sách tài khoản riêng biệt.

### 8.2. Role-Based Access Control

GLPI Profile và Permission hỗ trợ giới hạn chức năng theo vai trò.

### 8.3. Network Segmentation

GLPI được đặt trong SERVER Network; người dùng truy cập qua các tuyến và chính sách Firewall đã cấu hình.

### 8.4. Database Protection

MariaDB chỉ lắng nghe tại địa chỉ `127.0.0.1`.

Database không được mở trực tiếp cho các máy trạm.

### 8.5. Host Firewall

UFW được kích hoạt và sử dụng Default Deny Incoming.

### 8.6. HTTP Limitation

GLPI trong lab sử dụng HTTP thay vì HTTPS.

Do đó, môi trường này không được xem là phù hợp để truyền dữ liệu đăng nhập nhạy cảm qua các mạng không đáng tin cậy.

Việc triển khai HTTPS và quản lý Certificate là hạng mục cải tiến cần thực hiện trước khi cân nhắc áp dụng trong môi trường Production.

---

## 9. Technical Outcomes

Phase 2 đã xây dựng được nền tảng ITSM phục vụ quy trình hỗ trợ kỹ thuật trong môi trường lab.

Các kết quả chính:

- Triển khai GLPI với Apache và MariaDB.
- Tích hợp nguồn danh tính Active Directory.
- Mô phỏng vai trò Requester và Technician.
- Thiết lập quy trình quản lý Ticket.
- Áp dụng SLA TTO và TTR.
- Sử dụng GLPI để ghi nhận hoạt động Remote Support và Monitoring Incident.
- Cấu hình Firewall bảo vệ máy chủ ứng dụng.
- Xác minh kết nối giữa Endpoint và GLPI sau khi thực hiện Hardening.

### Skills Demonstrated

- IT Service Management.
- Incident Management.
- Helpdesk Ticket Workflow.
- LDAP & Active Directory Integration.
- SLA Configuration.
- Linux Server Administration.
- Apache & MariaDB Operations.
- Firewall Configuration.
- Technical Incident Documentation.

---

## 10. Scope & Limitations

Dự án tập trung vào việc triển khai và thực hành GLPI trong môi trường lab có quy mô nhỏ.

Những chức năng chưa nằm trong phạm vi:

- Multi-tenant Enterprise Service Desk.
- Tích hợp Email-to-Ticket hoàn chỉnh.
- Tự động tạo Ticket từ Zabbix thông qua API.
- High Availability cho GLPI.
- Enterprise Reporting trên khối lượng Ticket lớn.
- Kiểm thử tải và hiệu năng hệ thống.
- HTTPS Production Deployment.

Hệ thống được sử dụng để minh họa quy trình Helpdesk và Incident Management, không đại diện cho một nền tảng ITSM đang vận hành trong doanh nghiệp thực tế.

---

## 11. Conclusion

Phase 2 mở rộng môi trường hạ tầng từ Project "Enterprise-IT-Infrastructure-Lab" thành một hệ thống có khả năng quản lý hoạt động hỗ trợ người dùng.

GLPI được triển khai trên Ubuntu, tích hợp Active Directory và sử dụng MariaDB để lưu trữ thông tin Ticket.

Các quy trình thực hành bao gồm tiếp nhận, phân loại, phân công, điều tra, ghi nhận phương án xử lý và đóng Ticket.

Thông qua Ticket #6 và Ticket #7, GLPI được sử dụng để ghi nhận các hoạt động hỗ trợ từ xa và phản ứng với cảnh báo Monitoring trong những Phase tiếp theo.

Phase 2 tạo nền tảng cho **Phase 3 – IT Asset Management**, nơi thông tin thiết bị được thu thập bằng GLPI Agent và liên kết với quy trình hỗ trợ kỹ thuật.