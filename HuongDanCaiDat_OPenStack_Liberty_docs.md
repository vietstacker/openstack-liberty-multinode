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
netmask 255.255.255.255
gateway 172.16.69.1
dns-nameservers 8.8.8.8

EOF
```

Cấu hình file /etc/hosts để phân giản IP cho các node

```sh
cat << EOF > /etc/hosts 

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

