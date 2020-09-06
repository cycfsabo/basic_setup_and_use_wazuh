# Cài đặt cơ bản wazuh với mô hình 1 server
## Mô hình triển khai
### Không sử dụng Splunk:
![image](https://user-images.githubusercontent.com/41882267/92073650-6e767100-edde-11ea-9e45-cd389af44b5b.png)

### Sử dụng Splunk:
![image](https://user-images.githubusercontent.com/41882267/92312408-70148480-efea-11ea-9fed-1d71a4d88d2b.png)


Để thực hiện như mô hình, chuẩn bị 3 máy ảo trên VMWare:
- Wazuh server:

  OS: Ubuntu server 18.04
  
  CPU: 2 core
  
  Ram: 2.5gb
  
  Hard disk: 40gb
  
  Network interface card: NAT
  
- Monitored endpoint:

  OS: Ubuntu server 18.04
  
  CPU: 2 core
  
  Ram: 2gb
  
  Hard disk: 20gb
  
  Network interface card: NAT

- Splunk Indexer:
  
  OS: Ubuntu server 18.04
  
  CPU: 2 core
  
  Ram: 2.5gb
  
  Hard disk: 20gb
  
  Network interface card: NAT
  
## Cài đặt
### Cài đặt Wazuh server:
#### Cài đặt Wazuh manager
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

#### Cài đặt Wazuh API

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

#### Cài đặt Filebeat
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

### Cài đặt Monitored endpoint:

- Từ menu bên trái http://192.168.182.160:5601, chọn Wazuh, sau đó chon Total agents và nhập như sau:

![image](https://user-images.githubusercontent.com/41882267/92102997-8e6f5a00-ee09-11ea-888a-1666672d1c56.png)
![image](https://user-images.githubusercontent.com/41882267/92103052-a050fd00-ee09-11ea-91b8-e3da9f392295.png)

- Copy command để cài đặt trên máy ảo monitored endpoint:
```
curl -so wazuh-agent.deb https://packages.wazuh.com/3.x/apt/pool/main/w/wazuh-agent/wazuh-agent_3.13.1-1_amd64.deb && sudo WAZUH_MANAGER='192.168.182.160' dpkg -i ./wazuh-agent.deb
```
![image](https://user-images.githubusercontent.com/41882267/92103316-fb82ef80-ee09-11ea-9deb-bfbac3452ff6.png)

- Ngay lập tức Wazuh server đã nhận ra agent:

![image](https://user-images.githubusercontent.com/41882267/92103498-4270e500-ee0a-11ea-99fd-7002e0232b9e.png)
![image](https://user-images.githubusercontent.com/41882267/92103537-4ac92000-ee0a-11ea-978e-9c78a4ddfbb7.png)


### Cài đặt Splunk (Single-node)
#### Cài đặt Splunk Indexer

- Download Splunk v8.0.4 package từ trang chủ.

![image](https://user-images.githubusercontent.com/41882267/92311266-dd6ee800-efdf-11ea-8b1a-b2ed40cfa0fe.png)

- Chuyển qua user root.
- Cài đặt Splunk v8.0.6 package bằng lệnh:
```
dpkg --install splunk-8.0.6-152fb4b2bb96-linux-2.6-amd64.deb 
```
![image](https://user-images.githubusercontent.com/41882267/92311358-7b62b280-efe0-11ea-8489-0be9e9e1f35b.png)

- Tạo file inputs.conf:
```
curl -so /opt/splunk/etc/system/local/indexes.conf https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/splunk/peer-indexes.conf
```
- Tạo file ìnputs.conf:
```
curl -so /opt/splunk/etc/system/local/inputs.conf https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/splunk/peer-inputs.conf
```
- Chắc chắn rằng Splunk v8.0.4 được cài đặt ở /opt/splunk và start the service:
```
/opt/splunk/bin/splunk start
```
![image](https://user-images.githubusercontent.com/41882267/92311413-004dcc00-efe1-11ea-8587-8e9da5a14269.png)

Nhập y
Sau đó nhập admin username: hungcao / password: 12345678

![image](https://user-images.githubusercontent.com/41882267/92311458-64709000-efe1-11ea-9cf8-55f763841a7d.png)

Vào địa chỉ http://192.168.182.162:8000/ với 192.168.182.162 là địa chỉ ip của máy ảo để kiểm tra:
![image](https://user-images.githubusercontent.com/41882267/92311506-c29d7300-efe1-11ea-9b60-98b261fe6e34.png)


- Để bật Splunk khi khởi động, sử dụng lệnh:
```
/opt/splunk/bin/splunk enable boot-start
```
![image](https://user-images.githubusercontent.com/41882267/92311470-794d2380-efe1-11ea-9bda-7254ea1b3c75.png)


#### Cài đặt Wazuh app cho Splunk:

- Tải xuống phiên bản Wazuh app mới nhất cho Splunk:
```
curl -o SplunkAppForWazuh.tar.gz https://packages.wazuh.com/3.x/splunkapp/wazuhapp-splunk-3.13.1_8.0.4.tar.gz
```

- Cài đặt Wazuh app cho Splunk:
```
/opt/splunk/bin/splunk install app SplunkAppForWazuh.tar.gz
```
![image](https://user-images.githubusercontent.com/41882267/92311581-6dae2c80-efe2-11ea-941d-4e0d2096b940.png)

- Restart Splunk Server:
```
/opt/splunk/bin/splunk restart
```
![image](https://user-images.githubusercontent.com/41882267/92311595-96cebd00-efe2-11ea-9b20-6f4c72aca121.png)

- Vào lại Splunk trên trình duyệt để kiểm tra:

![image](https://user-images.githubusercontent.com/41882267/92311614-bbc33000-efe2-11ea-9fb9-9f1db06b8f65.png)


- Kết nối đến Wazuh API như sau:

![image](https://user-images.githubusercontent.com/41882267/92311889-7f450380-efe5-11ea-86b6-9cadd93cf4bd.png)
![image](https://user-images.githubusercontent.com/41882267/92311897-94ba2d80-efe5-11ea-8f11-41c7baf4d704.png)

#### Cài đặt và cấu hình Splunk Forwarder:

- Chuyển qua máy ảo Wazuh Server. 
- Tải về Splunk Forwarder v8.0.5 package từ trang chủ.
- Cài đặt Splunk Forwarder bằng lệnh:
```
dpkg --install splunkforwarder-8.0.5-a1a6394cc5ae-linux-2.6-amd64.deb 
```
![image](https://user-images.githubusercontent.com/41882267/92312022-25453d80-efe7-11ea-81b1-450893a84f13.png)

- Tải xuống và chèn vào the props.conf template:
```
curl -so /opt/splunkforwarder/etc/system/local/props.conf https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/splunk/props.conf
```
- Download và chèn vào inputs.conf template:
```
curl -so /opt/splunkforwarder/etc/system/local/inputs.conf https://raw.githubusercontent.com/wazuh/wazuh/v3.13.1/extensions/splunk/inputs.conf
```
- Set Wazuh manager hostname:
```
sed -i "s:MANAGER_HOSTNAME:$(hostname):g" /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Point Forwarder xuất ra Wazuh’s Splunk Indexer bằng lệnh sau:
```
/opt/splunkforwarder/bin/splunk add forward-server 192.168.182.162:9997
```
Chọn y và nhập vào username: hungcao / password: 12345678

![image](https://user-images.githubusercontent.com/41882267/92312122-10b57500-efe8-11ea-9589-404e96afc977.png)

- Restart Splunk Forwarder service:
```
/opt/splunkforwarder/bin/splunk restart
```
![image](https://user-images.githubusercontent.com/41882267/92312150-5114f300-efe8-11ea-8c57-62a9614fc1e1.png)

- Để bật Splunk Forwarder khi khởi động, sử dụng lệnh:
```
/opt/splunkforwarder/bin/splunk enable boot-start
```
![image](https://user-images.githubusercontent.com/41882267/92312150-5114f300-efe8-11ea-8c57-62a9614fc1e1.png)

- Vào trở lại địa chỉ http://192.168.182.162:8000/, vào Wazuh app > Agents > Security events và xem kết quả:

![image](https://user-images.githubusercontent.com/41882267/92312333-dbaa2200-efe9-11ea-8783-c74ec4e5e9cb.png)

## Use cases:
### Phát hiện SSH brute-force attack:
#### Tấn công
- Thực hiện ssh vào Wazuh agent với 1 user không tồn tại bằng lệnh:
```
ssh abc@192.168.182.161
```
![image](https://user-images.githubusercontent.com/41882267/92318877-8d723e80-f03c-11ea-830b-6099ab13470e.png)

#### Ở Wazuh server
- Wazuh server sẽ lập tức ghi nhận:

![image](https://user-images.githubusercontent.com/41882267/92318899-be527380-f03c-11ea-9a0d-690143f20d58.png)

#### Ở Kibana:
- Vào Kibana > Discover và search "abc" trong wazuh alert cũng sẽ thấy có cảnh báo:

![image](https://user-images.githubusercontent.com/41882267/92320084-f495f000-f048-11ea-8c2b-5336b074c076.png)

{
### Phát hiện các tiến trình ẩn:
#### Tạo tiến trình ẩn
Trong bài thực hành này, tôi sẽ triển khai một cách an toàn rootkit chế độ hạt nhân trên máy lab của mình để phát hiện Wazuh phát hiện.
- Thực hiện các lệnh sau:
```
echo "syscheck.debug=2" > /var/ossec/etc/local_internal_options.conf
echo "agent.debug=2" >> /var/ossec/etc/local_internal_options.conf
echo "rootcheck.sleep=0" >> /var/ossec/etc/local_internal_options.conf
echo "syscheck.sleep=0" >> /var/ossec/etc/local_internal_options.conf
systemctl restart wazuh-agent
```
- Cài đặt gcc, Kernel Headers, libgcc-7-dev:
```
apt install gcc
apt install linux-headers-$(uname -r)
apt-get install -y libgcc-7-dev
```
- Tải về rootkit bằng lệnh:
```
git clone https://github.com/wazuh/Diamorphine.git
```
- Vào folder Diamorphine và compile:
```
cd Diamorphine
make
```
}


### Thay đổi rule:
- Vào Wazuh trên Kibana, vào Management, chọn Rules

![image](https://user-images.githubusercontent.com/41882267/92321440-2b253800-f054-11ea-824d-9c325a7955bc.png)

- Chọn Manage rules file, sau đó search sshd và xem file 0095-sshd_rules.xml:
![image](https://user-images.githubusercontent.com/41882267/92321499-bdc5d700-f054-11ea-834c-a5351a520a67.png)

- Xuống Rule id 5716 và copy từ <rule> đến </rule>

![image](https://user-images.githubusercontent.com/41882267/92321540-109f8e80-f055-11ea-9540-4e306109b6ca.png)

- Chọn Custom rules để hiện ra file local_rules.xml:

![image](https://user-images.githubusercontent.com/41882267/92321772-dc2cd200-f056-11ea-8752-8f9c50c42ddd.png)

- Paste đoạn bên trên vào file local_rules.xml ở trong tag group mới, sau đó thêm overwrite="yes" vào tag rule, sửa rule level và nhấn Save:

![image](https://user-images.githubusercontent.com/41882267/92321833-6117eb80-f057-11ea-9a07-fd82cbe47b9d.png)

- Chọn Restart now sau đó chọn Confirm:

![image](https://user-images.githubusercontent.com/41882267/92321854-899fe580-f057-11ea-8b69-45177bd44600.png)


- Thực hiện SSH với username đúng nhưng password sai vào Agent và kiểm tra log ở Kibana sẽ thấy log gần nhất có rule level là 7:
![image](https://user-images.githubusercontent.com/41882267/92323205-02576f80-f061-11ea-8362-1f5396a24ebb.png)




