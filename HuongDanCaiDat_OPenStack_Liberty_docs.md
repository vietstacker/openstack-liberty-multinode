## Hướng dẫn cài đặt OpenStack từ docs.

### Mô hình cài đặt


### Các thiết lập cơ bản

- Các thao tác được thực hiện với tài khoản root.
- Chạy các lệnh dưới ngay sau khi cài đặt xong máy ảo.
- Mô hình phải đảm bảo cấu hình đúng dải IP ở trên.

`
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
`

#### Node Controller
##### Thiết lập IP, hostname


- Thiết lập hostname với tên là `controller` 

```sh 
echo "controller" > /etc/hostname
hostname -F /etc/hostname
```

Thiết lập địa chỉ IP

- Sao lưu file cấu hình của card mạng

`
cp /etc/network/interfaces /etc/network/interfaces.bak
`

- Sử dụng script dưới để cấu hình IP tĩnh cho card mạng.

```sh
cat << EOF > /etc/network/interfaces

# NIC loopback
auto lo
iface lo inet loopback

# NIC MGNG
auto eth0
iface eth0 inet static
address 10.10.10.164
netmask 255.255.255.0

# NIC EXTERNAL
auto eth1
iface eth1 inet static
address 172.16.69.164
netmask 255.255.255.0
gateway 172.16.69.1
dns-nameservers 8.8.8.8


EOF

```

Cấu hình file /etc/hosts để phân giản IP cho các node

```sh
cat << EOF > /etc/hosts 
127.0.0.1   controller
10.10.10.164    controller
10.10.10.165    compute1
10.10.10.166    compute2
10.10.10.167    cinder
10.10.10.169    swift1
10.10.10.170    swift2
EOF

```
Khởi động lại node Controller

`
int 6
`

### Cài đăt các gói phần mềm
#### Cài đặt gói NTP

Cài đăt gói dưới

`
apt-get -y install chrony
`

Sao lưu file /etc/chrony/chrony.conf 

`
cp /etc/chrony/chrony.conf  /etc/chrony/chrony.conf.ba
`

Sửa file ` /etc/chrony/chrony.conf` với lệnh sed

```sh
sed -i 's/server 0.debian.pool.ntp.org offline minpoll 8/#server 3.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf 
sed -i 's/server 1.debian.pool.ntp.org offline minpoll 8/#server 1.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf 
sed -i 's/server 2.debian.pool.ntp.org offline minpoll 8/#server 2.debian.pool.ntp.org offline minpoll 8/g' /etc/chrony/chrony.conf 
sed -i 's/server 3.debian.pool.ntp.org offline minpoll 8/server controller iburst/g' /etc/chrony/chrony.conf 
```

Khởi động lại NTP
```sh
service chrony restart
```

Khai báo các gói để cài đặt OpenStack Liberty

```sh
apt-get -y install software-properties-common
add-apt-repository -y cloud-archive:liberty 
```

Update và khởi động lại sau khi khai báo repos cho OpenStack Liberty

```sh
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
```

Cài đặt gói OpenStack Client

```sh
apt-get -y install python-openstackclient
```

#### Cài đặt My SQL

Trong quá trình cài đặt yêu cầu nhập mật khẩu My SQL, sử dụng mật khẩu Welcome123 để thống nhất.

```sh
apt-get -y install mariadb-server python-pymysql
```


Tạo file với nội dung sau

```sh
cat << EOF  /etc/mysql/conf.d/mysqld_openstack.cnf

[mysqld]
bind-address = 10.10.10.164

[mysqld]
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
EOF

``` 

#### Cài đặt Message queue

Cài đặt gói rabbitmq

```sh
apt-get -y install rabbitmq-server
```

Tạo tài khoản `openstack` cho rabbitmq

```sh 
rabbitmqctl add_user openstack Welecome123
```

Cấp quyền cho tài khoản openstack 

```sh
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Cài đặt dịch vụ Keystone

##### Tạo database cho keystone

Đăng nhập vào MariaDB

```sh
mysql -u root -pWelcome123
```

Tạo DB tên là keystone và gán quyền

```sh
CREATE DATABASE keystone;

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'Welcome123';

GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'Welcome123';

```

##### Cài đặt Keystone

Cấu hình không cho Keystone tự động khởi động.

```sh
echo "manual" > /etc/init/keystone.override
```

Cài đặt các gói dành cho Keystone

```sh
apt-get -y install keystone apache2 libapache2-mod-wsgi memcached python-memcache
```

Chỉnh sửa file cấu hình của keystone như dưới

```sh
sed -i 's/#admin_token = ADMIN/admin_token = Welcome123/g' /etc/keystone/keystone.conf 

sed -i '/connection = .*/{s|sqlite:///.*|mysql+pymysql://'"keystone"':'Welcome123'@'10.10.10.164'/keystone|g}' /etc/keystone/keystone.conf 

sed -i 's/#servers = localhost:11211/servers = localhost:11211/g' /etc/keystone/keystone.conf 

sed -i 's/#provider = uuid/provider = uuid/g' /etc/keystone/keystone.conf 


sed -i 's/\[token\]/ \
\[token\] \
driver = memcache/g' /etc/keystone/keystone.conf 

sed -i 's/\[revoke\]/ \
\[revoke\] \
driver = sql/g' /etc/keystone/keystone.conf 


sed -i 's/#verbose = true/verbose = true/g' /etc/keystone/keystone.conf 


```

Đồng bộ database cho keystone

```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

Cấu hình apache cho Keystone

```sh
echo "ServerName 10.10.10.164" > /etc/apache2/conf-available/servername.conf
sudo a2enconf servername
```

Tạo file `/etc/apache2/sites-available/wsgi-keystone.conf` với nội dung sau


```sh
cat << EOF > /etc/apache2/sites-available/wsgi-keystone.conf
Listen 5000
Listen 35357
 
<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /var/www/cgi-bin/keystone/main
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>
 
<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /var/www/cgi-bin/keystone/admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    LogLevel info
    ErrorLog /var/log/apache2/keystone-error.log
    CustomLog /var/log/apache2/keystone-access.log combined
</VirtualHost>
 
EOF
```