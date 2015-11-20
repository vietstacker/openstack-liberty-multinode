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
- Cài đặt git
```sh
apt-get instal -y git
```

##### Tải script và thực thi script
- Tải script
```sh
git clone https://github.com/vietstacker/openstack-liberty-multinode.git

mv /root/openstack-liberty-multinode/LIBERTY-U14.04-AIO
rm -rf openstack-liberty-multinode

cd LIBERTY-U14.04-AIO 
chmod +x *.sh
```


