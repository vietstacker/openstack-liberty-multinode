# Cài đặt & HDSD OpenStack LIBERTY AIO

#### Giới thiệu
- Script cài đặt OpenStack Liberty trên một máy chủ
- Các thành phần cài đặt bao gồm
  - MariaDB, NTP
  - Keystone Version 3
  - Glance
  - Neutron (ML2, OpenvSwitch)
  
##### Môi trường cài đặt
- LAB trên Vmware Workstation hoặc máy vật lý, đáp ứng yêu cầu tối thiểu sau:
 - RAM: 4GB
 - HDD
  - HDD1: 60GB (cài OS và các thành phần của OpenStack)
  - HDD2: 40GB (sử dụng để cài CINDER - cung cấp VOLUME cho OpenStack)
 - 02 NIC với thứ tự sau
  - NIC 1:  - eth0 - Management Network
  - NIC 2: - eth1 - External Network
 - CPU hỗ trợ ảo hóa


