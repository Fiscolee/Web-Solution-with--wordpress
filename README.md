# Web-Solution-with--wordpress
WEB-SOLUTION-WITH-WORDPRESS
PROJECT 6 - WEB SOLUTION WITH WORDPRESS
WEB SOLUTION WITH WORDPRESS
In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This Project consists of two parts:

Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.

Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

image

Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server In this project, you will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.
Your 3-Tier Setup
A Laptop or PC to serve as a client
An EC2 Linux Server as a web server (This is where you will install WordPress)
An EC2 Linux server as a database (DB) server Use RedHat OS for this project and set the security group to all traffic (not recommend for advanced or real projects)
image

Step 1 — Prepare a Web Server
Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
image

Attach all three volumes one by one to your Web Server EC2 instance. select each volume, go to action and select attach volume, select the webserver instance and save
Open up the Linux terminal to begin configuration

Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
lsblk 
image

Use (df -h) command to see all mounts and free space on your server

Use gdisk utility to create a single partition on each of the 3 disks

sudo gdisk /dev/xvdf
you will see this:

image

image

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): B505F4F7-5D34-4BE7-A6c7-DFCA67AA55D2
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): yes
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
Now,  your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.

repeat same steps for all disks

sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
Use lsblk utility to view the newly configured partition on each of the 3 disks.

image

Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
sudo yum install lvm2 -y
sudo lvmdiskscan 
Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
Verify that your Physical volume has been created successfully by running sudo pvs
sudo pvs
image

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
Verify that your VG has been created successfully by running
sudo vgs
image

Use lvcreate utility to create 2 logical volumes. app-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: app-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
sudo lvcreate -n app-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
Verify that your Logical Volume has been created successfully by running
sudo lvs
image

Verify the entire setup
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
Use mkfs.ext4 to format the logical volumes with ext4 filesystem
sudo mkfs.ext4 /dev/vg-webdata-vg/app-lv
sudo mkfs.ext4 /dev/vg-webdata-vg/logs-lv
Create /var/www/html directory to store website files
sudo mkdir -p /var/www/html
Create /home/recovery/logs to store backup of log data
sudo mkdir -p /home/recovery/logs
Mount /var/www/html on app-lv logical volume
sudo mount /dev/vg-webdata/app-lv /var/www/html/
Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
sudo rsync -av /var/log/. /home/recovery/logs/
Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
sudo mount /dev/vg-webdata/logs-lv /var/log
Restore log files back into /var/log directory
sudo rsync -av /home/recovery/logs/. /var/log
Update /etc/fstab file so that the mount configuration will persist after restart of the server.
UPDATE THE /ETC/FSTAB FILE
The UUID of the device will be used to update the /etc/fstab file;

sudo blkid
image

sudo vi /etc/fstab
Step 1 - Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
image

Test the configuration and reload the daemon
sudo mount -a
sudo systemctl daemon-reload
Verify your setup by running df -h, output must look like this:
df -h
image

Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of app-lv create db-lv and mount it to /db directory instead of /var/www/html/.

Step 3 — Install WordPress on your Web Server EC2
Update the repository
sudo yum -y update
Install wget, Apache and it’s dependencies
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
Start Apache
sudo systemctl enable httpd
sudo systemctl start httpd
To install PHP and it’s dependencies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
https://www.tecmint.com/install-php-8-on-centos/

Restart Apache
sudo systemctl restart httpd
Download wordpress and copy wordpress to var/www/html
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp -R wordpress/wp-config-sample.php wordpress/wp-config.php
cd wordpress
sudo cp -R wordpress/. /var/www/html/  
Configure SELinux Policies
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
Step 4 — Install MySQL on your DB Server EC2
sudo yum update
sudo yum install mysql-server -y
Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo systemctl status mysqld
Step 5 — Configure DB to work with WordPress
sudo mysql
CREATE DATABASE wordpress;

CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpress';
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

SHOW DATABASES;
exit
image

image

Step 6 — Configure WordPress to connect to remote database.
Set this up if you did not allow all traffic in your security group Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
sudo vi /etc/my.cnf
image

From the web server

sudo vi wp-config.php
update the DB_NAME, DB_USER DB_PASSWORD and DB_HOST

image

sudo systemctl restart httpd
Change permissions and configuration so Apache could use WordPress on the webserver:
https://www.thegeekdiary.com/how-to-disable-the-default-apache-welcome-page-in-centos-rhel-7/#:~:text=Method%201%20%3A%20removing%2Frenaming%20Welcome%20Page&text=In%20order%20to%20disable%20this,you%20don't%20need%20it.

sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
sudo yum install mysql
sudo mysql -u wordpress -p -h <DB-Server-Private-IP-address>
Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
show databases;
exit mysql

image

select user, host from mysql.user;
sudo chown -R apache:apache /var/www/html/
image

sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1
image

Try to access from your browser the link to your WordPress http:///wordpress/
you see this:

image

Fill out your DB credentials and click on install wordpress:

image

If you see this message – it means your WordPress has successfully connected to your remote MySQL database

image

image

Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

You have learned how to configure Linux storage susbystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!
