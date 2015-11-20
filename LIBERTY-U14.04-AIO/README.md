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
  - HDD2: 40GB (sử dụng để cài CINDER - cung cấp VOLUME cho OpenStack)
 - 02 NIC với thứ tự sau
  - NIC 1:  - eth0 - Management Network
  - NIC 2: - eth1 - External Network
 - CPU hỗ trợ ảo hóa
```

### Các bước thực hiện
#### Chuẩn bị môi trường trên VMware


#### Thực hiện cài đặt script
#### Tải GIT và kiểm tra các NIC
- Cài đặt git
```sh
apt-get instal -y git
```

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



