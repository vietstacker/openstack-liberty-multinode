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






