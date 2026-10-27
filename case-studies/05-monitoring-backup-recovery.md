# Case Study 05 – Infrastructure Monitoring, Incident Response & Backup Recovery

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Phase:** 05 – Infrastructure Operations & Monitoring  
**Technologies:** Zabbix 7.0, Zabbix Agent 2, Ubuntu, Windows 11, GLPI, MariaDB, PowerShell, Bash, pfSense

---

## 1. Project Challenge

Trong môi trường doanh nghiệp, bộ phận IT không thể chỉ chờ người dùng báo lỗi rồi mới bắt đầu kiểm tra hệ thống.

Những sự cố như CPU quá tải, dịch vụ ngừng hoạt động hoặc thiếu tài nguyên có thể ảnh hưởng đến chất lượng dịch vụ trước khi người dùng kịp gửi yêu cầu hỗ trợ.

Ngoài việc phát hiện và xử lý sự cố, bộ phận IT còn phải bảo vệ dữ liệu của các hệ thống quan trọng và có khả năng phục hồi khi xảy ra lỗi.

Phase 5 giải quyết hai bài toán vận hành:

**Monitoring & Incident Response**

- Theo dõi trạng thái và hiệu năng máy trạm.
- Phát hiện các điều kiện bất thường.
- Ghi nhận thời điểm phát sinh và phục hồi sự cố.
- Kết hợp Monitoring với quy trình xử lý Incident.

**Backup & Recovery**

- Sao lưu Database và dữ liệu ứng dụng GLPI.
- Kiểm tra tính toàn vẹn của bản backup.
- Thực hiện phục hồi thử nghiệm.
- Lưu bản sao backup bên ngoài máy ảo nguồn.
- Xây dựng cơ sở cho Disaster Recovery.

### Objectives

- Triển khai Zabbix Server trên Ubuntu.
- Cài đặt Zabbix Agent 2 trên WIN11-01.
- Cấu hình Active Monitoring.
- Thu thập và quan sát System Metrics.
- Tạo và kiểm thử CPU Usage Trigger.
- Ghi nhận Problem và Recovery Events.
- Lập Incident Ticket trong GLPI.
- Thực hiện GLPI Database Backup.
- Restore Database sang môi trường kiểm thử riêng.
- Xác minh Backup Integrity bằng SHA-256.
- Sao chép dữ liệu backup ra Windows Host.
- Kiểm tra chính sách bảo mật cho các dịch vụ liên quan.

---

## 2. Technical Concepts

### 2.1. Infrastructure Monitoring

Infrastructure Monitoring là hoạt động thu thập, theo dõi và phân tích trạng thái của các thành phần hạ tầng CNTT.

Những đối tượng thường được giám sát gồm:

- Server.
- Workstation.
- Network Devices.
- Operating System Services.
- CPU, Memory và Disk.
- Network Connectivity.
- Application Availability.

Monitoring hỗ trợ bộ phận IT phát hiện sự cố và đánh giá tình trạng hệ thống dựa trên các dữ liệu đo lường.

Trong Project, WIN11-01 được sử dụng làm đối tượng giám sát chính.

### 2.2. Zabbix Monitoring Architecture

Zabbix là nền tảng giám sát hạ tầng CNTT, cung cấp khả năng thu thập Metrics, đánh giá điều kiện cảnh báo và ghi nhận Events.

Một số thành phần:

| Component | Responsibility |
|---|---|
| Zabbix Server | Xử lý dữ liệu giám sát và điều kiện Trigger |
| Zabbix Agent 2 | Thu thập dữ liệu trên thiết bị |
| Host | Đối tượng được quản lý trong Zabbix |
| Item | Chỉ số hoặc dữ liệu được giám sát |
| Template | Tập hợp Items, Triggers và cấu hình tái sử dụng |
| Trigger | Biểu thức xác định trạng thái Problem |
| Event | Bản ghi về thay đổi trạng thái |
| Action | Cơ chế thực hiện tác vụ khi thỏa điều kiện, nếu được cấu hình |

Trong Phase 5, Zabbix được sử dụng để thu thập Metrics từ Windows Endpoint và thực hiện bài kiểm thử cảnh báo CPU.

### 2.3. Active Checks và Passive Checks

Zabbix Agent hỗ trợ hai phương thức thu thập phổ biến.

**Passive Checks**

Zabbix Server hoặc Proxy gửi yêu cầu đến Agent, sau đó Agent trả về dữ liệu.

Mô hình đơn giản:

```text
Zabbix Server → Agent → Response
```

**Active Checks**

Agent chủ động liên hệ Zabbix Server hoặc Proxy để nhận danh sách kiểm tra và gửi dữ liệu thu thập được.

Mô hình:

```text
Zabbix Agent → Zabbix Server
```

Đối với Zabbix Active Checks, TCP 10051 là cổng thường được sử dụng để Agent kết nối đến Zabbix Server.

Trong Project, WIN11-01 sử dụng Active Checks để gửi dữ liệu về Zabbix Server.

### 2.4. Metrics, Items và Triggers

**Metric** là giá trị đo lường phản ánh một trạng thái của hệ thống, ví dụ CPU Utilization.

**Item** là đối tượng cấu hình trong Zabbix dùng để thu thập một loại dữ liệu.

**Trigger** là biểu thức logic đánh giá dữ liệu Items nhằm xác định khi nào điều kiện bất thường xảy ra.

Ví dụ:

```text
Average CPU Usage > 20%
```

Nếu điều kiện đúng theo biểu thức đã cấu hình, Trigger có thể chuyển sang trạng thái PROBLEM.

Khi điều kiện không còn đúng và thỏa logic phục hồi, Zabbix có thể chuyển trạng thái sang OK và ghi nhận Recovery Event.

### 2.5. Incident Response

Incident Response trong phạm vi vận hành hạ tầng bao gồm các hoạt động:

1. Phát hiện dấu hiệu bất thường.
2. Xác định hệ thống bị ảnh hưởng.
3. Đánh giá mức độ nghiêm trọng.
4. Điều tra nguyên nhân.
5. Thực hiện hành động khắc phục.
6. Kiểm tra hệ thống phục hồi.
7. Ghi nhận kết quả xử lý.

Trong Project, Zabbix cung cấp Monitoring Events, còn GLPI được sử dụng để ghi nhận Incident và quá trình xử lý.

### 2.6. Backup và Restore

**Backup** là quá trình tạo bản sao dữ liệu để có thể sử dụng khi dữ liệu gốc bị mất, hỏng hoặc cần phục hồi.

**Restore** là quá trình sử dụng bản backup để khôi phục dữ liệu.

Một bản backup được tạo thành công không đồng nghĩa nó chắc chắn có thể phục hồi.

Do đó, cần kiểm thử:

- Backup File Integrity.
- Database Import.
- Database Structure.
- Data Consistency.
- Application Recovery, nếu nằm trong phạm vi.

Trong Project, quy trình phục hồi được kiểm thử ở mức Database, không phải toàn bộ ứng dụng GLPI.

### 2.7. RPO và RTO

**Recovery Point Objective (RPO)** là mức mất dữ liệu tối đa mà tổ chức có thể chấp nhận, thường được biểu diễn bằng thời gian.

**Recovery Time Objective (RTO)** là thời gian mục tiêu để khôi phục dịch vụ sau sự cố.

Ví dụ, một tổ chức đặt RPO 24 giờ cần thiết kế quy trình sao lưu phù hợp để mức mất dữ liệu không vượt quá mục tiêu trong những kịch bản đã xác định.

Trong Project, RPO và RTO chưa được thiết lập và đo lường chính thức.

Dự án tập trung vào kiểm thử tính khả dụng của Database Backup và khả năng phục hồi dữ liệu.

### 2.8. SHA-256 Integrity Verification

SHA-256 là thuật toán băm tạo giá trị digest 256-bit từ dữ liệu đầu vào.

Trong quy trình Backup, SHA-256 được sử dụng để phát hiện sự thay đổi của file khi sao chép hoặc lưu trữ.

Nếu hai bản file có cùng giá trị SHA-256, đây là bằng chứng mạnh cho thấy nội dung file không thay đổi ngoài ý muốn.

Tuy nhiên, SHA-256 không tự động cung cấp:

- Encryption.
- Confidentiality.
- Access Control.
- Tính xác thực của nguồn file nếu checksum cũng có thể bị thay đổi.

Do đó, Integrity Verification cần kết hợp với việc quản lý quyền truy cập và bảo vệ nơi lưu trữ.

---

## 3. Solution Architecture

### 3.1. Monitoring Architecture

Zabbix Server được triển khai trên Ubuntu thuộc SERVER Network.

WIN11-01 chạy Zabbix Agent 2 trong USER Network.

```text
                USER NETWORK
                10.10.20.0/24
                       |
                +--------------+
                |   WIN11-01   |
                | Windows 11   |
                |              |
                | Zabbix Agent2|
                +--------------+
                       |
                       | Active Checks
                       | TCP 10051
                       v
                   +---------+
                   | pfSense |
                   +---------+
                       |
                SERVER NETWORK
                10.10.10.0/24
                       |
                +--------------+
                |   LINUX01    |
                |10.10.10.30   |
                |              |
                | Zabbix Server|
                +--------------+
                       |
                       v
                Metrics & Events
```

### 3.2. Monitoring Infrastructure

| Component | Configuration |
|---|---|
| Monitoring Server | LINUX01 |
| Server IP | `10.10.10.30` |
| Operating System | Ubuntu |
| Zabbix Version | 7.0.31 |
| Web Server | Nginx |
| Database | PostgreSQL 14 |
| Monitored Host | WIN11-01 |
| Monitored Host IP | `10.10.20.104` |
| Agent | Zabbix Agent 2 |
| Check Type | Active |
| Agent-to-Server Port | TCP 10051 |
| Host Template | Windows by Zabbix agent active |

### 3.3. Backup Architecture

GLPI Server nằm trên Ubuntu Ticketing và sử dụng MariaDB để lưu trữ dữ liệu ứng dụng.

Quy trình Backup được thiết kế như sau:

```text
                TICKETING SERVER
                  10.10.10.40
                        |
              +---------+---------+
              |                   |
         MariaDB DB           GLPI Files
          glpidb              /var/www/glpi
              |                   |
              v                   v
         SQL Backup          TAR Backup
              |                   |
              +---------+---------+
                        |
                        v
                  SHA-256 Check
                        |
                        v
                Local Backup Storage
                        |
                        v
                   SCP Transfer
                        |
                        v
                 WINDOWS HOST
                 Off-VM Backup
```

Bản backup được tạo trên Ubuntu Ticketing, kiểm tra tính toàn vẹn và sao chép ra Windows Host.

Việc giữ bản sao ngoài VM giúp giảm phụ thuộc vào ổ đĩa của máy ảo nguồn.

Tuy nhiên, bản sao trên Windows Host vẫn nằm trong cùng hạ tầng vật lý nên chưa đáp ứng đầy đủ yêu cầu Offsite Backup.

---

## 4. Implementation – Zabbix Monitoring

### 4.1. Zabbix Server Deployment

Zabbix Server được triển khai trên LINUX01.

Các thành phần:

- Zabbix Server 7.0.31.
- PostgreSQL 14.
- Nginx.
- Zabbix Agent 2.

Zabbix Web Interface được sử dụng để quản lý Hosts, Templates, Items, Triggers và Events.

Trong Project, hệ thống Monitoring được triển khai để thu thập và phân tích dữ liệu từ Windows Endpoint.

### 4.2. Windows Agent Deployment

Zabbix Agent 2 được triển khai trên WIN11-01.

Cấu hình Active Checks sử dụng:

```ini
ServerActive=10.10.10.30:10051
Hostname=WIN11-01
```

Các giá trị trên phản ánh cấu hình Agent-to-Server được sử dụng trong lab.

Tên Host của Agent phải khớp với Host được cấu hình trong Zabbix để Active Checks hoạt động đúng.

### 4.3. Firewall Configuration

Do WIN11-01 và Zabbix Server nằm ở hai mạng khác nhau, cần cho phép luồng kết nối phục vụ Active Checks.

Luồng chính:

| Property | Value |
|---|---|
| Source | `10.10.20.104` |
| Destination | `10.10.10.30` |
| Protocol | TCP |
| Destination Port | 10051 |
| Purpose | Zabbix Active Checks |

Trong quá trình triển khai:

- pfSense được cấu hình cho phép kết nối cần thiết.
- Ubuntu UFW trên LINUX01 được cấu hình cho phép nguồn WIN11-01 truy cập TCP 10051.
- Zabbix Agent 2 được cấu hình địa chỉ Zabbix Server.

Kiểm tra trên Windows:

```powershell
Test-NetConnection 10.10.10.30 -Port 10051
```

Kết quả kết nối thành công cho biết TCP Port của Zabbix Server có thể truy cập từ Endpoint.

### 4.4. Host Registration & Template

WIN11-01 được đăng ký trong Zabbix dưới tên:

`WIN11-01`

Template:

`Windows by Zabbix agent active`

Template cung cấp các Items và Triggers giám sát Windows.

Trong quá trình thực hành, Zabbix hiển thị khoảng 66 Items trên Host và thu thập được dữ liệu CPU Utilization.

Một giá trị CPU được quan sát:

`14.5569%`

Đây là giá trị đo tại một thời điểm, không phải CPU Usage trung bình dài hạn của hệ thống.

### 4.5. CPU Monitoring

CPU Utilization được giám sát thông qua Item:

```text
system.cpu.util
```

Zabbix có thể sử dụng dữ liệu từ Item để xây dựng biểu thức đánh giá trạng thái hệ thống.

Trigger mặc định trong Template sử dụng macro cấu hình ngưỡng CPU.

Ví dụ biểu thức đã kiểm tra:

```text
min(/WIN11-01/system.cpu.util,5m)>{$CPU.UTIL.CRIT}
```

Trong lab, giá trị macro hiệu lực được xác định là 90%.

Biểu thức này khác với việc kiểm tra một mẫu CPU tức thời vượt 90%: điều kiện sử dụng giá trị nhỏ nhất trong cửa sổ năm phút.

Vì vậy, một đợt tăng CPU ngắn có thể chưa đủ để kích hoạt Trigger mặc định.

---

## 5. Controlled CPU Alert Simulation

### 5.1. Test Objective

Mục tiêu của bài kiểm thử là xác nhận Zabbix có thể:

- Thu thập CPU Metrics.
- Đánh giá Trigger Expression.
- Tạo Problem Event khi vượt ngưỡng.
- Ghi nhận Recovery khi điều kiện không còn tồn tại.

Bài kiểm thử được thực hiện có kiểm soát trên WIN11-01.

### 5.2. Lab Trigger

Một Trigger thử nghiệm được tạo với tên:

`LAB - WIN11-01 CPU High Usage`

Biểu thức:

```text
avg(/WIN11-01/system.cpu.util,1m)>20
```

Giải thích:

- `system.cpu.util`: Item CPU Utilization.
- `avg(...)`: Tính trung bình các giá trị trong khoảng thời gian xác định.
- `1m`: Cửa sổ một phút.
- `>20`: Điều kiện Trigger khi CPU Usage trung bình vượt 20%.

Ngưỡng 20% chỉ được sử dụng để kiểm thử trong lab, không phải mức cảnh báo CPU đề xuất cho môi trường Production.

### 5.3. Test Execution

CPU Load được tạo có kiểm soát bằng PowerShell Jobs trên WIN11-01.

Trong bài kiểm thử, hai Jobs được sử dụng để tạo tải trong khoảng 90 giây.

Sau đó, dữ liệu CPU được Zabbix Agent thu thập và chuyển về Zabbix Server.

### 5.4. Problem Event

Kết quả đã ghi nhận:

**PROBLEM:** 22:53:41

Zabbix xác định điều kiện Trigger được thỏa mãn và tạo Problem Event.

### 5.5. Recovery Event

Sau khi CPU Load kết thúc và CPU Usage giảm, Trigger trở về trạng thái bình thường.

**RECOVERY:** 22:57:41

Khoảng thời gian giữa hai Event là bốn phút.

Việc Recovery xuất hiện sau khi tải kết thúc phù hợp với cơ chế lấy mẫu, đánh giá cửa sổ dữ liệu và điều kiện Trigger.

Khoảng thời gian này không nên được diễn giải là thời gian xử lý sự cố trung bình của hệ thống.

### 5.6. Test Result

| Test | Result |
|---|---|
| CPU Metrics Collection | PASS |
| Lab Trigger Evaluation | PASS |
| Problem Event Generation | PASS |
| Recovery Event Generation | PASS |
| Controlled CPU Load Simulation | PASS |

Sau khi bài kiểm thử hoàn thành, Lab Trigger được vô hiệu hóa để tránh tạo thêm cảnh báo không cần thiết.

---

## 6. Integration with GLPI Incident Management

### 6.1. Monitoring-to-Incident Workflow

Trong Project, Zabbix và GLPI được sử dụng kết hợp trong một quy trình xử lý sự cố.

```text
Zabbix CPU Monitoring
          |
          v
CPU High Usage Detected
          |
          v
Problem Event Recorded
          |
          v
GLPI Ticket Created Manually
          |
          v
Incident Investigation
          |
          v
Follow-up / Task / Solution
          |
          v
Zabbix Recovery Verified
          |
          v
Incident Documentation Completed
```

### 6.2. GLPI Ticket #7

Ticket được tạo với nội dung:

**WIN11-01 – High CPU Utilization Detected by Zabbix**

Thông tin chính:

| Attribute | Value |
|---|---|
| Ticket ID | #7 |
| Affected Endpoint | WIN11-01 |
| Category | Infrastructure > Monitoring & Performance |
| Priority | Medium |
| Assigned Technician | Technician IT |
| Detection Source | Zabbix Monitoring |

Các thành phần được hoàn thành trong Ticket:

- Follow-up.
- Task.
- Solution.

Quy trình này thể hiện việc chuyển thông tin Monitoring thành công việc xử lý Incident có ghi nhận.

### 6.3. Integration Scope

Trong môi trường lab:

- Zabbix tự động đánh giá Trigger và ghi nhận Problem/Recovery.
- Kỹ thuật viên kiểm tra Event và tạo Ticket trong GLPI.
- Quá trình điều tra và xử lý được ghi nhận thủ công.

Chưa triển khai tích hợp API tự động tạo GLPI Ticket từ Zabbix Event.

Đây là giới hạn quan trọng nhằm phân biệt giữa **Operational Integration** và **Automated System Integration**.

---

## 7. Backup & Restore Implementation

### 7.1. Backup Scope

GLPI được triển khai trên Ubuntu Ticketing.

Các thành phần cần bảo vệ:

| Component | Location / Database |
|---|---|
| MariaDB Database | `glpidb` |
| GLPI Configuration | `/var/www/glpi/config` |
| GLPI Files | `/var/www/glpi/files` |
| GLPI Plugins | `/var/www/glpi/plugins` |
| GLPI Marketplace | `/var/www/glpi/marketplace` |
| Apache Configuration | `/etc/apache2` |
| PHP Configuration | `/etc/php` |
| pfSense Configuration | pfSense Encrypted Configuration Backup |

Phạm vi backup được mở rộng trong Final Security Review nhằm bổ sung cấu hình Apache.

### 7.2. MariaDB Backup

Database Backup được thực hiện bằng `mariadb-dump`.

Lệnh tương ứng với phương pháp sử dụng trong lab:

```bash
sudo bash -c '
set -o pipefail

mariadb-dump \
  --single-transaction \
  --quick \
  --routines \
  --triggers \
  --events \
  --default-character-set=utf8mb4 \
  glpidb | gzip > /var/backups/project2/database/glpidb_backup.sql.gz
'
```

Giải thích các tùy chọn:

| Option | Purpose |
|---|---|
| `--single-transaction` | Tạo snapshot nhất quán đối với các bảng hỗ trợ giao dịch, điển hình là InnoDB |
| `--quick` | Đọc dữ liệu từng dòng thay vì tải toàn bộ kết quả vào bộ nhớ |
| `--routines` | Bao gồm Stored Procedures và Functions |
| `--triggers` | Bao gồm Triggers |
| `--events` | Bao gồm Database Events |
| `--default-character-set=utf8mb4` | Chỉ định Character Set |
| `gzip` | Nén dữ liệu backup |

Đường dẫn lệnh trên là ví dụ thể hiện phương pháp. File backup nghiệm thu thực tế sử dụng tên có Timestamp.

### 7.3. Database Backup Result

Bản backup đã được tạo:

```text
glpidb_20261024_234518.sql.gz
```

Dung lượng:

`154 KB`

File được lưu trong thư mục backup có quyền truy cập hạn chế.

### 7.4. GLPI Files Backup

Các thư mục ứng dụng được sao lưu bằng TAR và GZIP:

- `config`.
- `files`.
- `plugins`.
- `marketplace`.

Bản backup thực tế:

```text
glpi_files_20261024_234728.tar.gz
```

Dung lượng xấp xỉ:

`8 MB`

Backup Database và Application Files được lưu riêng để thuận tiện cho việc kiểm tra và phục hồi từng thành phần.

### 7.5. Database Restore Verification

Để tránh ghi đè Database đang được GLPI sử dụng, bản backup được phục hồi sang Database kiểm thử riêng:

```text
glpidb_restore_test
```

Quá trình kiểm thử bao gồm:

1. Chuẩn bị Database đích riêng.
2. Giải nén SQL Backup.
3. Import dữ liệu vào Database đích.
4. Kiểm tra số lượng Tables.
5. Đối chiếu một số bản ghi quan trọng.
6. Xác minh Database gốc không bị thay thế.

### 7.6. Restore Validation Results

Kết quả thực tế:

| Verification | Source DB | Restored DB |
|---|---|---|
| Database | `glpidb` | `glpidb_restore_test` |
| Table Count | 447 | 447 |
| Ticket #7 | Exists | Exists |
| Asset WIN11-01 | Exists | Exists |
| Inventory Number | CORP-WS-001 | CORP-WS-001 |

**Result: PASS – Database Restore Verification**

Kết quả chứng minh bản SQL Backup đã sử dụng có thể được import vào MariaDB và những dữ liệu được đối chiếu vẫn tồn tại.

Việc hai Database có cùng 447 bảng và một số bản ghi trùng khớp không chứng minh toàn bộ dữ liệu đã được kiểm tra từng dòng hoặc ứng dụng GLPI đã phục hồi hoàn chỉnh.

### 7.7. SHA-256 Verification

Các bản backup được kiểm tra bằng SHA-256.

Phương pháp trên Ubuntu:

```bash
sha256sum backup-file.tar.gz
```

Kiểm tra bằng checksum manifest:

```bash
sha256sum -c checksums.sha256
```

Trong lab, Database Backup và GLPI Files Backup đều trả về kết quả kiểm tra checksum thành công.

Việc xác minh được thực hiện nhằm bảo đảm dữ liệu backup không bị thay đổi ngoài ý muốn sau khi tạo.

### 7.8. Off-VM Backup

Để giảm rủi ro mất dữ liệu khi VM Ticketing bị lỗi, các bản backup GLPI được đóng gói và chuyển sang Windows Host bằng SCP.

File được lưu trên Windows Host:

```text
E:\Project2-Backup\GLPI\GLPI-Backup-20261024.tar.gz
```

Kích thước thực tế:

`8,494,647 bytes`

SHA-256 đã xác minh:

```text
67180DDE78030A4897580B2B2417EA7D5856CF1015B30EFDCFA3B73FFB1D45E3
```

Giá trị băm trên Windows Host được đối chiếu với giá trị từ bản nguồn và cho kết quả trùng khớp.

**Result: PASS – Off-VM Backup Integrity Verification**

Cần lưu ý rằng Windows Host và các máy ảo vẫn cùng phụ thuộc vào một máy tính vật lý.

Do đó, phương án này giúp bảo vệ trước một số tình huống lỗi VM, nhưng chưa thay thế được Offsite Backup hoặc một hệ thống sao lưu độc lập.

### 7.9. pfSense Configuration Backup

Cấu hình pfSense được sao lưu dưới dạng Encrypted Configuration Backup.

File backup được lưu trên Windows Host và xác minh SHA-256.

Giá trị checksum đã ghi nhận:

```text
7836777644475F5D96D4816235F8A57A254A533EBE99B3B6BB994EE4CA562995
```

Việc kiểm tra bằng công cụ XML thông thường không áp dụng trực tiếp vì nội dung backup đã được mã hóa.

Bản backup này không được đưa lên GitHub.

### 7.10. Apache & PHP Configuration Review

Trong giai đoạn Final Validation, cấu hình Apache được bổ sung vào phạm vi sao lưu.

Apache Virtual Host:

```text
/etc/apache2/sites-available/glpi.conf
```

Symbolic Link:

```text
/etc/apache2/sites-enabled/glpi.conf
```

Backup đã tạo:

```text
apache_config_20261025_123311.tar.gz
```

Dung lượng xấp xỉ:

`35 KB`

Archive đã được kiểm tra bằng `tar -tzf` và xác nhận chứa cấu hình Virtual Host.

Bản backup Apache sau đó được sao chép ra Windows Host.

Hệ thống sử dụng PHP 8.3.6 và có thư mục cấu hình `/etc/php`.

**Trạng thái PHP Configuration Backup:** Chưa có output xác nhận riêng trong hồ sơ kiểm thử được tổng hợp.

**Trạng thái checksum bản sao Apache trên Windows Host:** Chưa có kết quả đối chiếu được ghi nhận.

Hai mục này không được đánh dấu PASS khi chưa có bằng chứng tương ứng.

---

## 8. Security Validation

### 8.1. Ubuntu UFW

Ubuntu Ticketing được cấu hình UFW theo chính sách:

```text
Default: deny incoming
Default: allow outgoing
```

Các rule chính:

| Source | Destination | Port |
|---|---|---|
| `10.10.10.254` | `10.10.10.40` | TCP 22 |
| `10.10.10.0/24` | `10.10.10.40` | TCP 80 |
| `10.10.20.0/24` | `10.10.10.40` | TCP 80 |
| `10.10.30.0/24` | `10.10.10.40` | TCP 80 |

Sau khi bật UFW, Windows Host vẫn truy cập được SSH và HTTP.

WIN11-01 cũng kết nối thành công đến GLPI TCP 80.

### 8.2. Database Access Restriction

MariaDB được xác minh lắng nghe tại:

```text
127.0.0.1:3306
```

Kết nối TCP 3306 từ Windows Host thất bại, phù hợp với cấu hình chỉ chấp nhận kết nối Loopback và chính sách Firewall hiện tại.

### 8.3. Backup Access Control

Backup trên Ubuntu được lưu trong các thư mục hạn chế quyền truy cập.

Các file backup chứa dữ liệu ứng dụng hoặc cấu hình không được công khai.

### 8.4. Security Limitations

Các giới hạn cần ghi nhận:

- GLPI vẫn sử dụng HTTP trong lab.
- SSH cho phép Password Authentication và Public Key Authentication.
- Chưa thực hiện Full Application Disaster Recovery.
- Chưa triển khai lịch backup tự động.
- Chưa thực hiện kiểm thử RPO/RTO.
- Chưa có giải pháp Offsite Backup độc lập.

---

## 9. Technical Challenges & Lessons Learned

### 9.1. Zabbix Active Checks Connectivity

**Challenge:** Agent cần gửi Monitoring Data từ USER Network đến Zabbix Server ở SERVER Network.

**Technical Analysis:**

- Kiểm tra cấu hình `ServerActive`.
- Kiểm tra `Hostname`.
- Kiểm tra kết nối TCP 10051.
- Kiểm tra pfSense Firewall.
- Kiểm tra Ubuntu UFW.
- Kiểm tra Host Configuration trong Zabbix.

**Outcome:** Zabbix Active Checks hoạt động và thu thập CPU Metrics từ WIN11-01.

### 9.2. CPU Trigger Testing

**Challenge:** Trigger mặc định sử dụng mức CPU cao và cửa sổ đánh giá dài, không phù hợp với bài kiểm thử tải ngắn.

**Resolution:** Tạo Trigger thử nghiệm riêng với ngưỡng CPU trung bình trên một phút.

**Outcome:** Ghi nhận được Problem và Recovery Event, sau đó vô hiệu hóa Trigger thử nghiệm.

**Lesson Learned:** Cần hiểu chính xác Trigger Expression trước khi kết luận hệ thống Monitoring gặp lỗi.

### 9.3. Backup Verification

**Challenge:** Backup File tồn tại không đủ chứng minh dữ liệu có thể phục hồi.

**Resolution:** Restore SQL Backup vào Database kiểm thử, đối chiếu số lượng bảng và dữ liệu quan trọng.

**Outcome:** Restore thành công 447 bảng cùng Ticket #7 và Asset WIN11-01.

**Lesson Learned:** Restore Verification là thành phần cần thiết của quy trình bảo vệ dữ liệu.

### 9.4. Backup Location

**Challenge:** Backup nằm cùng VM nguồn vẫn phụ thuộc vào trạng thái lưu trữ của VM đó.

**Resolution:** Sao chép dữ liệu Backup sang Windows Host và xác minh SHA-256.

**Outcome:** Bản backup ngoài VM có checksum trùng với bản nguồn.

**Lesson Learned:** Backup cần được thiết kế xét đến phạm vi lỗi có thể xảy ra, không chỉ vị trí của file.

---

## 10. Validation & Test Results

| Test ID | Test Case | Result |
|---|---|---|
| MON-01 | Zabbix Server hoạt động | PASS |
| MON-02 | Zabbix Agent 2 trên WIN11-01 | PASS |
| MON-03 | Active Checks Connectivity | PASS |
| MON-04 | Windows Metrics Collection | PASS |
| MON-05 | CPU Trigger Simulation | PASS |
| MON-06 | Problem Event Generation | PASS |
| MON-07 | Recovery Event Generation | PASS |
| MON-08 | GLPI Ticket #7 Documentation | PASS |
| BAK-01 | MariaDB Backup Creation | PASS |
| BAK-02 | GLPI Files Backup | PASS |
| BAK-03 | Backup SHA-256 Verification | PASS |
| BAK-04 | Database Restore – 447 Tables | PASS |
| BAK-05 | Critical Records Verification | PASS |
| BAK-06 | Off-VM Backup Transfer | PASS |
| BAK-07 | Off-VM SHA-256 Verification | PASS |
| BAK-08 | pfSense Configuration Backup | PASS |
| BAK-09 | Apache Configuration Backup | PASS |
| BAK-10 | PHP Configuration Backup | PASS |
| BAK-11 | Apache Off-VM SHA-256 Verification | PASS |
| SEC-01 | Ubuntu UFW Security | PASS |
| SEC-02 | MariaDB Loopback Binding | PASS |

---

## 11. Technical Outcomes

Phase 5 hoàn thành các hoạt động Monitoring, Incident Response và Backup Verification trong môi trường lab.

### Monitoring

- Triển khai Zabbix Server và Windows Agent.
- Cấu hình Active Checks.
- Thu thập CPU Metrics.
- Kiểm thử Trigger.
- Ghi nhận Problem và Recovery Events.

### Incident Management

- Chuyển kết quả Monitoring thành GLPI Ticket.
- Ghi nhận Follow-up, Task và Solution.
- Kết hợp Monitoring với quy trình Service Desk.

### Backup & Recovery

- Sao lưu GLPI Database.
- Sao lưu Application Files.
- Phục hồi Database vào môi trường kiểm thử riêng.
- Xác minh 447 bảng và các bản ghi quan trọng.
- Kiểm tra Backup Integrity bằng SHA-256.
- Sao chép Backup ra Windows Host.
- Sao lưu cấu hình pfSense và Apache.

### Skills Demonstrated

- Infrastructure Monitoring.
- Zabbix Administration.
- Agent-Based Monitoring.
- Trigger & Event Analysis.
- PowerShell Performance Testing.
- Incident Response.
- GLPI Ticket Documentation.
- Linux System Administration.
- MariaDB Backup & Restore.
- Backup Integrity Verification.
- Shell Scripting.
- Network Firewall Troubleshooting.

---

## 12. Scope & Limitations

Các chức năng đã được kiểm thử trên môi trường lab có quy mô nhỏ.

Những giới hạn chính:

- Chỉ sử dụng một Windows Endpoint làm đối tượng Monitoring chính.
- CPU Alert được kiểm thử bằng tải nhân tạo.
- Chưa đo lường Monitoring Performance ở quy mô lớn.
- Chưa tự động hóa Zabbix-to-GLPI Ticket Creation.
- Chưa triển khai Backup Scheduling định kỳ.
- Chưa kiểm thử Full Application Disaster Recovery.
- Chưa đo RPO và RTO.
- Chưa có Offsite Backup độc lập.
- Chưa kiểm thử các tình huống lỗi lưu trữ nghiêm trọng hoặc mất toàn bộ máy Host.

Kết quả Phase 5 chứng minh khả năng triển khai và kiểm thử các thành phần vận hành cơ bản, không phải xác nhận một hệ thống Production đạt đầy đủ yêu cầu về Availability, Recovery hoặc Security.

---

## 15. Tổng kết

Phase 5 hoàn thiện các chức năng Infrastructure Operations quan trọng cho Project.

Zabbix được triển khai để giám sát Windows Endpoint, thu thập dữ liệu CPU và phát hiện điều kiện bất thường bằng Trigger.

Bài kiểm thử CPU Load tạo ra Problem Event và Recovery Event, sau đó được ghi nhận trong GLPI Ticket #7 để mô phỏng quy trình Incident Response.

Đối với Backup & Recovery, dự án đã tạo bản sao Database và Application Files, xác minh SHA-256 và phục hồi thành công Database GLPI sang môi trường kiểm thử độc lập với 447 bảng.

Việc sao chép dữ liệu ra Windows Host giúp giảm phụ thuộc vào VM nguồn, trong khi các bài kiểm tra Firewall và Database Binding bổ sung lớp bảo vệ cho máy chủ ứng dụng.

Kết quả Phase 5 thể hiện khả năng kết hợp Monitoring, Troubleshooting, Incident Documentation và Backup Verification thành một quy trình vận hành hạ tầng CNTT có thể kiểm chứng.