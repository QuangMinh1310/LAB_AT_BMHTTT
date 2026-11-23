# LAB 6: XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense

## 1. Thông tin sinh viên

| Thông tin | Nội dung |
|---|---|
| Họ và tên | **Nguyễn Quang Minh** |
| MSSV | **1150080105** |
| Lớp | **11DH_THMT** |
| Tên bài thực hành | **LAB 6: XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense** |

## 2. Mục tiêu

- Xây dựng mô hình mạng riêng cho LAB 6 gồm LAN, DMZ và mạng quản trị MGMT.
- Tích hợp pfSense với Active Directory qua LDAP để xác thực người dùng.
- Sử dụng Captive Portal trên LAN và quản trị pfSense qua MGMT.
- Thiết lập Alias, Schedule và firewall rule để kiểm soát truy cập theo máy, dịch vụ và thời gian.
- Kiểm thử chính sách và đối chiếu kết quả với Firewall Log.

## 3. Môi trường thực hành

### 3.1. Phần mềm và phiên bản

| Thành phần | Môi trường sử dụng | Phiên bản |
|---|---|---|
| Nền tảng ảo hóa | VMware Workstation | *Cần bổ sung phiên bản thực tế* |
| Firewall | pfSense | *Cần bổ sung phiên bản thực tế* |
| Domain Controller / DNS | Windows Server (DC-01) | *Cần bổ sung phiên bản Windows Server* |
| Web Server trong DMZ | Windows Server, IIS (DMZ-Web) | *Cần bổ sung phiên bản Windows Server* |
| Máy trạm | Windows 11 (Client A, Client B) | Windows 11; *build chưa xác nhận* |
| Dịch vụ xác thực | Active Directory Domain Services, LDAP, Captive Portal | Theo cấu hình LAB 6 |

> Các mục “Cần bổ sung” được giữ nguyên vì ảnh báo cáo chưa xác nhận chính xác số phiên bản. Có thể kiểm tra trong giao diện **Status > Dashboard** của pfSense, **Help > About** của VMware và lệnh `winver` trên Windows.

### 3.2. Sơ đồ địa chỉ và máy ảo

| Thiết bị | Vai trò | Địa chỉ / mạng |
|---|---|---|
| pfSense-LAB6 (LAN) | Gateway mạng LAN | `10.6.0.1/24` |
| pfSense-LAB6 (DMZ) | Gateway mạng DMZ | `172.16.6.1/24` |
| pfSense-LAB6 (MGMT) | WebGUI quản trị | `192.168.56.254` |
| DC-01 | Domain Controller, DNS, LDAP | `10.6.0.10` |
| Client A | Máy người dùng `sv_a` | `10.6.0.21` |
| Client B | Máy người dùng `sv_b` | `10.6.0.22` |
| DMZ-Web | Máy chủ IIS, mô phỏng các website | `172.16.6.10`, `.20`, `.30`, `.40` |

- Domain Active Directory: **`lab6.test`**.
- OU người dùng: **`LAB6-Users`**; tài khoản kiểm thử: **`sv_a`**, **`sv_b`**.
- Kết nối mạng ảo đã dùng: LAN **VMnet3**, MGMT **VMnet4**, DMZ **LAN Segment**; kiểm tra lại mapping card mạng trước khi chạy.
- Địa chỉ quản trị pfSense: **`https://192.168.56.254`**.

### 3.3. Các website mô phỏng trong DMZ

| Alias | Địa chỉ | Dịch vụ |
|---|---|---|
| `GENERAL_SITE` | `172.16.6.10` | HTTP/80 |
| `JOB_SITE` | `172.16.6.20` | HTTP/80 |
| `SOCIAL_SITE` | `172.16.6.30` | HTTP/80 |
| `RESTRICTED_SITE` | `172.16.6.40` | HTTP/80 |

Đây là các địa chỉ IP mô phỏng danh mục website trong phòng lab, **không phải hệ thống Web Category Filtering thực tế**.

## 4. Cách dựng và khởi chạy môi trường

1. Mở **VMware Workstation**, khởi động các máy ảo: **pfSense-LAB6**, **DC-01**, **DMZ-Web**, **Client A** và **Client B**.
2. Kiểm tra kết nối các card mạng ảo: LAN `10.6.0.0/24`, DMZ `172.16.6.0/24`, MGMT `192.168.56.0/24`; bảo đảm pfSense có IP gateway tương ứng.
3. Trên **DC-01**, kiểm tra AD DS, DNS và domain `lab6.test`; xác nhận OU `LAB6-Users` có hai tài khoản `sv_a`, `sv_b`.
4. Trên **Client A/B**, kiểm tra IP và DNS `10.6.0.10`; bảo đảm hai client liên lạc được với các dịch vụ nội bộ cần thiết.
5. Trên **DMZ-Web**, cấu hình các IP `172.16.6.10/20/30/40` trên card DMZ, gateway `172.16.6.1`; khởi động dịch vụ **IIS (W3SVC)** và kiểm tra HTTP.
6. Truy cập WebGUI pfSense từ mạng **MGMT** tại `https://192.168.56.254`; kiểm tra cấu hình interface.
7. Cấu hình **LDAP Authentication Server** trỏ về AD trên DC-01; thử xác thực `sv_a`, `sv_b` trong **Diagnostics > Authentication**.
8. Cấu hình **Captive Portal** vùng `LAB6_USERS` trên **LAN** và xác thực bằng tài khoản AD.
9. Tạo các **Alias** (`CLIENT_A`, `CLIENT_B`, `POLICY_CLIENTS`, các website và `WEB_PORTS`) cùng **Schedule** (`LUNCH_WINDOW`, `WORK_HOURS`).
10. Thiết lập firewall rule theo các kịch bản bên dưới; nhấn **Save > Apply Changes** sau khi chỉnh sửa, rồi tạo kết nối mới để kiểm thử.

**Kiểm tra nhanh trên DMZ-Web (PowerShell):**

```powershell
Get-Service W3SVC
curl.exe -I --max-time 8 http://172.16.6.10/
```

**Kiểm tra nhanh trên Client A/B (PowerShell):**

```powershell
ipconfig /all
curl.exe -I --max-time 8 --noproxy "*" http://172.16.6.10/
```

> Nếu nhận HTTP 302 chuyển về Captive Portal, cần đăng nhập trước; đó chưa phải kết quả truy cập website thành công.

## 5. Các tình huống đã thực hiện và kết quả PASS/FAIL

**Quy ước:** **PASS** = có bằng chứng kiểm thử phù hợp với mục tiêu; **PARTIAL** = đã cấu hình hoặc kiểm thử một phần, chưa đủ bằng chứng xác nhận toàn bộ; **FAIL** = kết quả thực tế trái yêu cầu. Bảng này tổng hợp theo ảnh trong `LAB6.docx` và các bước kiểm thử đã thực hiện, không tự nhận các phép thử chưa có bằng chứng là PASS.

| STT | Tình huống | Chính sách / kiểm thử | Kết quả |
|---|---|---|---|
| 1 | Client A bị chặn JOB_SITE | Block `CLIENT_A` → `JOB_SITE` (`172.16.6.20:80`) | **PASS** – đã cấu hình và ghi nhận kiểm thử trong quá trình thực hành |
| 2 | Cấm RESTRICTED_SITE cho A và B | Block `POLICY_CLIENTS` → `RESTRICTED_SITE` (`172.16.6.40:80`) | **PASS** – báo cáo có ảnh client không truy cập được website |
| 3 | Client B chỉ truy cập SOCIAL_SITE trong giờ nghỉ | Pass theo `LUNCH_WINDOW` đặt trên Block không lịch; đã thử lịch tạm `TEST_NOW` | **PASS khi kiểm thử lịch tạm** – cần bảo đảm đã khôi phục `LUNCH_WINDOW` |
| 4 | Chỉ cho HTTP/HTTPS trong giờ hành chính | Pass `POLICY_CLIENTS` → `WEB_PORTS` theo `WORK_HOURS`, Block lưu lượng còn lại | **PARTIAL** – đã thử truy cập được với lịch thử và bị chặn ngoài lịch; chưa có đủ bằng chứng riêng cho tất cả giao thức khác |
| 5 | Cô lập DMZ khỏi LAN nhưng vẫn cho LAN truy cập site cần thiết | Block `DMZ net` → `LAN net` trên interface DMZ; thử DMZ-Web → DC-01 TCP/389 và ICMP | **PASS** – `TcpTestSucceeded: False` và Firewall Log xác nhận `KB5_BLOCK_DMZ_TO_LAN`; chiều LAN → DMZ có ảnh IIS hiển thị |

### 5.1. Kịch bản 1 – Client A bị chặn JOB_SITE

- Nguồn: `CLIENT_A` (`10.6.0.21`).
- Đích: `JOB_SITE` (`172.16.6.20`), TCP/80.
- Hành động: **Block**.
- Mục tiêu: Client A không truy cập JOB_SITE; vẫn có thể sử dụng các dịch vụ được cho phép khác.

### 5.2. Kịch bản 2 – Chặn RESTRICTED_SITE cho cả A và B

- Nguồn: `POLICY_CLIENTS` (Client A và Client B).
- Đích: `RESTRICTED_SITE` (`172.16.6.40`), TCP/80.
- Rule: `KB2_BLOCK_RESTRICTED`.
- Kết quả báo cáo: trình duyệt không truy cập được trang bị hạn chế.

### 5.3. Kịch bản 3 – SOCIAL_SITE theo giờ nghỉ

- Nguồn: `CLIENT_B` (`10.6.0.22`).
- Đích: `SOCIAL_SITE` (`172.16.6.30`), TCP/80.
- `KB3_ALLOW_SOCIAL_LUNCH`: **Pass**, lịch `LUNCH_WINDOW` (thứ Hai–thứ Sáu, 11:30–13:00).
- `KB3_BLOCK_SOCIAL_OUTSIDE_LUNCH`: **Block**, không lịch, đặt **ngay dưới** rule Pass.
- Đã dùng `TEST_NOW` để thử tại thời điểm thực hành; sau kiểm thử phải chuyển lại `LUNCH_WINDOW`.

### 5.4. Kịch bản 4 – HTTP/HTTPS trong giờ hành chính

- Nguồn: `POLICY_CLIENTS`.
- `KB4_ALLOW_WEB_WORK_HOURS`: **Pass**, TCP đến alias `WEB_PORTS` (80, 443), lịch `WORK_HOURS` (thứ Hai–thứ Sáu, 08:00–17:00).
- `KB4_BLOCK_OTHER_TRAFFIC`: **Block**, IPv4 Any, không lịch.
- Rule Pass theo lịch phải đứng trên rule Block tương ứng.
- Khi kiểm thử bằng lịch tạm đang hiệu lực, HTTP truy cập được; khi trả về lịch không hiệu lực tại thời điểm thử, truy cập bị chặn. Cần kiểm tra các rule Pass khác để không vô tình vượt qua chính sách giờ hành chính.

### 5.5. Kịch bản 5 – Cô lập DMZ khỏi LAN

- Interface: **DMZ**.
- Rule `KB5_BLOCK_DMZ_TO_LAN`: **Block**, nguồn `DMZ net`, đích `LAN net`, IPv4 Any; đặt **trên** các rule Pass tổng quát của DMZ.
- Kiểm thử từ DMZ-Web (`172.16.6.10`) đến DC-01 (`10.6.0.10`):

```powershell
Test-NetConnection 10.6.0.10 -Port 389
```

- Kết quả: **`TcpTestSucceeded: False`**, ping không thành công.
- **Firewall Log** ghi nhận rule `KB5_BLOCK_DMZ_TO_LAN` chặn gói tin từ `172.16.6.10` đến `10.6.0.10:389` và ICMP. Đây là bằng chứng xác nhận chính pfSense thực thi chính sách chặn.
- Chiều LAN → DMZ: Client A truy cập trang IIS `http://172.16.6.10` khi chính sách LAN cho phép.

## 6. Lỗi gặp phải và cách khắc phục

| Vấn đề | Nguyên nhân / dấu hiệu | Cách khắc phục |
|---|---|---|
| Không truy cập website sau khi thay đổi thứ tự rule Kịch bản 4 | `WORK_HOURS` chưa có hiệu lực tại thời điểm thử hoặc rule Block áp dụng trước | Đặt Pass theo lịch trên Block; dùng lịch thử `TEST_NOW` để kiểm chứng rồi **khôi phục `WORK_HOURS`** |
| Rule DMZ đã kéo lên đầu nhưng kết nối chưa bị chặn như mong đợi | Thay đổi chưa được áp dụng; có thể còn state cũ hoặc kiểm thử sai đích | Nhấn **Apply Changes**, tạo kết nối mới; thử đến đúng IP LAN và kiểm tra Firewall Log |
| `Test-NetConnection` từ DMZ-Web trả `False` | Kết nối không thành công; riêng kết quả `False` chưa xác định nguyên nhân | Lọc Firewall Log theo nguồn `172.16.6.10`, đích `10.6.0.10`; đã xác nhận rule `KB5_BLOCK_DMZ_TO_LAN` thực sự chặn |
| Log chỉ hiện `KB4_BLOCK_OTHER_TRAFFIC` hoặc `Default deny rule` | Đang xem log khác interface, IP đích hoặc thời gian | Lọc đúng **DMZ**, IP nguồn/đích, thời gian mới nhất; tạo lại lưu lượng kiểm thử |
| Trình duyệt chuyển đến trang đăng nhập Captive Portal | Người dùng chưa xác thực hoặc phiên đăng nhập không còn hiệu lực | Đăng nhập bằng tài khoản AD (`sv_a`/`sv_b`), kiểm tra **Status > Captive Portal** rồi thử lại |
| Truy cập bị timeout nhưng chưa biết rule nào chặn | Timeout có thể do firewall, máy chủ hoặc đường mạng | Đối chiếu IP/port, trạng thái dịch vụ IIS và **Status > System Logs > Firewall**; không kết luận chỉ từ màn hình timeout |

## 7. Bằng chứng thực hành

Các ảnh chụp màn hình được tổng hợp trong tài liệu **`LAB6.docx`**:

| Trang tài liệu | Nội dung minh chứng |
|---|---|
| 1 | DMZ-Web, kết nối mạng và rule DMZ ban đầu |
| 2 | OU `LAB6-Users`, hai tài khoản AD và kiểm tra DC-01 |
| 3 | Thiết lập Client A, Client B |
| 4 | Kiểm tra xác thực LDAP thành công |
| 5 | Đăng nhập `sv_a`, trạng thái Captive Portal |
| 6 | Alias và Schedule |
| 7–8 | Kịch bản 2: kết quả Client A/B |
| 9–10 | Kịch bản 3: kết quả truy cập và firewall rule |
| 11–12 | Kịch bản 4: rule theo lịch và kết quả kiểm thử |
| 13–15 | Kịch bản 5: rule LAN/DMZ, `Test-NetConnection` và Firewall Log |

> Nếu đưa README lên GitHub, nên đưa ảnh minh chứng vào thư mục `images/` và chèn bằng đường dẫn tương đối, hoặc đính kèm `LAB6.docx` theo yêu cầu nộp bài. Không đưa mật khẩu tài khoản AD/pfSense lên repository.

## 8. Kết luận

LAB 6 đã xây dựng được mô hình pfSense kết hợp Active Directory, xác thực người dùng qua LDAP/Captive Portal và thực hiện năm tình huống kiểm soát truy cập. Các kịch bản chặn website và cô lập DMZ có bằng chứng kiểm thử; riêng Kịch bản 4 cần bổ sung kiểm thử các giao thức ngoài HTTP/HTTPS nếu yêu cầu nghiệm thu đầy đủ. Kết quả cho thấy thứ tự firewall rule, lịch áp dụng và trạng thái kết nối ảnh hưởng trực tiếp đến hiệu lực chính sách. Firewall Log là nguồn quan trọng để xác nhận nguyên nhân cho phép hoặc chặn lưu lượng.
