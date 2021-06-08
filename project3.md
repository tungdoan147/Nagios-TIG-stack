# Project 3: Setup Nagios và viết Bash Script phục vụ cảnh báo NRPE cho Nagios

## Lý thuyết 



## Mục lục

- Dựng server Nagios
- Thiết lập NRPE trên client và server
- Viết Bash Script cảnh báo qua mail

## Thực hiện

### 1. Dựng server Nagios

#### - Cài đặt một số gói cần thiết

`yum install httpd php php-cli gcc glibc glibc-common gd gd-devel net-snmp openssl-devel wget unzip -y`

#### - Tắt SE Linux và firewalld

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux`

`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

`systemctl disable firewalld`

`systemctl stop firewalld`

- Kiểm tra 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/d6saejt5hz_image.png)

#### - Tạo user

> 
useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios 
usermod -a -G nagcmd apache

`htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin`

( Tạo tài khoản để đăng nhập vào web Nagios )


#### - Cài đặt Nagios

> 
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz
tar zxf nagios-4.1.1.tar.gz
cd nagios-4.1.1
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode
make install-webconf

- Download cài đặt Nagios plugins

>
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
tar zxf nagios-plugins-2.1.1.tar.gz
cd /tmp/nagios-plugins-2.1.1
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make all
make install

`systemctl start httpd` ( khởi động dịch vụ web)

`systemctl start nagios` (khởi động dịch vụ Nagios )

#### - Kết quả 
- Truy nhập vào trang web với đường dẫn `Địa-chỉ-IP-server/nagios`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/thtfwjc3z3_image.png)

Như vậy đã thiết lập xong server Nagios

### 2. Thiết lập NRPE trên client và server

#### - Mô hình 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/im7a8rtgum_image.png)

#### - Các thiết lập trên client

- Cài đặt gói
 
`yum install -y gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel`

- Tạo NRPE user

`useradd nagios`
`passwd nagios`

- Download , cài đặt file plugins 

`wget https://www.nagios-plugins.org/download/nagios-plugins-2.1.2.tar.gz
`

>
 tar -xvf nagios-plugins-2.1.2.tar.gz
cd nagios-plugins-2.1.2
./configure
 make
 make install
 
 - Thêm user vào group, cấp quyền sử dụng tệp lưu NRPE cho user và group nagios

>
usermod -a -G nagios nagios
chown nagios.nagios /usr/local/nagios
chown -R nagios.nagios /usr/local/nagios/libexec

- Cài đặt `xinetd` `yum install xinetd -y`
- Download cài đặt NRPE

>
wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz
tar xzf nrpe-3.2.1.tar.gz
cd nrpe-3.2.1
./configure
make all 
make install
make install-plugin
make install-config 
make install-init
make install-inetd

- Truy nhập file  `/usr/local/nagios/etc/nrpe.cfg` 

`allowed_hosts=127.0.0.1,(nagios server IP )`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/uhplwh3zt8_image.png)

- Sửa file /etc/services sử dụng port 5666 cho NRPE. Thêm dòng 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/7gjnfuvq07_image.png)

- Khởi động dịch vụ 

>
service xinetd restart
systemctl start nrpe 
systemctl enable nrpe 

- Kiểm tra dịch vụ 

- ![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/o70sx7qmtt_image.png)

Như vậy là hoàn thiện các thao tác trên client

#### - Thiết lập trên Nagios server vừa tạo 

- Download, cài đặt NRPE

`wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-3.2.1/nrpe-3.2.1.tar.gz`

`tar xzf nrpe-3.2.1.tar.gz`

`cd nrpe-3.2.1`

`./configure`

`make check_nrpe`

`make install-plugin`

- Kiểm tra xem đã cài đặt thành công chưa 

`/usr/local/nagios/libexec/check_nrpe -H <remote_linux_ip_address>`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/dcg506cste_image.png)

- Thêm 1 host để Nagios server giám sát 

Thêm vào file `/usr/local/nagios/etc/nagios.cfg` Khai báo thông tin các fiile chứa thông tin host

>
cfg_file=/usr/local/nagios/etc/hosts.cfg
cfg_file=/usr/local/nagios/etc/services.cfg

- Khai báo lệnh NRPE ở file `/usr/local/nagios/etc/objects/commands.cfg`

>
`###############################################################################`
`# NRPE CHECK COMMAND`
`#`
`# Command to use NRPE to check remote host systems`
`###############################################################################`
`define command{`
       `command_name check_nrpe`
        `command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$`
        `}`

- Thêm thông tin host vào file `/usr/local/nagios/etc/hosts.cfg`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/q9726mxja1_image.png)

- Thêm thông tin services vào file `/usr/local/nagios/etc/services.cfg`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/7rfmsfzcxs_image.png)

- Khởi động lại dịch vụ và kiểm tra kết quả 

`systemctl restart nagios`

Lệnh kiểm tra 

`/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/eavh30aq5y_image.png)

Như thế này là thiết lập hoàn thành. Vào Server Nagios sẽ thấy giám sát được các thông số của client

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/dznc82tlhn_image.png)

### 3. Viết Bash Script cảnh báo qua mail( thực hiện công việc trên Nagios server)

- Cài đặt gói mail postfix

`yum -y install postfix cyrus-sasl-plain mailx`

- Tạo file lưu trữ email và password trong file:  `/etc/postfix/sasl_passwd`
  + Cấu trúc 
  
  [smtp.gmail.com]:587 <tên gmail>:mật khẩu

- Khai báo file chứa địa chỉ mail tại file cấu hình chính của dịch vụ mail

`vi /etc/postfix/main.cf`

>
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt

- Thiết lập chế độ less secure của Gmail
- Cấp quyền sử dụng vào đọc file chứa user passwd của mail mình sử dụng để gửi đến các mail cần nhận cảnh báo

>
chown root:postfix /etc/postfix/sasl_passwd*
chmod 600 /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd

- Khởi động dịch vụ postfix

`systemctl start postfix`
`systemctl enable postfix`

- Kiểm tra dịch vụ đã chạy chưa

` echo "This is a test." | mail -s "test message" nguyendoantung69@gmail.com`

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/q2wufnqo2s_image.png)

Như thế này là dịch vụ đã thành công

- Khai báo contact nhận cảnh báo mail

Truy nhập vào file `/usr/local/nagios/etc/objects/contacts.cfg`. Sửa thông tin ở phần `define contact ` như sau 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/drbei163k0_image.png)

- Khởi động lại dịch vụ 

`systemctl restart nagios`

- Kiểm tra dịch vụ. Thử tắt client và kiểm tra xem mail có thông báo chưa

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/qclhwx5att_image.png)

Đã có thông báo về gmail về việc client down. Như vậy việc setup hoàn tất.

Ngoài ra chúng ta có thể được báo cáo về tình trạng CPU 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/tiysly0nq3_image.png)

Và ngoài những dịch vụ trên chúng ta có thể cấu hình giám sát nhiều dịch vụ khác như httpd, ssh.....
