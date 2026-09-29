\# LAB 4 – NMAP: QUÉT VÀ PHÂN TÍCH MẠNG



\## 1. Thông tin sinh viên



\- Họ và tên: Nguyễn Quang Minh

\- MSSV: 1150080105

\- Tên Lab: LAB4 – Nmap

\- Nội dung: Sử dụng Nmap để thực hiện host discovery, quét cổng, nhận diện dịch vụ và so sánh trạng thái hệ thống trước/sau khi thay đổi cấu hình phòng thủ.



\---



\## 2. Phiên bản môi trường



Môi trường thực hành được xây dựng trên VMware Workstation.



Các máy ảo sử dụng:



\- Kali Linux: máy thực hiện quét Nmap.

\- Metasploitable2-Linux: máy mục tiêu phục vụ thực hành quét và nhận diện dịch vụ.

\- Windows 11 x64: máy mục tiêu sử dụng trong tình huống Before/After.

\- Kiểu mạng: Host-Only.

\- Công cụ chính: Nmap 7.99.

\- Hệ điều hành máy thật: Windows 11.



Một số địa chỉ IP sử dụng trong quá trình thực hành:



\- Metasploitable2: `192.168.23.130`

\- Windows 11 VM: `192.168.23.128`



\---



\## 3. Cách dựng môi trường



1\. Cài đặt VMware Workstation trên máy thật.

2\. Tạo và khởi động máy ảo Kali Linux.

3\. Import và khởi động máy ảo Metasploitable2-Linux.

4\. Tạo máy ảo Windows 11 x64 phục vụ tình huống Before/After.

5\. Cấu hình các máy ảo sử dụng mạng Host-Only để tạo môi trường thực hành cô lập.

6\. Kiểm tra địa chỉ IP của các máy trước khi thực hiện quét.

7\. Từ Kali Linux, sử dụng Nmap để kiểm tra host, cổng và dịch vụ của các máy mục tiêu.

8\. Lưu kết quả quét ra các tệp để làm minh chứng và phục vụ so sánh.



\---



\## 4. Các tình huống đã thực hiện



\### 4.1. Host Discovery



Thực hiện dò tìm các host đang hoạt động trong mạng Host-Only bằng Nmap.



Mục đích:



\- Xác định các máy đang hoạt động.

\- Xác định địa chỉ IP của máy mục tiêu trước khi thực hiện quét chi tiết.



\*\*Kết quả: PASS\*\*



\---



\### 4.2. Quét cổng TCP



Thực hiện quét các cổng TCP trên máy mục tiêu bằng Nmap.



Qua quá trình quét Metasploitable2, Nmap phát hiện nhiều cổng và dịch vụ đang mở, ví dụ:



\- 21/tcp – FTP

\- 22/tcp – SSH

\- 23/tcp – Telnet

\- 25/tcp – SMTP

\- 53/tcp – DNS

\- 80/tcp – HTTP

\- 139/tcp – NetBIOS

\- 445/tcp – SMB

\- 3306/tcp – MySQL

\- 5432/tcp – PostgreSQL

\- 5900/tcp – VNC

\- 8180/tcp – HTTP/Tomcat



\*\*Kết quả: PASS\*\*



\---



\### 4.3. Nhận diện dịch vụ và phiên bản



Sử dụng tùy chọn `-sV` của Nmap để xác định dịch vụ và phiên bản phần mềm đang hoạt động trên các cổng mở.



Kết quả quét cho phép xác định được nhiều dịch vụ của Metasploitable2 như FTP, SSH, Telnet, Apache HTTP, Samba, MySQL, PostgreSQL, VNC và Apache Tomcat.



Kết quả được lưu phục vụ cho việc phân tích và làm minh chứng.



\*\*Kết quả: PASS\*\*



\---



\### 4.4. Kiểm tra dịch vụ SMB



Từ kết quả quét, máy Metasploitable2 có các cổng SMB/NetBIOS liên quan như:



\- 139/tcp

\- 445/tcp



Kết quả liên quan đến SMB được lọc và lưu vào tệp:



`smb.txt`



\*\*Kết quả: PASS\*\*



\---



\### 4.5. Xuất kết quả Nmap



Kết quả quét được lưu dưới nhiều định dạng để thuận tiện cho việc kiểm tra và làm báo cáo.



Các tệp đã tạo gồm:



\- `nmap\_normal.txt`

\- `nmap\_result.xml`

\- `nmap\_result.html`

\- `smb.txt`



Trong quá trình chuyển kết quả XML sang HTML có sử dụng `xsltproc`.



\*\*Kết quả: PASS\*\*



\---



\### 4.6. Tình huống Before/After trên Windows 11



Máy Windows 11 có địa chỉ:



`192.168.23.128`



Thực hiện quét bằng cùng một câu lệnh Nmap trước và sau khi thay đổi trạng thái Windows Defender Firewall.



\#### Before



Khi Firewall chưa chặn kết nối quét, Nmap phát hiện:



\- 135/tcp – open – msrpc

\- 139/tcp – open – netbios-ssn

\- 445/tcp – open – microsoft-ds



Kết quả được lưu tại:



`windows\_before.txt`



\#### After



Sau khi bật Windows Defender Firewall và thực hiện lại cùng phép quét:



\- 1000 TCP ports ở trạng thái filtered/no-response.

\- Không còn hiển thị các cổng 135, 139 và 445 ở trạng thái open như kết quả Before.



Kết quả được lưu tại:



`windows\_after.txt`



Điều này cho thấy thay đổi cấu hình Firewall đã làm thay đổi kết quả quan sát được từ phía máy quét.



\*\*Kết quả: PASS\*\*



\---



\## 5. Kết quả tổng hợp



| Nội dung | Kết quả |

|---|---|

| Dựng môi trường VMware | PASS |

| Kết nối mạng Host-Only | PASS |

| Host Discovery | PASS |

| Quét TCP | PASS |

| Nhận diện service/version | PASS |

| Kiểm tra SMB | PASS |

| Xuất kết quả TXT/XML | PASS |

| Chuyển kết quả sang HTML | PASS |

| Quét Windows Before | PASS |

| Thay đổi Firewall | PASS |

| Quét Windows After | PASS |

| Lưu minh chứng Before/After | PASS |



\*\*Kết quả chung: PASS\*\*



\---



\## 6. Lỗi gặp phải và cách khắc phục



\### Lỗi 1: Không tìm thấy file `nmap\_result.xml`



Khi kiểm tra:



`ls -lh nmap\_result.xml`



hệ thống báo:



`No such file or directory`



\*\*Cách khắc phục:\*\* Thực hiện lại lệnh Nmap và xuất đúng kết quả XML. Sau đó kiểm tra lại file trước khi xử lý tiếp.



\---



\### Lỗi 2: Lỗi khi chuyển XML sang HTML



Trong quá trình sử dụng `xsltproc`, ban đầu xuất hiện lỗi parse HTML/XML.



\*\*Cách khắc phục:\*\* Sử dụng đúng stylesheet của Nmap và đúng thứ tự tham số:



`xsltproc -o nmap\_result.html /usr/share/nmap/nmap.xsl nmap\_result.xml`



Sau đó kiểm tra và xác nhận `nmap\_result.html` đã được tạo.



\---



\### Lỗi 3: Nhập nhầm lệnh `map` thay vì `nmap`



Terminal báo:



`sudo: map: command not found`



\*\*Cách khắc phục:\*\* Kiểm tra lại câu lệnh và thực hiện bằng `nmap`.



\---



\### Lỗi 4: Nhập sai địa chỉ IP Windows



Trong lần quét đầu tiên nhập nhầm địa chỉ IP khiến Nmap báo không tìm thấy mục tiêu.



\*\*Cách khắc phục:\*\* Sử dụng `ipconfig` trên Windows 11 để kiểm tra lại IPv4 Address. Địa chỉ đúng là:



`192.168.23.128`



Sau đó thực hiện lại phép quét.



\---



\### Lỗi 5: Shared Folder không xuất hiện trong Kali



Ban đầu thư mục `/mnt/hgfs` không hiển thị thư mục chia sẻ từ Windows.



\*\*Cách khắc phục:\*\* Mount lại VMware Shared Folder bằng:



`sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow\_other`



Sau khi mount thành công, thư mục chia sẻ `Nmap` xuất hiện tại:



`/mnt/hgfs/Nmap`



Từ đó có thể sao chép các tệp kết quả từ Kali sang máy thật.



\---



\## 7. Các tệp kết quả



Các tệp minh chứng chính của bài LAB gồm:



\- `nmap\_normal.txt`

\- `nmap\_result.xml`

\- `nmap\_result.html`

\- `smb.txt`

\- `windows\_before.txt`

\- `windows\_after.txt`



Các tệp trên được lưu lại để phục vụ kiểm tra kết quả, viết báo cáo và làm minh chứng cho quá trình thực hành.



\---



\## 8. Kết luận



Qua LAB4, em đã thực hành sử dụng Nmap để dò tìm host, quét cổng TCP, nhận diện dịch vụ và phiên bản, kiểm tra một số dịch vụ trên máy mục tiêu và lưu kết quả dưới nhiều định dạng.



Ở tình huống Before/After, kết quả quét cho thấy trước khi áp dụng thay đổi phòng thủ, Nmap phát hiện các cổng 135, 139 và 445 của Windows ở trạng thái open. Sau khi bật Firewall, kết quả quét cho thấy 1000 cổng TCP ở trạng thái filtered/no-response. Qua đó có thể quan sát được ảnh hưởng của cấu hình Firewall đối với kết quả quét từ một máy khác trong mạng Host-Only.

