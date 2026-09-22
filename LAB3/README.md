\# LAB 3 – NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ĐẾN AN TOÀN THÔNG TIN

## 1. Thông tin sinh viên

- Họ và tên: Nguyễn Quang Minh
- Mã số sinh viên: 1150080105
- Môn học: Thực hành An toàn Hệ thống Thông tin
- Bài thực hành: Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

---

## 2. Môi trường thực hành

- Máy chủ: Windows 11
- Phần mềm ảo hóa: VMware Workstation Pro 26H1 (26.0.1)
- Máy ảo thực hành: Windows 11 x64
- PowerShell: Windows PowerShell 5.1
- Python: Python 3.14.7
- Wireshark/TShark: Wireshark 4.6.8
- Công cụ Sysinternals:
  - Sysmon
  - Autoruns/Autorunsc
  - Process Explorer
- Microsoft Defender Antivirus: Bật trong quá trình thực hành
- Windows Defender Firewall: Bật
- Thư mục thực hành: `C:\LAB3`
- Thư mục lưu bằng chứng: `C:\LAB3\Evidence`

---

## 3. Nội dung đã thực hiện

Trong Lab 3, em đã thực hiện các nội dung chính sau:

### TH1 – Nhận diện các nguồn đe dọa

Phân tích và phân loại các tình huống đe dọa đối với hệ thống thông tin, bao gồm hành động vô ý, hành động cố ý, thảm họa tự nhiên, lỗi kỹ thuật và lỗi quản lý.

### TH2 – Malware và Microsoft Defender

Sử dụng chuỗi kiểm thử EICAR để kiểm tra khả năng phát hiện của Microsoft Defender.

Theo dõi kết quả bằng Windows Security và PowerShell, đồng thời giữ Microsoft Defender hoạt động trong suốt quá trình thực hành, không tạo exclusion và không tắt cơ chế bảo vệ.

### TH3 – Tấn công mật khẩu và xác thực

Tạo tài khoản cục bộ `lab3user` dành riêng cho bài thực hành.

Thực hiện các lần xác thực thành công và không thành công để quan sát dấu vết trong Windows Security Event Log, tập trung vào các Event ID:

- 4624 – đăng nhập thành công.
- 4625 – đăng nhập thất bại.
- 4648 – sử dụng thông tin xác thực rõ ràng.

Sau đó thực hiện thay đổi mật khẩu của tài khoản thử nghiệm và kiểm tra lại trạng thái xác thực.

### TH4 – Backdoor và Persistence

Cài đặt và sử dụng Sysmon, Autoruns và Process Explorer để quan sát hoạt động của hệ thống.

Tạo persistence lành tính phục vụ bài thực hành bằng:

- Registry Run Key: `LAB3_Run_Demo`
- Scheduled Task: `LAB3_Persistence_Demo`

Sử dụng Sysmon và Autoruns để phát hiện các thay đổi trên hệ thống.

Khởi tạo HTTP server cục bộ trên:

`127.0.0.1:8080`

Sau đó sử dụng Process Explorer và các công cụ của Windows để xác định process đang lắng nghe trên cổng 8080.

### TH5 – Sniffing, HTTPS và TLS

Sử dụng Wireshark để bắt và phân tích lưu lượng HTTP cục bộ.

Quan sát HTTP request và thông tin được truyền dưới dạng có thể đọc được.

Tiếp tục quan sát lưu lượng HTTPS/TLS để so sánh với HTTP. Qua đó nhận thấy nội dung ứng dụng của HTTPS được mã hóa, trong khi vẫn có thể quan sát một số metadata như địa chỉ IP, cổng 443 và thông tin TLS handshake.

Không thực hiện MITM hoặc spoofing chủ động.

### TH6 – DoS, DDoS và Mail Bombing

Chạy script `local_load_test.py` do bài Lab cung cấp để tạo tải giới hạn trên HTTP server cục bộ `127.0.0.1:8080`.

Phân tích `ddos_sample.csv` để quan sát sự khác biệt giữa nguồn lưu lượng trong tình huống DDoS.

Phân tích `mailbomb_sample.csv` để thống kê số lượng email theo Sender và tổng dung lượng email nhằm nhận diện dấu hiệu Mail Bombing.

Không thực hiện DoS/DDoS hoặc gửi email hàng loạt tới hệ thống bên ngoài.

### TH7 – Social Engineering và Phishing

Phân tích mẫu email trong `phishing_email.txt` và xác định các dấu hiệu đáng ngờ như:

- Tạo cảm giác khẩn cấp.
- Uy tín giả.
- Domain bất thường.
- From và Reply-To không phù hợp.
- Yêu cầu truy cập liên kết hoặc cung cấp thông tin xác thực.

Phân tích `social_engineering_cases.csv` và phân loại các tình huống Social Engineering gồm Phishing, Spear Phishing, Pretexting, Baiting, Quid Pro Quo và Watering Hole.

### TH8 – Cleanup và Recovery

Sau khi hoàn thành các tình huống thử nghiệm, tiến hành cleanup môi trường:

- Xóa Registry Run Key `LAB3_Run_Demo`.
- Xóa Scheduled Task `LAB3_Persistence_Demo`.
- Dừng HTTP listener trên cổng 8080.
- Xóa tài khoản thử nghiệm `lab3user`.
- Kiểm tra lại Microsoft Defender.
- Thu thập Autoruns sau cleanup và so sánh với baseline.
- Tính SHA-256 cho các file bằng chứng trong thư mục Evidence.

---

## 4. Kết quả thực hiện

Hoàn thành các nội dung thực hành về nhận diện và ứng phó với những nhóm mối đe dọa chính gồm Malware, Password Attack, Keylogging, Backdoor/Persistence, Sniffing, MITM/Spoofing, DoS/DDoS, Mail Bombing, Social Engineering và Phishing.

Microsoft Defender phát hiện mẫu kiểm thử EICAR trong khi cơ chế bảo vệ thời gian thực vẫn được duy trì.

Windows Security Event Log được sử dụng để theo dõi các sự kiện xác thực của tài khoản thử nghiệm.

Sysmon, Autoruns và Process Explorer được sử dụng để xác định các dấu vết liên quan đến process, persistence và network listener.

Wireshark được sử dụng để quan sát sự khác biệt giữa HTTP và HTTPS/TLS.

Các dataset và script offline của bài Lab được sử dụng để phân tích DoS/DDoS, Mail Bombing, Phishing và Social Engineering mà không tác động đến hệ thống bên ngoài.

Sau khi kết thúc bài thực hành, các artefact thử nghiệm được cleanup và các file Evidence được kiểm tra tính toàn vẹn bằng SHA-256.

---

## 5. Bằng chứng thực hành

Các file kết quả được lưu tại:

`C:\LAB3\Evidence`

Báo cáo Word chứa các ảnh chụp trực tiếp từ máy ảo trong quá trình thực hành.

Các hình H1–H11 được sử dụng để minh chứng cho từng nội dung tương ứng của bài Lab.

---

## 6. Lưu ý khi kiểm tra

- Các thao tác được thực hiện trong máy ảo Windows 11 dành cho LAB3.
- Không sử dụng tài khoản, mật khẩu hoặc dữ liệu cá nhân thật trong các tình huống thử nghiệm.
- Microsoft Defender không bị tắt và không tạo exclusion để vượt qua cơ chế bảo vệ.
- HTTP server và bài kiểm tra tải chỉ sử dụng địa chỉ loopback `127.0.0.1`.
- Dataset DDoS và Mail Bombing được phân tích offline.
- Không thực hiện MITM, spoofing, DoS/DDoS hoặc phishing chủ động đối với hệ thống bên ngoài.
- Các artefact được tạo trong quá trình thực hành được cleanup sau khi hoàn thành.
- File Evidence được tính SHA-256 để phục vụ kiểm tra tính toàn vẹn.

---

## 7. Video thực hành

Link video: [DÁN LINK YOUTUBE TẠI ĐÂY]

---

## 8. Cấu trúc thư mục

LAB3/
├── README.md
├── [Báo cáo Lab 3]
├── Evidence/
│   └── Các file bằng chứng của bài thực hành
└── [Các file liên quan khác]
