# LAB THỰC HÀNH AN TOÀN BẢO MẬT HỆ THỐNG THÔNG TIN

## 1. Thông tin sinh viên

- Họ và tên: Nguyễn Quang Minh
- MSSV: 1150080105
- LAB5: THIẾT LẬP MÔ HÌNH TƯỜNG LỬA pfSense


---

## 2. Phiên bản và môi trường thực hành

- Máy thật: Windows 11
- Phần mềm ảo hóa: VMware Workstation
- Firewall: pfSense CE 2.7.2-RELEASE (amd64)
- Domain Controller: Windows Server
- LAN-Test: Ubuntu Server 24.04 LTS
- Trình duyệt dùng để quản trị pfSense: Google Chrome

Mô hình mạng gồm 3 vùng:

- WAN: nhận địa chỉ IP từ mạng bên ngoài.
- LAN: 10.0.0.0/8
- DMZ: 172.16.0.0/16

Địa chỉ pfSense:

- LAN: 10.0.0.1/8
- DMZ: 172.16.0.1/16
- Domain Controller: 10.0.0.2
- LAN-Test: 10.0.0.3/8, Gateway 10.0.0.1

---

## 3. Cách dựng môi trường

### pfSense

Tạo máy ảo pfSense trên VMware Workstation và cấu hình 3 Network Adapter:

- Adapter 1: Bridged - sử dụng cho WAN.
- Adapter 2: Host-only - sử dụng cho LAN.
- Adapter 3: LAN Segment `dmz-net` - sử dụng cho DMZ.

Gán interface trong pfSense:

- WAN = em0
- LAN = em1
- DMZ = em2

Cấu hình địa chỉ:

- LAN: 10.0.0.1/8
- DMZ: 172.16.0.1/16

Truy cập WebConfigurator thông qua:

https://10.0.0.1

### LAN-Test

Tạo một máy ảo Ubuntu Server với:

- RAM: 1 GB
- Network Adapter: Host-only, cùng mạng LAN với pfSense.
- IP: 10.0.0.3/8
- Gateway: 10.0.0.1
- DNS: 8.8.8.8

### Domain Controller

Domain Controller được đặt trong mạng LAN:

- IP: 10.0.0.2
- Gateway: 10.0.0.1

---

## 4. Các tình huống đã thực hiện

### Tình huống 1: Chặn Ping nhưng vẫn cho phép DNS và Web

Cấu hình Firewall Rules trên LAN:

- Block ICMP từ LAN net đến Any.
- Pass TCP/UDP port 53 để cho phép DNS.
- Pass TCP port 80/443 để cho phép HTTP/HTTPS.

Kiểm tra:

- Ping 8.8.8.8: thất bại.
- Resolve-DnsName example.com -Server 8.8.8.8: thành công.
- curl.exe -4 https://example.com: thành công.

Kết quả: **PASS**

Kết luận: ICMP đã bị chặn nhưng máy trong LAN vẫn có thể phân giải DNS và truy cập Web.

---

### Tình huống 2: Chỉ cho một host cụ thể ra Internet

Mục tiêu là chỉ cho phép Domain Controller 10.0.0.2 truy cập Internet và chặn các host LAN còn lại.

Cấu hình rule theo thứ tự:

1. Pass: Source 10.0.0.2 -> Any.
2. Block: Source LAN net -> Any.

Rule cho phép toàn bộ LAN truy cập Internet được vô hiệu hóa khi kiểm thử.

Kiểm tra:

- Domain Controller 10.0.0.2 ping 8.8.8.8: thành công.
- LAN-Test 10.0.0.3 ping 8.8.8.8: thất bại sau khi áp dụng rule Block.

Kết quả: **PASS**

Kết luận: pfSense cho phép áp dụng chính sách truy cập khác nhau đối với từng host trong cùng mạng LAN.

---

### Tình huống 3: Cô lập DMZ khỏi LAN

Mục tiêu là cho phép DMZ truy cập Internet nhưng không được truy cập vào LAN.

Trước khi cấu hình Block, thực hiện kiểm tra baseline:

- DMZ-Web ping Domain Controller 10.0.0.2.
- Kết quả: thành công.

Sau đó tạo các rule trên interface DMZ theo thứ tự:

1. Block DMZ net -> LAN net.
2. Pass DMZ net -> Any.

Apply Changes và Reset States.

Kiểm tra sau khi cấu hình:

- DMZ-Web ping 10.0.0.2: thất bại.
- DMZ-Web ping 8.8.8.8: thành công.
- Resolve-DnsName example.com: thành công.

Kết quả: **PASS**

Kết luận: DMZ đã được cô lập khỏi LAN nhưng vẫn có khả năng truy cập Internet và sử dụng DNS.

---

## 5. Tổng hợp kết quả

| Tình huống | Nội dung kiểm thử | Kết quả |
|---|---|---|
| 1 | Chặn ICMP nhưng vẫn cho DNS/Web | PASS |
| 2 | Chỉ cho host 10.0.0.2 ra Internet | PASS |
| 3 | Chặn DMZ truy cập LAN nhưng vẫn cho DMZ ra Internet | PASS |

---

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: LAN-Test không truy cập được Internet

Trong quá trình cấu hình Ubuntu LAN-Test, lệnh `apt update` xuất hiện lỗi:

`Temporary failure resolving 'vn.archive.ubuntu.com'`

Đồng thời không thể kết nối đến các địa chỉ Internet.

Nguyên nhân liên quan đến cấu hình mạng/rule trong quá trình thiết lập và kiểm thử.

Cách khắc phục:

- Kiểm tra lại Network Adapter của LAN-Test.
- Đảm bảo LAN-Test nằm cùng mạng LAN với pfSense.
- Cấu hình IP LAN-Test là 10.0.0.3/8.
- Cấu hình Default Gateway là 10.0.0.1.
- Cấu hình DNS là 8.8.8.8.
- Kiểm tra Firewall Rules và NAT trên pfSense.
- Apply Changes và Reset States sau khi thay đổi rule.

Sau khi cấu hình đúng, LAN-Test có thể ping 8.8.8.8 thành công khi firewall cho phép.

### Lỗi 2: Ubuntu thiếu lệnh ping

Khi kiểm tra bằng:

`ping 8.8.8.8`

Ubuntu báo:

`ping: command not found`

Cách khắc phục:

Sau khi kết nối Internet hoạt động, cập nhật repository và cài gói cần thiết để sử dụng lệnh ping.

### Lỗi 3: Rule Block không hoạt động như mong muốn

Trong quá trình thử nghiệm, nếu phía trên rule Block tồn tại một rule Pass tổng quát thì traffic vẫn có thể được cho phép.

Cách khắc phục:

- Kiểm tra thứ tự các Firewall Rules.
- Đặt rule Block/Pass đúng vị trí theo yêu cầu của từng tình huống.
- Disable các rule Pass tổng quát không cần thiết.
- Apply Changes.
- Reset States để các kết nối cũ không ảnh hưởng đến kết quả kiểm thử.

---

## 7. Kết luận

Qua bài lab đã thực hiện được việc xây dựng mô hình mạng gồm WAN, LAN và DMZ bằng pfSense. Các firewall rule được sử dụng để kiểm soát ICMP, DNS, HTTP/HTTPS, giới hạn quyền truy cập Internet theo từng host và cô lập DMZ khỏi LAN.

Kết quả cả ba tình huống kiểm thử đều đạt yêu cầu PASS. Qua bài thực hành có thể thấy thứ tự firewall rule, cấu hình địa chỉ IP, gateway, NAT và việc Reset States có ảnh hưởng trực tiếp đến kết quả kiểm soát lưu lượng trên pfSense.