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
- Mở file `/etc/network/interfaces` sửa với nội dung dưới.

`
vi /etc/network/interfaces
`

- Nội dung như sau

```sh
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# MGNG
auto eth0
iface eth0 inet static
address 10.10.10.164
netmask 255.255.255.0

#  EXT
auto eth1
iface eth1 inet static
address 172.16.69.164
netmask 255.255.255.255
gateway 172.16.69.1
dns-nameservers 8.8.8.8
```


