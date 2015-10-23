## Hướng dẫn cài đặt OpenStack từ docs.

### Mô hình cài đặt


### Các thiết lập cơ bản
- Các thao tác được thực hiện với tài khoản root.
- Chạy các lệnh dưới ngay sau khi cài đặt xong máy ảo.
`
apt-get update -y && apt-get upgrade -y && apt-get dist-upgrade -y && init 6
`
#### Thiết lập IP, hostname
##### Node Controller

- Thiết lập hostname với tên là `controller` 
`echo "controller" > /etc/hostname
hostname -F /etc/hostname
`

