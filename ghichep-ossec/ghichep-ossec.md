# Ghi chép về OSSEC

# Mô hình standalone
## Môi trường thực hiện

- Ubuntu 14.04 64 bit

##  Update OS  và cài đặt gói bổ trợ

- Update OS 

    ```sh
    apt-get update -y 
    ```

- Cài đặt gói bổ trợ

    ```sh
    apt-get -y install build-essential make libssl-dev git unzip
    
    apt-get -y install mysql-server libmysqlclient-dev mysql-client \
        apache2 php5 libapache2-mod-php5 php5-mysql php5-curl php5-gd \
        php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache \
        php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl
    ```

## Cài đăt OSSEC

### Cài từ repos

    ```sh
    apt-key adv --fetch-keys http://ossec.wazuh.com/repos/apt/conf/ossec-key.gpg.key
    echo "deb http://ossec.wazuh.com/repos/apt/ubuntu trusty main" >> /etc/apt/sources.list
    apt-get update

    apt-get install ossec-hids
    apt-get install ossec-hids-agent
    ```

### Cài từ gói

- Tải gói cài đặt

    ```sh
    wget https://bintray.com/artifact/download/ossec/ossec-hids/ossec-hids-2.8.3.tar.gz
    ```

- Giải nén

    ```sh
    tar -zxf ossec-hids-2.8.3.tar.gz
    ```
- Thực hiện cài đặt

    ```sh
    cd ossec-hids-2.8.1
    ```

### Draf

- Cai postfix-as-a-send-only-smtp-server-on-ubuntu-14-04

 - https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-14-04
 
- Lệnh test ssh

```sh
hydra -t 128 -l user_name -V -x '4:4:aA1"@#$!()=`~?><;:%^&*_-+/,.\ ' 172.16.69.230 ssh
``` 

# Các bước cài với mô hình Client Server
## Trên Server
### Cài `OSSEC`

- Update OS 

    ```sh
    apt-get update -y 
    ```
    
- Tải các gói bổ trợ

    ```sh
    apt-get -y install build-essential make libssl-dev git unzip
    ```

    ```sh
    apt-get -y install mysql-server libmysqlclient-dev mysql-client \
        apache2 php5 libapache2-mod-php5 php5-mysql php5-curl php5-gd \
        php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache \
        php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl
    ```

- Tải và giải nén `OSSEC`

    ```sh    
    wget https://bintray.com/artifact/download/ossec/ossec-hids/ossec-hids-2.8.3.tar.gz
    tar -xzf ossec-hids-2.8.3.tar.gz
    cd ossec-hids-2.8.3
    ```
- Kích hoạt để `OSSEC` sử dụng MySQL

    ```sh
    cd src
    make setdb
    ```

- Cài đặt

    ```sh
    cd  ../
    ./install.sh
    ```
    
- Lựa chọn các tùy chọn sau

    ```sh
    /var/ossec/bin/ossec-control start
    ```

- Tạo DB cho `OSSEC`

    ```sh
    mysql -u root -p
    create database ossec;
    grant all privileges on ossec.* to ossecuser@localhost identified by 'PTCC@!2o015';
    flush privileges;
    exit
    ```
    
- Tạo schema cho ossec
    
    ```sh
    mysql -u root -p ossec < src/os_dbd/mysql.schema
    ```

- Khai bao trong /var/ossec/etc/ossec.conf

    ```sh
    <ossec_config>
        <database_output>
            <hostname>127.0.0.1</hostname>
            <username>ossecuser</username>
            <password>PTCC@!2o015</password>
            <database>ossec</database>
            <type>mysql</type>
        </database_output>
    </ossec_config>
    ```

- Kích hoạt DB cho WEB 

    ```sh
    /var/ossec/bin/ossec-control enable database
    /var/ossec/bin/ossec-control restart 
    ```
    
### Cài đặt OSSEC WEB UI

    ```sh
    cd /var/www/html/
    wget https://github.com/ossec/ossec-wui/archive/master.zip
    unzip master.zip
    mv ossec-wui-master/ ossec/
    mkdir ossec/tmp/
    chown www-data: -R ossec/
    chmod 666 /var/www/html/ossec/tmp/
    usermod -a -G ossec www-data
    ```