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


## 
