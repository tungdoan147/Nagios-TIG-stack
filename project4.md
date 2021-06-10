# Projiect 4: Dựng mô hình Graph monitor (Grafana + InfluxDB + Telegraf + Kapacitor)


## I. Mục tiêu
+ Dùng Telegraf để đẩy metric CPU, RAM, DISK từ 1 server cần monitor vào Server database chạy InfluxDB, sau đó hiển thị lên Grafana
+ Setup giao diện web InfluxDB và query metric CPU trên đó
+ Cảnh báo dùng Kapacitor ( trong bài viết sẽ sử dụng luôn Grafana )

## II. Các bước
1. Cài server chạy TIG
2. Cài server cần monitor và đấy các tham số lên,  query metric CPU
3. Dùng Grafana cảnh báo qua mail, telegram

## III. Thực hiện

### 1. Cài đặt server chạy TIG stack 

#### + Cài đặt InfluxDB

+ Tạo repo cho InfluxDB

`# vi /etc/yum.repos.d/influxdb.repo`

Thêm vào nội dung sau:

>
		[influxdb]
		name = InfluxDB Repository - RHEL $releasever
		baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
		enabled = 1
		gpgcheck = 1
		gpgkey = https://repos.influxdata.com/influxdb.key

Cập nhật repo: `yum repolist`

+ Cài đặt InfluxDB 

`# yum install influxdb -y`

+ Khởi động dịch vụ và cấu hình khởi động cùng hệ thống và cho phép các port 8086 8088 đi qua firewalld

`# systemctl start influxdb`

`# systemctl enable influxdb`

`# firewall-cmd --permanent --add-port=8086/tcp`

`# firewall-cmd --permanent --add-port=8088/tcp`

`# firewall-cmd --reload`

+ Test trạng thái `systemctl status infuxdb`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/vhbbk2r6vh_image.png)

+ Set up databases và user trên InfluxDB

`# influx`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/nft35rn1c0_image.png)

Tiếp đó ta sẽ tạo database

`> create database telegraf`

`> create user telegraf with password 'P@ssw0rd'`

Show database và user đã tạo bằng lệnh 

`> show databases`

`> show users`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/kohax5j69p_image.png)

#### + Cài Telegraf Agent

+ Thiết lập cài đặt 

`# yum install -y telegraf`

`# systemctl start telegraf`

`# systemctl enable telegraf`

+ Tạo backup file cấu hình 	

`# cp telegraf.conf telegraf.conf.bak`

+ Chỉnh file cấu hình telegraf.conf `# vi telegraf.conf`

> 
		...
		hostname = "tig_server"           (dòng 94)
		...
		[[outputs.influxdb]]
		...
		urls = ["http://127.0.0.1:8086"]    (dòng 112)
		...
		database = "telegraf"               (dòng 116)
		...
		username = "telegraf"               (dòng 149)
		password = "P@ssw0rd"               (dòng 150)
		...

*Chú ý: Nếu muốn xem số dòng trong VI gõ `:se nu`

+ Khởi động lại dịch vụ `# systemctl restart telegraf`

#### + Cài đặt Grafana

+ Tạo repo Grafana

`# vi /etc/yum.repos.d/grafana.repo`

Thêm vào nội dung sau:

>
	[grafana]
	name=grafana
	baseurl=https://packages.grafana.com/oss/rpm
	repo_gpgcheck=1
	enabled=1
	gpgcheck=1
	gpgkey=https://packages.grafana.com/gpg.key
	sslverify=1
	sslcacert=/etc/pki/tls/certs/ca-bundle.crt

+ Cập nhật repo `# yum repolist`

+ Cài đặt Grafana + cho phép các port 3000 qua firewalld

`# yum install -y grafana`

`# systemctl start grafana-server`

`# systemctl enable grafana-server`

`# firewall-cmd --zone=public --add-port=3000/tcp --permanent`

`# firewall-cmd --reload`

#### + Set up Grafana

+ Truy nhập link `IP server:3000` 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/f724ejllu8_image.png)

*Vì máy đã được thiết lập rồi. Nếu lần đầu sẽ Grafana sẽ yêu cầu user và password

+ Trong tab `configuration` chọn `Datasource`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/qbxed06q25_image.png)

+ Chọn `Add data source`và chọn InfluxDB để liên kết với InfluxDB đã cài ở trên 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/lrmv22ta98_image.png)

+ Điền các thông tin cần giám sát vào Telegraf sau đó chọn Save & test 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/duj6xbdktb_image.png)

+ Liên kết database thành công sẽ có kết quả sau:

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/yeo7sp5b0c_image.png)

+ Tại tab Create, chọn Import để thêm template dashboard đã có sẵn (được public) hoặc có thể tự vẽ dashboard 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/jbxa5x0rg7_image.png)

+ Chọn data source chon Import. Sau dó sẽ có Dashboard Grafana nếu cài thành công

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/krqku0wsx4_image.png)

### 2. Cài server cần monitor

+ Ở bên server này chỉ cần cài InfluxDB với telegraf tương tự như trên và không cần cài Grafana

+ Truy nhập vào Grafana của server chủ tạo Data Source tương tụ ở trên với thông tin của server cần monitor

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/daa80nzf3g_image.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/33ucs1f15b_image.png)

+ Chọn `Create` và `Add an empty panel`

+ Chọn các tham số mong muốn theo dõi và thông tin sẽ được hiển thị trên đồ thị

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/qwcey0qmvc_image.png)

*Trên hình là query CPU. Các tham số khác thực hiện tương tự 

### 3. Cấu hình cảnh báo qua Grafana

#### 1. Cấu hình gửi cảnh báo qua mail

+ Truy nhập vào server TIG sửa file `/etc/grafana/grafana.ini`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/fz13xkmzmp_image.png)

+ Bật quyền truy cập của ứng dụng kém an toàn trên gmail

+ Vào Alerting + Notification channel + New channel. Điền thông tin về mail gửi

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/tm03urwby7_image.png)

+ Cấu hình cảnh báo Ram vượt quá 16% sẽ cảnh báo ( giả sử)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ureaatk6za_image.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/s9hpbslmye_image.png)

+ Sau đó sẽ có mail gửi về (đợi tầm 3 4 phút )

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/3gguq5kcmj_image.png)

#### 2. Cấu hình gửi cảnh báo qua telegram

+ Tạo bot và lấy chat ID của bot trên telegram
+ Tạo New channel tương tự Gmail

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/2mxaweorc7_image.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/2ojtyid84g_image.png)
