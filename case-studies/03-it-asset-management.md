# Case Study 03 – IT Asset Management & Automated Inventory

**Project:** Enterprise IT Support & Infrastructure Operations Lab  
**Phase:** 03 – IT Asset Management  
**Technologies:** GLPI, GLPI Agent, Windows 11 Pro, Active Directory, MariaDB, PowerShell

---

## 1. Project Challenge

Trong môi trường doanh nghiệp, bộ phận IT cần quản lý thông tin các thiết bị đang được sử dụng, bao gồm máy tính, phần cứng, phần mềm và tình trạng vận hành.

Khi số lượng thiết bị tăng lên, việc theo dõi bằng bảng tính hoặc nhập dữ liệu thủ công có thể dẫn đến:

- Thông tin phần cứng không được cập nhật thường xuyên.
- Khó xác định thiết bị đang được sử dụng hoặc cần bảo trì.
- Không biết chính xác máy tính nào đang gặp sự cố.
- Khó liên kết các yêu cầu hỗ trợ với tài sản CNTT cụ thể.
- Thiếu lịch sử bảo trì hoặc thay đổi thông tin tài sản.
- Tốn thời gian thu thập cấu hình máy khi xử lý Incident.

Phase 3 giải quyết bài toán này bằng cách triển khai **GLPI Agent trên Windows Endpoint**, đồng bộ thông tin Inventory vào GLPI và sử dụng dữ liệu đó để quản lý thiết bị.

### Objectives

- Triển khai GLPI Agent trên Windows 11 Pro.
- Thiết lập kết nối từ Endpoint đến GLPI Server.
- Thu thập thông tin Hardware và Software Inventory.
- Đồng bộ thông tin máy tính vào GLPI.
- Quản lý Computer Asset bằng Inventory Number.
- Thực hành các thuộc tính quản lý vòng đời tài sản.
- Liên kết Asset với Ticket hỗ trợ.
- Kiểm tra khả năng nhận diện đúng thiết bị trong hệ thống ITSM.

---

## 2. Technical Concepts

### 2.1. IT Asset Management

**IT Asset Management (ITAM)** là quá trình quản lý tài sản CNTT trong suốt vòng đời sử dụng, từ khi tiếp nhận, triển khai, vận hành cho đến khi thu hồi hoặc thanh lý.

Tài sản CNTT có thể bao gồm:

- Desktop và Laptop.
- Server.
- Network Devices.
- Storage Devices.
- Phần mềm và giấy phép sử dụng.
- Các thiết bị ngoại vi hoặc tài nguyên CNTT khác.

ITAM giúp tổ chức hiểu rõ tài sản nào đang tồn tại, tài sản đó thuộc phạm vi quản lý của ai và đang được sử dụng trong trạng thái nào.

Trong Project, phạm vi ITAM tập trung vào việc quản lý Windows Workstation thông qua GLPI.

### 2.2. Asset Inventory

**Asset Inventory** là tập hợp thông tin mô tả các tài sản CNTT được quản lý.

Đối với máy tính, thông tin Inventory thường bao gồm:

| Nhóm | Ví dụ thông tin |
|---|---|
| Identification | Computer Name, Inventory Number, Serial Number |
| Hardware | CPU, RAM, Storage |
| Operating System | Windows Edition, Version, Build |
| Network | IP Address, Network Adapter |
| Software | Danh sách phần mềm được phát hiện |
| Administration | User, Location, Status, Assigned Group |

Inventory là nguồn dữ liệu hỗ trợ hoạt động IT Support và quản lý tài sản.

Ví dụ, khi người dùng báo máy tính hoạt động chậm, kỹ thuật viên có thể tra cứu Asset để xác định cấu hình phần cứng và hệ điều hành trước khi tiến hành chẩn đoán.

### 2.3. Manual Inventory và Automated Inventory

Có hai cách phổ biến để thu thập thông tin thiết bị.

**Manual Inventory**

Kỹ thuật viên nhập thông tin bằng tay vào hệ thống quản lý.

Ưu điểm:

- Dễ bắt đầu triển khai.
- Phù hợp với một số thuộc tính quản trị không thể tự động phát hiện.

Hạn chế:

- Phụ thuộc vào thao tác con người.
- Có nguy cơ thiếu hoặc sai thông tin.
- Không tự động phản ánh các thay đổi cấu hình.

**Automated Inventory**

Phần mềm Agent thu thập thông tin từ hệ điều hành và chuyển dữ liệu đến hệ thống quản lý.

Ưu điểm:

- Giảm nhập liệu thủ công.
- Thu thập được nhiều thuộc tính kỹ thuật.
- Có khả năng cập nhật dữ liệu khi thiết bị thay đổi.

Hạn chế:

- Cần triển khai và duy trì Agent.
- Phụ thuộc vào kết nối và cấu hình thu thập dữ liệu.
- Không thay thế việc quản lý các thông tin nghiệp vụ như người sở hữu tài sản, lịch sử mua sắm hoặc tình trạng bàn giao.

Trong Phase 3, GLPI Agent đảm nhiệm phần thu thập Inventory tự động.

### 2.4. GLPI Agent

**GLPI Agent** là phần mềm có khả năng thu thập thông tin thiết bị và gửi dữ liệu Inventory đến GLPI.

Đối với Windows Endpoint, Agent có thể thu thập các thông tin như:

- Computer Identification.
- Operating System.
- Hardware Components.
- Installed Software.
- Network Interfaces.
- Các thuộc tính hệ thống được Agent hỗ trợ.

Dữ liệu Inventory được GLPI xử lý và cập nhật thành thông tin tài sản trong hệ thống.

Cần phân biệt:

**GLPI Agent** là thành phần thu thập dữ liệu trên thiết bị.

**GLPI Server** là nền tảng tiếp nhận, xử lý và quản lý thông tin Inventory, Ticket và Asset.

### 2.5. Asset Lifecycle

**Asset Lifecycle** mô tả các giai đoạn mà một tài sản CNTT trải qua trong quá trình sử dụng.

Mô hình tham khảo:

```text id="l1c30k"
Acquisition
    |
    v
Registration
    |
    v
Deployment
    |
    v
Operation
    |
    v
Maintenance
    |
    v
Retirement / Disposal
```

Trong doanh nghiệp, mỗi giai đoạn có thể bao gồm:

- **Acquisition:** Mua sắm hoặc tiếp nhận thiết bị.
- **Registration:** Gán mã tài sản và nhập vào hệ thống quản lý.
- **Deployment:** Cấu hình và bàn giao thiết bị.
- **Operation:** Sử dụng trong hoạt động hằng ngày.
- **Maintenance:** Bảo trì, sửa chữa hoặc thay thế linh kiện.
- **Retirement:** Thu hồi hoặc loại bỏ thiết bị.

Trong Project, các chức năng Lifecycle được thực hành ở mức quản lý thông tin và trạng thái Asset trong GLPI, không mô phỏng đầy đủ nghiệp vụ mua sắm hoặc thanh lý tài sản thực tế.

---

## 3. Solution Architecture

### 3.1. Inventory Architecture

GLPI Agent được triển khai trên máy trạm WIN11-01 thuộc USER Network.

GLPI Server nằm trong SERVER Network.

```text id="2t3uv6"
                 USER Network
                 10.10.20.0/24
                        |
                  +-----------+
                  | WIN11-01  |
                  | Windows 11|
                  |           |
                  | GLPI Agent|
                  +-----------+
                        |
                        | Inventory Upload
                        | HTTP TCP 80
                        v
                   +----------+
                   | pfSense  |
                   | Firewall |
                   +----------+
                        |
                 SERVER Network
                 10.10.10.0/24
                        |
                  +------------+
                  | TICKETING  |
                  |10.10.10.40 |
                  |            |
                  |    GLPI    |
                  +------------+
                        |
                        v
                  +------------+
                  |  MariaDB   |
                  |  glpidb    |
                  +------------+
```

Khi Agent thực hiện Inventory, thông tin thu thập được chuyển về hệ thống GLPI thông qua endpoint đã cấu hình.

GLPI xử lý dữ liệu và cập nhật Asset tương ứng.

### 3.2. Asset Configuration

| Property | Value |
|---|---|
| Managed Endpoint | WIN11-01 |
| Operating System | Windows 11 Pro |
| IP Address | `10.10.20.104` |
| Domain | `corp.local` |
| Inventory Agent | GLPI Agent |
| ITAM Platform | GLPI |
| GLPI Server | `10.10.10.40` |
| Inventory Number | `CORP-WS-001` |
| Asset Type | Computer |

### 3.3. Design Decisions

| Quyết định | Lý do |
|---|---|
| Sử dụng GLPI Agent | Tự động hóa thu thập thông tin thiết bị |
| Quản lý Inventory trong GLPI | Kết hợp dữ liệu Asset với quy trình Helpdesk |
| Giữ WIN11-01 trong Active Directory | Mô phỏng thiết bị đang được quản lý tập trung |
| Gán Inventory Number | Tạo định danh quản trị ổn định |
| Liên kết Ticket với Computer Asset | Hỗ trợ điều tra và theo dõi sự cố theo thiết bị |
| Sử dụng cùng GLPI Server của Phase 2 | Tái sử dụng hạ tầng, tránh phát sinh hệ thống quản lý riêng |

---

## 4. Implementation

### 4.1. GLPI Agent Deployment

GLPI Agent được triển khai trên máy WIN11-01.

Endpoint sử dụng Windows 11 Pro và đã gia nhập Active Directory Domain `corp.local` từ Phase 1.

Quá trình triển khai bao gồm:

1. Cài đặt GLPI Agent trên Windows Endpoint.
2. Cấu hình địa chỉ GLPI Inventory Endpoint.
3. Xác minh Agent hoạt động.
4. Thực hiện Inventory Collection.
5. Kiểm tra dữ liệu đã xuất hiện trong GLPI.
6. Đối chiếu thông tin máy tính với cấu hình thực tế.

Việc triển khai trong Phase 3 được thực hiện cho WIN11-01, chưa phải quá trình triển khai Agent hàng loạt bằng GPO hoặc phần mềm quản trị tập trung.

### 4.2. Connectivity Validation

Trước khi Agent gửi dữ liệu, Endpoint cần kết nối được đến GLPI Server.

Trên WIN11-01, kiểm tra:

```powershell id="gjwvb8"
Test-NetConnection 10.10.10.40 -Port 80
```

Trong giai đoạn Final Security Validation, phép kiểm tra trả về:

```text id="kpuu46"
ComputerName     : 10.10.10.40
RemoteAddress    : 10.10.10.40
RemotePort       : 80
InterfaceAlias   : Ethernet0
SourceAddress    : 10.10.20.104
TcpTestSucceeded : True
```

Điều này xác nhận kết nối TCP từ Windows Endpoint đến GLPI Server vẫn hoạt động sau khi cấu hình UFW.

Kết quả TCP thành công là điều kiện kết nối cần thiết, nhưng không tự nó chứng minh Inventory Upload thành công.

Việc xác nhận Inventory cần thực hiện tại GLPI bằng cách kiểm tra Asset được tạo hoặc cập nhật.

### 4.3. Inventory Collection

Sau khi GLPI Agent hoạt động, hệ thống thu thập các thông tin phần cứng và phần mềm được hỗ trợ.

Các nhóm thông tin cần kiểm tra trong GLPI:

- Computer Name.
- Operating System.
- Processor.
- Memory.
- Storage.
- Network Interfaces.
- Installed Software.

Dữ liệu thu thập giúp hình thành hồ sơ kỹ thuật của máy trạm.

Không nên xem mọi thuộc tính Inventory là luôn chính xác tuyệt đối, bởi chất lượng thông tin còn phụ thuộc Agent, quyền truy cập, hệ điều hành và cách GLPI xử lý dữ liệu.

### 4.4. Inventory Synchronization

Sau quá trình thu thập, dữ liệu WIN11-01 đã được ghi nhận trong GLPI.

Thông tin Asset chính:

| Attribute | Value |
|---|---|
| Computer Name | WIN11-01 |
| Asset Category | Computer |
| Inventory Number | CORP-WS-001 |
| Platform | Windows 11 Pro |
| Domain | corp.local |
| Source | GLPI Agent Inventory |

Trong lab, Asset WIN11-01 được ghi nhận là Computer Asset ID 1 trong GLPI.

**Phân biệt Asset ID và Inventory Number:**

- **Asset ID:** Định danh bản ghi nội bộ do GLPI quản lý.
- **Inventory Number:** Mã quản trị tài sản được gán theo quy ước của tổ chức.

Asset ID có thể thay đổi khi dữ liệu được di chuyển hoặc tạo lại trên hệ thống khác.

Inventory Number được sử dụng để nhận diện tài sản theo quy trình quản lý của tổ chức.

### 4.5. Asset Identification

Project sử dụng mã Inventory:

`CORP-WS-001`

Trong đó:

| Thành phần | Ý nghĩa quy ước |
|---|---|
| CORP | Phạm vi tổ chức mô phỏng |
| WS | Workstation |
| 001 | Số thứ tự thiết bị |

Quy ước này hỗ trợ nhận diện tài sản khi thực hiện các thao tác:

- Tra cứu thiết bị.
- Xử lý yêu cầu hỗ trợ.
- Kiểm tra thông tin cấu hình.
- Ghi nhận hoạt động bảo trì.
- Quản lý tài sản theo từng nhóm thiết bị.

Mã Inventory giúp hạn chế việc nhận diện thiết bị chỉ dựa vào hostname, vốn có thể được thay đổi trong quá trình vận hành.

### 4.6. Asset Lifecycle Management

Sau khi Asset xuất hiện trong GLPI, các thuộc tính quản trị có thể được sử dụng để theo dõi vòng đời thiết bị.

Các thông tin quản lý gồm:

- Status.
- Assigned User.
- Location.
- Inventory Number.
- Notes.
- Maintenance Information.

Trong Project, việc thực hành tập trung vào quản lý hồ sơ Asset và sử dụng thông tin đó trong các quy trình hỗ trợ kỹ thuật.

Không có bằng chứng về quy trình mua sắm, phê duyệt tài sản hoặc thanh lý thực tế, nên những nghiệp vụ này được xem là ngoài phạm vi dự án.

---

## 5. Asset-to-Ticket Integration

### 5.1. Problem Context

Trong thực tế, một Ticket Helpdesk thường liên quan đến một thiết bị cụ thể.

Ví dụ:

- Người dùng báo máy tính hoạt động chậm.
- Không truy cập được Network Share.
- Máy tính không nhận Group Policy.
- Một Endpoint phát sinh cảnh báo CPU Usage cao.

Nếu Ticket chỉ chứa mô tả bằng văn bản, kỹ thuật viên có thể mất thêm thời gian xác định thiết bị.

Việc liên kết Ticket với Asset giúp truy cập nhanh thông tin kỹ thuật của máy.

### 5.2. Integration Approach

GLPI cung cấp khả năng gắn đối tượng Asset với Ticket.

Quy trình được sử dụng trong lab:

```text id="d843nl"
User Reports Incident
        |
        v
   GLPI Ticket
        |
        v
Identify Affected Asset
        |
        v
     WIN11-01
     CORP-WS-001
        |
        v
Review Hardware / OS / Software
        |
        v
Investigation & Resolution
```

Việc kết hợp ITAM và ITSM tạo mối liên hệ giữa tài sản đang được quản lý và các sự cố đã xảy ra.

### 5.3. Operational Example

Trong Phase 5, WIN11-01 là đối tượng được Zabbix giám sát và phát sinh CPU High Usage Alert.

GLPI Ticket #7 được tạo để ghi nhận Incident liên quan đến WIN11-01.

Thông tin liên quan:

| Attribute | Value |
|---|---|
| Incident | High CPU Utilization |
| Endpoint | WIN11-01 |
| Inventory Number | CORP-WS-001 |
| Monitoring Platform | Zabbix |
| ITSM Platform | GLPI |
| Ticket | #7 |

Trường hợp này thể hiện cách sử dụng hồ sơ thiết bị trong quá trình xử lý Incident.

Thông tin Monitoring Event và quy trình Ticket #7 được phân tích chi tiết trong Case Study 05.

---

## 6. Technical Challenges & Operational Considerations

### 6.1. Agent Connectivity

Do WIN11-01 và GLPI Server nằm ở hai subnet khác nhau, quá trình gửi Inventory phụ thuộc vào khả năng kết nối liên mạng.

Các thành phần cần kiểm tra khi Inventory không đồng bộ:

- GLPI Agent Service.
- Inventory Endpoint Configuration.
- DNS Resolution.
- pfSense Firewall Rules.
- Ubuntu UFW Rules.
- Apache Web Service.
- GLPI Inventory Processing.

Không nên kết luận lỗi nằm ở Agent chỉ vì dữ liệu chưa xuất hiện trong GLPI.

### 6.2. Duplicate Asset Prevention

Khi Agent đồng bộ dữ liệu, cần quan tâm đến khả năng tạo bản ghi thiết bị trùng lặp.

Các yếu tố có thể ảnh hưởng đến quá trình nhận diện Asset:

- Computer Name.
- Serial Number.
- UUID.
- Existing Inventory Records.
- Import và Matching Rules.

Trong môi trường doanh nghiệp, việc tạo Asset trùng có thể dẫn đến sai lệch số lượng thiết bị và lịch sử quản lý.

Phase 3 tập trung xác minh WIN11-01 được ghi nhận đúng trong GLPI. Chưa thực hiện kiểm thử chuyên biệt về xử lý Asset trùng lặp.

### 6.3. Asset Data Accuracy

Thông tin Inventory phản ánh trạng thái mà Agent có thể thu thập tại thời điểm chạy.

Do đó, hệ thống quản lý tài sản cần phân biệt:

- Thông tin tự động phát hiện.
- Thông tin do người quản trị khai báo.
- Trạng thái sử dụng hoặc phân công tài sản.
- Lịch sử thay đổi và bảo trì.

Dữ liệu từ Agent không thay thế hoàn toàn các bước kiểm kê thực tế.

### 6.4. Security Considerations

Agent phải được cấu hình giao tiếp đúng với GLPI Server được tin cậy.

Trong lab, dữ liệu Inventory được gửi qua HTTP trong mạng nội bộ.

Đối với môi trường Production, cần xem xét HTTPS, xác thực phù hợp, kiểm soát truy cập, bảo vệ dữ liệu thiết bị và chính sách quản lý Agent.

---

## 7. Validation & Test Results

| Test ID | Test Case | Result |
|---|---|---|
| ITAM-01 | GLPI Agent được cài trên WIN11-01 | PASS |
| ITAM-02 | Endpoint kết nối được GLPI Server | PASS |
| ITAM-03 | Inventory được ghi nhận trong GLPI | PASS |
| ITAM-04 | Computer Asset WIN11-01 được tạo | PASS |
| ITAM-05 | Inventory Number CORP-WS-001 | PASS |
| ITAM-06 | Xác minh thông tin hệ điều hành | PASS |
| ITAM-07 | Sử dụng Asset trong nghiệp vụ hỗ trợ | PASS |

---

## 8. Technical Outcomes

Sau Phase 3, WIN11-01 được quản lý như một tài sản CNTT trong hệ thống GLPI.

Các kết quả chính:

- Triển khai GLPI Agent trên Windows Endpoint.
- Thu thập và đồng bộ Hardware/Software Inventory.
- Ghi nhận Asset WIN11-01 trong GLPI.
- Sử dụng Inventory Number để quản lý danh tính tài sản.
- Kết hợp dữ liệu Inventory với quy trình IT Support.
- Chuẩn bị thông tin Asset phục vụ các Incident phát sinh ở Phase 4 và Phase 5.

### Skills Demonstrated

- IT Asset Management.
- Automated Inventory Collection.
- Windows Endpoint Administration.
- GLPI Agent Deployment.
- Hardware & Software Inventory.
- Asset Identification.
- Asset Lifecycle Fundamentals.
- ITSM & ITAM Integration.
- Network Connectivity Troubleshooting.

---

## 9. Scope & Limitations

Phase 3 tập trung vào một Windows Endpoint trong môi trường lab.

Các giới hạn:

- Chưa triển khai Agent đồng loạt trên nhiều máy trạm.
- Chưa tích hợp với hệ thống Procurement hoặc ERP.
- Chưa thực hiện quản lý License Compliance ở quy mô doanh nghiệp.
- Chưa xây dựng quy trình tài sản với phê duyệt và bàn giao thực tế.
- Chưa kiểm thử chuyên biệt các tình huống Asset Duplicate.
- Chưa triển khai tự động hóa toàn bộ Asset Lifecycle.

Những giới hạn này không ảnh hưởng đến mục tiêu của Phase: chứng minh khả năng triển khai và sử dụng hệ thống Inventory làm nền tảng hỗ trợ quản trị Endpoint.

---

## 10. Conclusion

Phase 3 mở rộng GLPI từ nền tảng Helpdesk thành hệ thống có khả năng quản lý thông tin tài sản CNTT.

Thông qua GLPI Agent, dữ liệu thiết bị WIN11-01 được thu thập và ghi nhận tập trung.

Việc sử dụng Inventory Number `CORP-WS-001` giúp định danh thiết bị trong quy trình quản trị, trong khi khả năng kết hợp Asset và Ticket tạo điều kiện để kỹ thuật viên truy xuất thông tin thiết bị khi xử lý Incident.

Phase 3 đóng vai trò kết nối giữa Endpoint Administration của Phase 1 và các hoạt động Remote IT Support, Monitoring và Incident Response trong những Phase tiếp theo.