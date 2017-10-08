## Ghi chép check_mk

## Cài đặt

### Môi trường
- CentOS 7.x
- Trên trang chủ http://mathias-kettner.com/check_mk.html có 02 phiên bản song song của check_mk Raw Edition: `1.2.8-x` và `1.4.0-x`. Cần chú ý khi cài đặt

### Các bước cài đặt các gói chuẩn bị

- Cài gói bổ trợ
```sh
yum install -y epel-release 
yum update -y
yum install -y wget byobu
```


- Tải check_mk phiên bản `check-mk-raw-1.4.0p13-el7-60.x86_64.rpm`

```sh
wget https://mathias-kettner.de/support/1.4.0p13/check-mk-raw-1.4.0p13-el7-60.x86_64.rpm
```

- Cài đặt check_mk (cài bằng cách này thì ko lo các gói phụ thuộc

```sh
yum install -y check-mk-raw*
```

