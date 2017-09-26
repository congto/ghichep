## ghi chep - pacemaker, corosync + DRBD

### Yêu cầu 
- 02 máy chủ CentOS 7. Node1 tên là `nsdb1`, Node2 tên là `nsdb2`
- mỗi máy gồm
  - 02 NICs: NIC1 dùng để ra internet và quản trị (`192.168.20.57, 192.168.20.58`), NIC2 dùng để replicate (`172.16.20.57, 172.16.20.58`) 
  - 02 ổ cứng: HDD1 dùng để cài OS,  HDD2 dùng để cấu hình DRBD
  
### Các bước cấu hình chuẩn bị
- Cấu hình trên cả 02 node 
- Đặt hostname, IP, cấu hình các service cơ bản.

  ```sh
  yum install -y epel-release 
  yum update -y
  yum install -y byobu wget 
  ```

### Khai báo hostname 
- Khai báo hostname để các máy có thể liên lạc với nhau thông qua tên hostname
- Thực hiện các lệnh sau trên cả 02 node 

  ```sh
  echo "192.168.20.57 nsdb1" >> /etc/hosts
  echo "192.168.20.58 nsdb2" >> /etc/hosts
  ```

### Cài đặt apache 

- Cài đặt httpd trên cả 02 node 

  ```sh
  yum install -y httpd*
  ```

### Cai dat pacemaker - corosync

- Thực hiện trên cả 02 node 
- Thực hiện cài đặt pacemaker - corosync trên cả 02 node 

```sh
yum install  -y  pacemaker pcs
```

- Khởi động pacemaker
  ```sh
  systemctl start pcsd 
  systemctl enable pcsd
  ```

- 


- Đặt mật khẩu cho user `hacluster` (lưu ý: vẫn thực hiện bước này trên cả 02 node)
  ```sh
  passwd hacluster
  ```
  - Nhập mật khẩu giống nhau trên 02 node và ghi nhớ để điền cho bước dưới.
  
- Đứng trên một trong 02 node và thực hiện lệnh dưới để xác thực các node sẽ tham gia vào cluster (trong hướng dẫn này đứng trên node1)

  ```sh
  pcs cluster auth nsdb1 nsdb2
  ```
  - Kết quả như dưới là ok, nếu chưa được thì kiểm tra lại các mục đặt hostname và đặt mật khẩu 
    ```sh
    [root@nsdb1 ~]# pcs cluster auth nsdb1 nsdb2
    Username: hacluster
    Password:
    nsdb1: Authorized
    nsdb2: Authorized
    [root@nsdb1 ~]#
    ```

-  Tạo cluster (vẫn đứng trên node1 để thực hiện)
  ```sh
  pcs cluster setup --name ha_cluster nsdb1 nsdb2
  ```
  - Kết quả như dưới là ok
    ```sh
    [root@nsdb1 ~]# pcs cluster setup --name ha_cluster nsdb1 nsdb2
    Destroying cluster on nodes: nsdb1, nsdb2...
    nsdb1: Stopping Cluster (pacemaker)...
    nsdb2: Stopping Cluster (pacemaker)...
    nsdb1: Successfully destroyed cluster
    nsdb2: Successfully destroyed cluster

    Sending 'pacemaker_remote authkey' to 'nsdb1', 'nsdb2'
    nsdb1: successful distribution of the file 'pacemaker_remote authkey'
    nsdb2: successful distribution of the file 'pacemaker_remote authkey'
    Sending cluster config files to the nodes...
    nsdb1: Succeeded
    nsdb2: Succeeded

    Synchronizing pcsd certificates on nodes nsdb1, nsdb2...
    nsdb1: Success
    nsdb2: Success
    Restarting pcsd on the nodes in order to reload the certificates...
    nsdb1: Success
    nsdb2: Success
    [root@nsdb1 ~]#
    ```

- Sau bước trên thì cả 02 node sẽ có nội dung file `/etc/corosync/corosync.conf` giống nhau. Có thể dùng lệnh `cat /etc/corosync/corosync.conf` để xem lại.

- Khởi động cluster (vẫn đứng trên node1 để thực hiện nhé)

  ```sh
  pcs cluster start --all 
  ```
  
- Kích hoạt cho cluster được khởi động cùng hệ điều hành.
  ```sh
  pcs cluster enable --all 
  ```

- Kiểm tra lại trạng thái của cluster sau khi thiết lập (đứng trên bất kỳ node nào để thực hiện các lệnh này)
  ```sh
  pcs status cluster 
  
  pcs status corosync
  ```
  
- Thực hiện các lệnh đưới đối với mô hình có 02 node (3 node thì không cần vì đây là cơ chế quorum và stonith - google để đọc thêm)
  ```sh
  
  pcs property set stonith-enabled=false
  
  pcs property set no-quorum-policy=ignore
  pcs property set default-resource-stickiness="INFINITY"
  ```
  
### Tham khảo:
- http://www.learnitguide.net/2016/07/integrate-drbd-with-pacemaker-clusters.html

