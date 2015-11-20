# Các bước thực hiện

### Chuẩn bị môi trường LAB

### Thực thi script
- Cài đặt gói git
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