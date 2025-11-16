# php-app

Deploy php app on 3 servers behine apache loadbalancer and  connect to DB server plus sharing NFS storage on NFS Server


**********----------------------------***********************Notes******************

172.31.0.0/16 ----------> VPC Cidr

172.31.57.255 ----------> webserver-3-private-ip

172.31.10.135 -----------> webserver-2-private-ip

172.31.24.115 ------------> webserver-1-private-ip

172.31.64.169 ------------> DB-server-private-ip

172.31.25.33 -------------> Private-ip-of-nfs-server


----------------------------------******************---------------------------------------
Final Result
![Alt text](./lb-php-app-final-result.png)


**************************Prepare NFS server ********************************************
1) Create NFS server and create 3 volumes of 8GIB and attach volumes to them

![Alt text](./NFS-serverr/NFS-server-with-3-volumes.png)

2) connect to NFS Server


3) Partition disks

![Alt text](./NFS-serverr/partition-disks.png)

5) After Partitioning

![Alt text](./NFS-serverr/afterpartitioning.png)

4) Create physical volume

$ sudo pvcreate /dev/nvme2n1p1

5) Create volume group

![Alt text](./NFS-serverr/Create-volumegroup.png)

6) Create logical volume

![Alt text](./NFS-serverr/Create-lvm.png)

7) add file system to lvms

![Alt text](./NFS-serverr/add-fs.png)


8) Create mountp points

![Alt text](./NFS-serverr/Create-mount-points.png)


9) Install NFS-Server

$ sudo apt install nfs-kernel-server -y

![Alt text](./NFS-serverr/install-nfs.png)

10) Check nfs-server.service status

$ sudo systemctl status nfs-server.service

![Alt text](./NFS-serverr/check-nfsserver-status.png)

11) change ownership of mount points

$ sudo chmod -R  777 /mnt/apps

![Alt text](./NFS-serverr/change-ownership-mount-points.png)

12) add mount points on exports file

![Alt text](./NFS-Server-side-Configuration.png)

13) export mount point

$ exportfs -arv

![Alt text](./NFS-serverr/cc.png)


14)  security group
![Alt text](./NFS-SG.png)

**************************Prepare DB server ***************************************************

a) Connect to db server

b) install mysql-server service

$ sudo apt install mysql-server -y

Start Service
$ sudo systemctl enable --now mysql-server.service


c) Connect to mysql and create db tooling and user webaccess
![Alt text](./DB-Server/connectto-db.png)

![Alt text](./DB-Server/Createdb.png)


************************ Prepare Web-server ******************************************************

1) Connect to Server

2) update server

$ sudo apt update -y

3) Install nfs-client 

$ sudo apt install nfs-kernel-server -y

![Alt text](./Web-server/install-nfs-client.png)

5) Create mount point 
$ sudo mkdir /var/www

![Alt text](./Web-server/create-mountpoint.png)

6) mount /var/www and target's nfs server
$ sudo mount -t nfs -o rw,nosuid /private_ip_of_NFS_Server:/mnt/apps /var/www

![Alt text](./Web-server/mount-step.png)

7) persistent mount

edit this file on /etc/fstab
172.31.25.33:/mnt/apps /var/www nfs defaults 0 0

![Alt text](./Web-server/persistent-mount.png)


9) install nginx
   $  sudo apt install nginx -y

![Alt text](./Web-server/install-nginx.png)
10) enable service
   $ sudo systemctl enable -- now nginx


i) Install php-fpm and php-mysql
    $  sudo apt install php-fpm php-mysql -y
   ![Alt text](./Web-server/install-phpfpm&&php-mysql.png)
    
j) check php version
   $ php -v 
   ![Alt text](./Web-server/get-version-of-php.png)
   $ sudo systemctl status php8.3-fpm
   ![Alt text](./Web-server/php-fpm-status.png)


k) mount logs of nginx on NFS server on /mnt/logs
 
 ![Alt text](./Web-server/mount-logs-of-nginx-to-nfs-server)


Clone repo
$ git clone https://github.com/darey-io/tooling.git

Copy content of tooling/html/* to /var/www/html

sudo cp -R html/* /var/www/html

update configuration to connect to db

on /var/www/html/functions.php

$db = mysqli_connect('172.31.64.169', 'webaccess', 'password123$$$', 'tooling');


********** GO to DB Server************************

Ensure that mysql binded address to all ip's

![Alt text](./Web-server/bind-address.png)

Switch to tooling db crerate users table
 CREATE TABLE users (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     username VARCHAR(50) NOT NULL UNIQUE,
    ->     password VARCHAR(255) NOT NULL,
    ->     email VARCHAR(100) NOT NULL UNIQUE,
    ->     user_type ENUM('admin', 'user', 'guest') DEFAULT 'user',
    ->     status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );

INSERT INTO users (`username`, `password`, `email`, `user_type`, `status`)
VALUES ('esraas', 'WelcomeAgain123$$$', 'esraas@gmail.com', 'admin', 'active');



Now finally we have 3 webservers and have nfs-client installed on them so shared some directories on nfs server



*****************Prepare LoadBalancer **************
a) Create Loadbalancer server


b) Connect to LB

c) Update server and intstall apache and some modules needed
![Alt text](./Load-Balancer/install-apache.png)


d) Enable required modules on apache

![Alt text](./Load-Balancer/modules-for-apache.png)

modules for apache lives on /etc/httpd/conf.modules.d/

e) Create configuration file for loadbalancer

/etc/httpd/conf.d/lb.conf

![Alt text](./Load-Balancer/lb-configuration.png)

f) Restart httpd

g) Access lb public ip address

![Alt text](./Load-Balancer/lb-access.png)

H) configure local DNS name resolution

edit on lb server hosts file && replace ips with names on configuration of lb

![Alt text](./Load-Balancer/hostsfile.png)

![Alt text](./Load-Balancer/lb-config-dns.png)


i) Restart apache


curl http://web1:80

![Alt text](./Load-Balancer/web1.png)

curl http://web2:80

![Alt text](./Load-Balancer/web2.png)

curl http://web3:80

![Alt text](./Load-Balancer/web3.png)








