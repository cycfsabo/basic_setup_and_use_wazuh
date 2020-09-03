# Cài đặt cơ bản wazuh với mô hình 1 server
## Mô hình triển khai
![image](https://user-images.githubusercontent.com/41882267/92073650-6e767100-edde-11ea-9e45-cd389af44b5b.png)

Để thực hiện như mô hình, chuẩn bị 2 máy ảo trên VMWare:
- Wazuh server:
  OS: Ubuntu server 18.04
  CPU: 2 core
  Ram: 4gb
  Hard disk: 40gb
  Network interface card: NAT
- Monitored endpoint:
  OS: Ubuntu server 18.04
  CPU: 2 core
  Ram: 2gb
  Hard disk: 20gb
  Network interface card: NAT
  
## Cài đặt
### Cài đặt Wazuh server:
- Cài đặt các công cụ phát triển và trình biên dịch bằng lệnh:
```
apt-get install python gcc make libc6-dev curl policycoreutils automake autoconf libtool lsb-release
```
![image](https://user-images.githubusercontent.com/41882267/92076658-260e8180-ede5-11ea-91c2-2618c6ed77ee.png)

- Tải về và giải nén phiên bản mới nhất của wazuh bằng lệnh:
```
curl -Ls https://github.com/wazuh/wazuh/archive/v3.13.1.tar.gz | tar zx
```
- Vào thư mục wazuh-3.13.1, chạy file install.sh để cài đặt:
```
cd wazuh-*
./install.sh
```

- Màn hình cài đặt hiện ra, nhấn Enter để mặc định sử dụng tiếng Anh:

![image](https://user-images.githubusercontent.com/41882267/92077132-00ce4300-ede6-11ea-9a38-933b9b2f9d43.png)

- Nhấn Enter để tiếp tục:

![image](https://user-images.githubusercontent.com/41882267/92077208-24918900-ede6-11ea-862e-0ae098defcda.png)

- Nhập vào manager để chọn cài đặt Wazuh server, sau đó nhấn Enter:

![image](https://user-images.githubusercontent.com/41882267/92077514-c87b3480-ede6-11ea-8f2a-364ff6098df0.png)›

- Nhấn enter để cài đặt môi trường mặc định:

![image](https://user-images.githubusercontent.com/41882267/92077550-db8e0480-ede6-11ea-9af8-a6468ce97d9d.png)


- Nhấn enter ở các bước sau để mặc định: không nhận email thông báo, chạy kiểm tra tính toàn vẹn daemon, chạy rootkit, chạy kiểm tra giám sát policy (OpenSCAP), không thêm IP vào whitelist, kích hoạt nhật ký hệ thống từ xa (cổng 514 udp), chạy daemon Auth, khởi động Wazuh sau khi cài đặt

![image](https://user-images.githubusercontent.com/41882267/92078159-02006f80-ede8-11ea-89b4-bd8491568549.png)

- Enter để tiến hành cài đặt:

![image](https://user-images.githubusercontent.com/41882267/92078265-2bb99680-ede8-11ea-875e-f38304d14362.png)



