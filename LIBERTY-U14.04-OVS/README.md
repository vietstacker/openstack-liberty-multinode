# Các bước thực hiện

### Chuẩn bị môi trường LAB

### Thực thi script
- Cài đặt gói git và tải script 
```sh
su -
apt-get update
apt-get -y install git 

git clone https://github.com/vietstacker/openstack-liberty-multinode.git
mv /root/openstack-liberty-multinode/LIBERTY-U14.04-OVS/ /root/
rm -rf openstack-liberty-multinode/
cd LIBERTY-U14.04-OVS/
chmod +x *.sh

```

## Cài đặt trên CONTROLLER NODE
### Cài đặt script thiết lập IP và repos cho Liberty
```sh
bash ctl-1-ipadd.sh
```

### Cài đặt gói NTP, MariaDB
```sh
bash ctl-2-prepare.sh
```

### Cài đặt KEYSTONE
- Cài đặt Keystone
```sh
bash ctl-3.keystone.sh
```

- Khai báo biến môi trường
```sh
source admin-openrc.sh
```

### Cài đặt GLANCE
```sh
bash ctl-4-glance.sh
```

### Cài đặt NOVA
```sh
bash ctl-5-nova.sh
```

### Cài đặt NEUTRON
```sh
bash ctl-6-neutron.sh
```
- Sau khi cài đặt NEUTRON xong, node controller sẽ khởi động lại.
- Đăng nhập bằng `root` và thực thi script cài đặt Horizon

### Cài đặt HORIZON
- Đăng nhập với quyền `root` và thực hiện script sau
```sh
bash ctl-horizon.sh
```

## Cài đặt trên COMPUTE NODE
### Tải GIT và script
- Cài đặt gói git và tải script 
```sh
su -
apt-get update
apt-get -y install git 

git clone https://github.com/vietstacker/openstack-liberty-multinode.git
mv /root/openstack-liberty-multinode/LIBERTY-U14.04-OVS/ /root/
rm -rf openstack-liberty-multinode/
cd LIBERTY-U14.04-OVS/
chmod +x *.sh

### Thiết lập IP và hostname
- Sửa file config để có IP phù hợp với máy hiện tại.
- Thực thi script để thiết lập IP, hostname
```sh
bash com1-ipdd.sh
```
- Máy chủ sẽ khởi động lại sau khi thực thi script `com1-ipdd.sh`
- Đăng nhập vào máy chủ với quyền root và thực thi script cài đặt các thành phần lên nova

```sh
su -
cd LIBERTY-U14.04-OVS/
bash com1-prepare.sh
```

Sau khi cài đặt xong COMPUTE NODE, chuyển qua bước sử dụng dashboad


## Hướng dẫn sử dụng dashboard để tạo network, VM, tạo các rule.
### Tạo rule cho project admin
- Đăng nhập vào dasboard
![liberty-horizon1.png](/images/liberty-horizon1.png)

- Chọn tab `admin => Access & Security => Manage Rules`
![liberty-horizon2.png](/images/liberty-horizon2.png)

- Chọn tab `Add Rule`
![liberty-horizon3.png](/images/liberty-horizon3.png)

- Mở rule cho phép từ bên ngoài SSH đến máy ảo
![liberty-horizon4.png](/images/liberty-horizon4.png)
- Làm tương tự với rule ICMP để cho phép ping tới máy ảo và các rule còn lại.

### Tạo network
#### Tạo dải external network
- Chọn tab `Admin => Networks => Create Network`
![liberty-net-ext1.png](/images/liberty-net-ext1.png)

- Nhập và chọn các tab như hình dưới.
![liberty-net-ext2.png](/images/liberty-net-ext2.png)

- Click vào mục `ext-net` vừa tạo để khai báo subnet cho dải external.
![liberty-net-ext3.png](/images/liberty-net-ext3.png)

- Chọn tab `Creat Subnet`
![liberty-net-ext4.png](/images/liberty-net-ext4.png)

- Khai báo dải IP của subnet cho dải external 
![liberty-net-ext5.png](/images/liberty-net-ext5.png)

- Khai báo pools và DNS
![liberty-net-ext6.png](/images/liberty-net-ext6.png)

#### Tạo dải internal network
- Lựa chọn các tab lần lượt theo thứ tự `Project admin => Network => Networks => Create Network"
![liberty-net-int1.png](/images/liberty-net-int1.png)

- Khai báo tên cho internal network
![liberty-net-int2.png](/images/liberty-net-int2.png)

- Khai báo subnet cho internal network
![liberty-net-int3.png](/images/liberty-net-int3.png)

- Khai báo dải IP cho Internal network
![liberty-net-int4.png](/images/liberty-net-int4.png)

#### Tạo Router cho project admin
- Lựa chọn theo các tab "Project admin => Routers => Create Router
![liberty-r1.png](/images/liberty-r1.png)

- Tạo tên router và lựa chọn như hình
![liberty-r2.png](/images/liberty-r2.png)

- Gán interface cho router
![liberty-r3.png](/images/liberty-r3.png)

![liberty-r4.png](/images/liberty-r4.png)

![liberty-r5.png](/images/liberty-r5.png)
- Kết thúc các bước tạo exteral network, internal network, router


## Tạo máy ảo (Instance)
- Lựa chọn các tab dưới `Project admin => Instances => Launch Instance`
![liberty-instance1.png](/images/liberty-instance1.png)

![liberty-instance2.png](/images/liberty-instance2.png)

![liberty-instance3.png](/images/liberty-instance3.png)













