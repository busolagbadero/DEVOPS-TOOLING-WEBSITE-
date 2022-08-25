#  IMPLEMENTING THREE WEB SERVERS WITH A COMMON DATABASE AND NFS SERVER THAT SERVES AS A FILE SHARE STORAGE.

 Spinned up  3 RHEL webservers , 1 NFS server(RHEL) and 1 Database Server (UBUNTU) on AWS 

![pp1](https://user-images.githubusercontent.com/94229949/186647214-4290b32c-eee7-46c6-81d4-14367d4b7765.png)

## PREPARE NFS SERVER

 Spun up a EC2 instance 

 Created 3 volumes with 10G

![pp2](https://user-images.githubusercontent.com/94229949/186649027-c7e08d4d-5b55-4c8d-8c04-2312c9d64ca5.png)

Attached  volumes  to the NFS server 


 SSH into the instance using powershell 

![pp3](https://user-images.githubusercontent.com/94229949/186649236-139cf839-1751-4c6d-8830-0f4030ff0581.png)

 Installed lvm2 and then Created Physical volume using 'sudo pvcreate' ,volume Group called 'webdata-vg' using 'vgcreate'  and three logical volumes named ' lv-opt lv-apps, and lv-logs ' using 'lvcreate' and then formatted the disk with 'XFS'

![pp5](https://user-images.githubusercontent.com/94229949/186651224-64e234e6-478d-45c6-b45b-eedf028ec30e.png)

![pp6](https://user-images.githubusercontent.com/94229949/186651853-5984e983-92b9-489b-b27f-939775d6fd59.png)

Created mount points on /mnt directory for the logical volumes as follow:
Mount lv-apps on /mnt/apps – To be used by webservers
Mount lv-logs on /mnt/logs – To be used by webserver logs
Mount lv-opt on /mnt/opt 

![ppee](https://user-images.githubusercontent.com/94229949/186653405-17e3d511-1a71-46a7-9f80-0980ef38e770.png)

Installed NFS server, configured it to start on reboot using these commands 
'sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service'

set up permission that will allow  Web servers to read, write and execute files on NFS using these commands '

sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service '

Configured access to NFS for clients within the same subnet using ' sudo vi /etc/exports '

  For NFS server to be accessible from the client, i opened the  following ports: TCP 111, UDP 111, UDP 2049
  
  ![pp7](https://user-images.githubusercontent.com/94229949/186656069-f1fc79ab-7371-4d9e-b989-916e788ac99b.png)

![pp8](https://user-images.githubusercontent.com/94229949/186656117-b1165522-a6bc-4abd-852d-7d5724630132.png)

![pp9](https://user-images.githubusercontent.com/94229949/186656149-b28ff1f4-0472-4a28-88ab-5b67f8fc2c98.png)


## CONFIGURE THE DATABASE SERVER

Installed MySQL server

Created a database and name it 'tooling'

Created a database user and name it 'webaccess'

Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![pp10](https://user-images.githubusercontent.com/94229949/186657459-ee1d8ec3-d366-45e6-ad3e-19255e40191c.png)

## Prepare the Web Servers

Spun up three web servers 

Installed NFS client using 'sudo yum install nfs-utils nfs4-acl-tools -y '

 Verified that NFS was mounted successfuly by running df -h and made sure that the changes persisted on the web server after reboot.
 
 ![pp11](https://user-images.githubusercontent.com/94229949/186659861-128eb537-52d8-422e-842c-267d44d24a4a.png)

Install Remi’s repository, Apache and PHP 

![pp12](https://user-images.githubusercontent.com/94229949/186660544-c5e57fef-e9bc-4a0e-9099-214645c96d24.png)

To Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. I created  new files 'test.txt' ' touch.md '  from one server and check if the same file is accessible from other Web Servers.

![pp18a](https://user-images.githubusercontent.com/94229949/186661411-29be43d3-8790-4e6d-b2c6-3ff45bfa0b21.png)

![pp18b](https://user-images.githubusercontent.com/94229949/186661463-5489b69e-e1be-4c2b-ac5d-d0185262048c.png)

![pp18c](https://user-images.githubusercontent.com/94229949/186661499-db97a8a3-6195-49f4-8582-7fd8de452de9.png)

Located the log folder for Apache on the web server, mounted it to the NFS server's export for logs i.e /var/log/httpd, confirmed that the NFS was successfully mounted and the changes persisted on the web server.

![pppppp0](https://user-images.githubusercontent.com/94229949/186662336-7d93a16b-b667-4ca1-94b6-cab70bec899a.png)

Installed git, initialized and forked the tooling website's code to the webserver.

![pp13](https://user-images.githubusercontent.com/94229949/186662750-41f82536-86c5-4081-9223-57d79904d901.png)

Deployed the tooling webiste's code to the webserver, and ensured that the html folder from the repository was deployed to /var/www/html. Disabled SELinux using the sudo setenforce 0 command. To make the changes permanent, I opened the /etc/sysconfig/selinux config file and set SELINUX=disabled and then restarted httpd.

![pp14](https://user-images.githubusercontent.com/94229949/186663395-50e258bd-aaad-4cc5-8a04-f8e5127dc7d2.png)

Updated the website’s configuration to connect to the database (in /var/www/html/functions.php file). Applied tooling-db.sql script to the database using this command

![pp15](https://user-images.githubusercontent.com/94229949/186664102-f84ea35b-2fb3-4c26-af01-b548b94c86d0.png)


Installed mysql on my webserver

![pppp22](https://user-images.githubusercontent.com/94229949/186665706-d5d7d99f-232b-4645-851e-6d59c66713d2.png)

Opened the TCP port 80 on the web server

![nn](https://user-images.githubusercontent.com/94229949/186667400-330e16d1-de99-4457-821c-3005eb1705cc.png)

Opened the webiste in my browser http://44.201.127.118/admin_tooling.php

![pp16](https://user-images.githubusercontent.com/94229949/186667700-7bf0ac4d-5831-4e17-a5e5-34b4a2021458.png)

![pp17](https://user-images.githubusercontent.com/94229949/186667749-ffcdeafc-cc1b-45a7-9d1d-b7353699d12c.png)




