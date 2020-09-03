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
- Chuyển vào chế độ user root.
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

- Enter để hoàn tất cài đặt:

![image](https://user-images.githubusercontent.com/41882267/92078551-ada9bf80-ede8-11ea-9d20-a9dc75500e1c.png)

- Để kiểm tra trạng thái wazuh-manager, sử dụng lệnh:
```
systemctl status wazuh-manager
```
Wazuh-manager đã hoạt động:

![image](https://user-images.githubusercontent.com/41882267/92078814-1ee97280-ede9-11ea-9ebc-beea1bd2e7b6.png)

### Cài đặt Wazuh API

- Cài đặt NodeJS bằng lệnh:
```
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
apt-get install -y nodejs
npm config set user 0
```
- Tải về và thực thi script cài đặt:
```
curl -s -o install_api.sh https://raw.githubusercontent.com/wazuh/wazuh-api/v3.13.1/install_api.sh && bash ./install_api.sh download
```
![image](https://user-images.githubusercontent.com/41882267/92090141-45170e80-edf9-11ea-82dc-ec42e6f53be3.png)

- Kiểm tra trạng thái của Wazuh API bằng lệnh:
```
systemctl status wazuh-api
```
![image](https://user-images.githubusercontent.com/41882267/92090251-67a92780-edf9-11ea-99a3-9bf65293152d.png)

### Cài đặt Filebeat
- Thêm vào Elastic repository và GPG key của nó bằng các lệnh:
```
apt-get install curl apt-transport-https
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
```
- Cài đặt Filebeat bằng lệnh:
```
apt-get install filebeat=7.9.0
```
![image](https://user-images.githubusercontent.com/41882267/92093074-1733c900-edfd-11ea-8f18-782538c76006.png)

- Tải xuống tệp cấu hình Filebeat từ kho lưu trữ Wazuh. Điều này được định cấu hình trước để chuyển tiếp các cảnh báo Wazuh tới Elasticsearch:
```
curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/filebeat/7.x/filebeat.yml
```
- Tải xuống alerts template cho Elasticsearch:
```
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/elasticsearch/7.x/wazuh-template.json
```
- Tải xuống module Wazuh cho Filebeat:
```
curl -s https://packages.wazuh.com/3.x/filebeat/wazuh-filebeat-0.1.tar.gz | sudo tar -xvz -C /usr/share/filebeat/module
```
- Sửa file /etc/filebeat/filebeat.yml bằng lệnh:
```
nano -c /etc/filebeat/filebeat.yml
```
Thay thế YOUR_ELASTIC_SERVER_IP bằng 0.0.0.0

![image](https://user-images.githubusercontent.com/41882267/92093458-93c6a780-edfd-11ea-9fb5-d11cf7d0541d.png)

- Enable và start the Filebeat service:
```
systemctl daemon-reload
systemctl enable filebeat.service
systemctl start filebeat.service
```
- Kiểm tra trạng thái của Filebeat bằng lệnh:
```
systemctl status filebeat.service
```
![image](https://user-images.githubusercontent.com/41882267/92093757-e86a2280-edfd-11ea-86cd-86825795b327.png)

### Cài đặt Elastic Stack
#### Chuẩn bị
- Thêm Elastic repository và GPG key của nó bằng các lệnh:
```
apt-get install curl apt-transport-https
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
apt-get update
```
![image](https://user-images.githubusercontent.com/41882267/92090948-4432ac80-edfa-11ea-81fe-fcb6fea04e2e.png)

#### Cài đặt Elasticsearch
- Cài đặt Elasticsearch package bằng lệnh:
```
apt-get install elasticsearch=7.9.0
```
![image](https://user-images.githubusercontent.com/41882267/92091184-94aa0a00-edfa-11ea-8d6f-69d835e9abc8.png)

- Sửa file /etc/elasticsearch/elasticsearch.yml bằng lệnh:
```
nano -c /etc/elasticsearch/elasticsearch.yml
```
Bỏ comment dòng 23, 55, 59, 72. 
Sửa dòng 55 thành: "network.host: 0.0.0.0"
Sửa dòng 72 thành: "cluster.initial_master_nodes: ["node-1"]"
Sau đó lưu lại.

![image](https://user-images.githubusercontent.com/41882267/92091665-421d1d80-edfb-11ea-8442-a6205e001084.png)
![image](https://user-images.githubusercontent.com/41882267/92091696-4ba68580-edfb-11ea-8102-26032d69b48a.png)

- Enable và start Elasticsearch service bằng các lệnh:
```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```
- Để kiểm tra, từ trình duyệt của laptop, truy cập vào địa chỉ http://192.168.182.160:9200/ với 192.168.182.160 là ip của máy ảo Wazuh server:

![image](https://user-images.githubusercontent.com/41882267/92092215-00d93d80-edfc-11ea-84b6-14a46ab009e1.png)

- Sau khi Elasticsearch được thiết lập và chạy, nên tải về Filebeat template. Chạy lệnh ở nơi Filebeat đã được cài đặt:
```
filebeat setup --index-management -E setup.template.json.enabled=false
```
![image](https://user-images.githubusercontent.com/41882267/92093882-09327800-edfe-11ea-9f70-99202270ebb9.png)

#### Cài đặt Kibana:
- Cài đặt Kibana package bằng lệnh:
```
apt-get install kibana=7.9.0
```
![image](https://user-images.githubusercontent.com/41882267/92095036-5b27cd80-edff-11ea-8b12-0f2a9d3ceb5f.png)

- Update permission thư mục optimize và plugins bằng các lệnh:
```
chown -R kibana:kibana /usr/share/kibana/optimize
chown -R kibana:kibana /usr/share/kibana/plugins
```
- Cài đặt Wazuh app plugin cho Kibana bằng các lệnh:
```
cd /usr/share/kibana/
sudo -u kibana bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-3.13.1_7.9.0.zip
```
![image](https://user-images.githubusercontent.com/41882267/92095338-b954b080-edff-11ea-9844-97a339de246a.png)

- Sửa file /etc/kibana/kibana.yml bằng lệnh:
```
nano -c /etc/kibana/kibana.yml
```
Bỏ comment các dòng 2, 7, 28. 
Sửa dòng 7 thành: "server.host: "0.0.0.0" "
Sửa dòng 28 thành: "elasticsearch.hosts: ["http://localhost:9200"]"
Sau đó lưu lại

![image](https://user-images.githubusercontent.com/41882267/92095677-28320980-ee00-11ea-9e20-e3cfcaac4a96.png)

- Enable và start the Kibana service bằng các lệnh:
```
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
```

- Chờ 5 đến 10 phút để Kibana hoàn toàn được bật lên. Truy cập vào địa chỉ http://192.168.182.160:5601 với 192.168.182.160 là địa chỉ ip của Wazuh-server để kiểm tra.

![image](https://user-images.githubusercontent.com/41882267/92096342-f79e9f80-ee00-11ea-80b2-cb126ddbabe2.png)






