## Hướng dẫn cài đặt OpenStack từ docs.

### Mô hình cài đặt
- Lưu ý:
 - Sử dụng mô hình cài đặt thu gọn gồm 1 node Controller và nhiều node Compute

### Các thiết lập cơ bản

- Các thao tác được thực hiện với tài khoản root.
- Chạy các lệnh dưới ngay sau khi cài đặt xong máy ảo.
- Mô hình phải đảm bảo cấu hình đúng dải IP ở trên.

Khai báo các gói để cài đặt OpenStack Liberty

```sh
apt-get -y install software-properties-common
add-apt-repository -y cloud-archive:liberty 
```

#### Node Controller
##### Thiết lập IP, hostname

- Bạn có thể sửa IP phù hợp với máy bạn, tốt nhất là sử dụng theo IP mà chúng tôi hướng dẫn.
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
address 10.10.10.120
netmask 255.255.255.0

# NIC EXTERNAL
auto eth1
iface eth1 inet static
address 172.16.69.120
netmask 255.255.255.0
gateway 172.16.69.1
dns-nameservers 8.8.8.8


EOF

```

Cấu hình file /etc/hosts để phân giản IP cho các node

```sh
cat << EOF > /etc/hosts 
127.0.0.1   controller localhost
10.10.10.120    controller
10.10.10.121    compute1

EOF
```

Update và khởi động lại node `controller`

```sh
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
```

Đăng nhập với IP mới của node controller

### Cài đăt các gói phần mềm

#### Cài đặt gói OpenStack Client

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
cat << EOF  > /etc/mysql/conf.d/mysqld_openstack.cnf

[mysqld]
bind-address = 10.10.10.120

[mysqld]
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
EOF

``` 

Khởi động lại MYSQL

```sh
service mysql restart
```

#### Cài đặt Message queue

Cài đặt gói rabbitmq

```sh
apt-get -y install rabbitmq-server
```

Tạo tài khoản `openstack` cho rabbitmq

```sh 
rabbitmqctl add_user openstack Welcome123
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
flush privileges;

quit

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

- Sao lưu file cấu hình của keystone.
```sh
cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
```

- Xóa file keystone gốc 
```sh
rm /etc/keystone/keystone.conf
```


- Tạo file keystone mới bằng lệnh `vi /etc/keystone/keystone.conf` chứa nội dung dưới.

```sh 

[DEFAULT]
log_dir = /var/log/keystone

admin_token = Welcome123
public_bind_host = 10.10.10.120
admin_bind_host = 10.10.10.120

[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
connection = mysql+pymysql://keystone:Welcome123@10.10.10.120/keystone


[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[eventlet_server_ssl]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[matchmaker_ring]
[memcache]
servers = localhost:11211


[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_qpid]
[oslo_messaging_rabbit]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[resource]
[revoke]
driver = sql

[role]
[saml]
[signing]
[ssl]
[token]
provider = uuid
driver = memcache

[tokenless_auth]
[trust]
[extra_headers]
Distribution = Ubuntu

```



Đồng bộ database cho keystone

```sh
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

- Cấu hình apache cho Keystone

```sh
echo "ServerName 10.10.10.164" > /etc/apache2/conf-available/servername.conf
```

Tạo file `/etc/apache2/sites-available/wsgi-keystone.conf` với nội dung sau


```sh
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

```

Cấu hình virtual host cho keystone

```sh 
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```

Khởi động lại apache

```sh
service apache2 restart
```

Xóa SQLite mặc định của keystone

```sh
rm -f /var/lib/keystone/keystone.db
```

Tạo endpoint 

```sh
export OS_TOKEN=Welcome123
export OS_URL=http://10.10.10.164:35357/v3
export OS_IDENTITY_API_VERSION=3
```

Tạo service cho Keystone

```sh
openstack service create --name keystone --description "OpenStack Identity" identity
openstack endpoint create --region RegionOne identity public http://10.10.10.120:5000/v2.0
openstack endpoint create --region RegionOne identity internal http://10.10.10.120:5000/v2.0
openstack endpoint create --region RegionOne identity admin http://10.10.10.120:35357/v2.0

openstack project create --domain default --description "Admin Project" admin
openstack user create  --domain default --password Welcome123 admin
openstack role create admin
openstack role add --project admin --user admin admin
openstack project create --domain default --description "Service Project" service
openstack project create --domain default --description "Demo Project" demo
openstack user create --domain default --password Welcome123 demo
openstack role create user
openstack role add --project demo --user demo user
```

- Hủy 02 biến môi trường đã khai báo trước đó
```sh
unset OS_TOKEN OS_URL
```

- Tạo file admin.sh với nội dung dưới bằng lệnh `vi admin.sh`

```sh
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=Welcome123
export OS_AUTH_URL=http://10.10.10.120:35357/v3
export OS_IDENTITY_API_VERSION=3
```

- Phân quyền cho file `admin.sh`
```sh
chmod +x admin.sh
```

- Chạy lênh dưới để khai báo biến môi trường
```sh
source admin.sh
```

- Kiểm tra xem keystone hoạt động tốt hay chưa bằng lệnh
```sh
openstack token issue
```

- Kết quả tương tự như dưới
```sh
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2015-11-17T09:53:40.242778Z      |
| id         | de796ac24b2545efb99487d9ff4e981a |
| project_id | c685a5fa3e474261b678aeb59332ce0d |
| user_id    | 818e335d15484101b6a2a69e5f9d4f61 |
+------------+----------------------------------+

```
