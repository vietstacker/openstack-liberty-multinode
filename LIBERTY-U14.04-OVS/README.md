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

### Cài đặt HORIZON
- Đăng nhập với quyền `root` và thực hiện script sau
```sh
bash ctl-horizon.sh
```

