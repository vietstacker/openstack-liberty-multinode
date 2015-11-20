# Cài đặt & HDSD OpenStack LIBERTY AIO

### Giới thiệu
- Script cài đặt OpenStack Liberty trên một máy chủ
- Các thành phần cài đặt bao gồm
  - MariaDB, NTP
  - Keystone Version 3
  - Glance
  - Neutron (ML2, OpenvSwitch)
  
### Môi trường cài đặt
- LAB trên Vmware Workstation hoặc máy vật lý, đáp ứng yêu cầu tối thiểu sau:
```sh
 - RAM: 4GB
 - HDD
  - HDD1: 60GB (cài OS và các thành phần của OpenStack)
  - HDD2: 40GB (sử dụng để cài CINDER - cung cấp VOLUME cho OpenStack) - CHÚ Ý: NẾU KHÔNG CÀI CINDER THÌ KHÔNG CẦN Ổ NÀY
 - 02 NIC với thứ tự sau
  - NIC 1:  - eth0 - Management Network
  - NIC 2: - eth1 - External Network
 - CPU hỗ trợ ảo hóa
```

### Các bước thực hiện
#### Chuẩn bị môi trường trên VMware


#### Thực hiện cài đặt script
#### Tải GIT và cấu hình ip động cho các card mạng.
- Cấu hình network bằng đoạn lệnh sau để đảm bảo máy chủ có 02 NIC
```sh

cat << EOF > /etc/network/interfaces
auto lo
iface lo inet loopback

# NIC MGNT
auto eth0
iface eth0 inet dhcp

# NIC EXT
auto eth1
iface eth1 inet dhcp
EOF

```

- Khởi động lại network
```sh
ifdown -a && ifup -a
```

- Kiểm tra lại địa chỉ IP của máy cài OpenStack, đảm bảo có đủ 02 NIC bằng lệnh `landscape-sysinfo`

```sh
root@controller:~# landscape-sysinfo

  System load:  0.93              Users logged in:       1
  Usage of /:   4.0% of 94.11GB   IP address for eth0:   10.10.10.159
  Memory usage: 53%               IP address for eth0  172.16.69.228
  Swap usage:   0%                
```

- Kiểm tra kết nối internet bằng lệnh `ping google.com`
```sh
root@controller:~# ping google.com

PING google.com (203.162.236.211) 56(84) bytes of data.
64 bytes from 203.162.236.211: icmp_seq=1 ttl=57 time=0.877 ms
64 bytes from 203.162.236.211: icmp_seq=2 ttl=57 time=0.786 ms
64 bytes from 203.162.236.211: icmp_seq=3 ttl=57 time=0.781 ms

```
- Cài đặt git với quền root
```sh
su -
apt-get update
apt-get -y install git
```

#### Tải script và thực thi script
- Tải script
- Sử dụng quyền root để đăng nhập, với Ubuntu 14.04 cần đăng nhập bằng user thường trước, sau đó chuyển qua root bằng lệnh su -

```sh
su -

git clone https://github.com/vietstacker/openstack-liberty-multinode.git

mv /root/openstack-liberty-multinode/LIBERTY-U14.04-AIO
rm -rf openstack-liberty-multinode

cd LIBERTY-U14.04-AIO 
chmod +x *.sh
```

##### Thực thi script đặt IP cho các card mạng
- Script sẽ thực hiện tự động việc đặt IP tĩnh cho các card mạng
```sh
bash 0-liberty-aio-ipadd.sh
```

##### Cài đặt các gói NTP, MARIADB, RABBITMQ
- Đăng nhập lại máy chủ với quyền root và thực thi script
```sh
bash 1-liberty-aio-prepare.sh
```

##### Cài đặt Keystone
- Thực thi script dưới để cài đặt Keystone
```sh
bash 2-liberty-aio-keystone.sh
```

- Thực thi lệnh dưới để khai báo biến môi trường cho OpenStack
```sh
source admin-openrc.sh
```

##### Cài đặt GLANCE
```sh
bash 3-liberty-aio-glance.sh
```

##### Cài đặt NOVA
```
bash 4-liberty-aio-nova.sh
```

##### Cài đặt NEUTRON
- Cài đặt OpenvSwitch và cấu hình lại NIC
```sh
bash 5-liberty-aio-config-ip-neutron.sh
```
- Sau khi thực thi xong script trên, máy chủ sẽ khởi động lại. Đăng nhập với quyền root và tiếp tục thực hiện lệnh dưới để cài NEUTRON

```sh
bash 6-liberty-aio-install-neutron.sh
```

##### Cài đặt Horizon
```
bash 7-liberty-aio-install-horizon.sh
```

#### Hướng dẫn tạo network, vm trên dashboard
```sh
đang update
```