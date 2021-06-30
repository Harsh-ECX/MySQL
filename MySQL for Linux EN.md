MYSQL with EXPRESSCLUSTER X on Linux
===

About this guide
---
This guide describes how to setup MySQL with EXPRESSCLUSTER X. 
For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


Configuration
---
In this setup, create 2 nodes (Node1 and Node2 as below) mirror disk type cluster.
Achieving MySQL high availability By using EXPRESSCLUSTER X. 

### Software versions
- MySQL 8.0(internal version:8.0.17) 
             
  OR
- MySQL 8.0(internal version:8.0.21)
- CLUSTERPRO X 4.1/2/3 for Linux 
- CLUSTERPRO X Replicator for Linux
- CLUSTERPRO X Database Agent for Linux

### Cluster configurations
- Group resources
  - exec resource
  - floting IP resource
  - mirror disk resource
  
- Monitor rerources
  - floting IP resource
  - mirror disk connect monitor resource
  - mirror disk monitor resource
  - mysql monitor resource

MySQL setup
---
Please note that the following points are different if you set MySQL to EXPRESSCLUSTER.
- database have to create Mirror disk that maneged by EXPRESSCLUSTER.
  You have to set only active server if you create database and database cluster.


Procedure
---
1. EXPRESSCLUSTER setup  

    - We assume the following 2node cluster and explain it.

    ### cluster information
    ||Node1(Active)|Node2(Stanby)|
    |---|---|---|
    |Server name|Server1|Server2|
    |IPaddress|192.168.1.1|192.168.1.2|  
    |cluster partition|/dev/sdb1|/dev/sdb1|
    |data partition|/dev/sdb2|/dev/sdb2|
    
    ### failover group information  
    |parameter|value|
    |---|---|
    |name|failover1|
    |Startup Server| Server1 -> Server2 |
    |floating ip address|192.168.1.10|
    |mirror disk resource (mount point))|/mnt/md1|
    
    - In Config mode of the Cluster WebUI, add failover group to use MySQL.  
      You need the following resources.
      - Floating ip resource  
      - Mirror disk resource
    
     If want to know how to add the resource, please refer to "EXPRESSCLUSTER X 4.1/2/3 for Linux Installation and Configuration Guide". 
     After you add failver group and execute apply the configuration file, you start failover group by server1.  
     
2. Install MySQL on both servers

    - Install rpm files in the following order 
     1. mysql-community-common-8.0.17-1.el7.x86_64.rpm
     2. mysql-community-libs-8.0.17-1.el7.x86_64.rpm
     3. mysql-community-client-8.0.17-1.el7.x86_64.rpm
     4. mysql-community-server-8.0.17-1.el7.x86_64.rpm
            
    OR  
       (Run the below mentioned command to install mysql using yum )

     1.  sudo yum install mysql-server  

           Note :- Please visit [this site](https://www.server-world.info/query?os=CentOS_8&p=mysql8&f=1) if any problems arise with the installation and setup of MYSQL

    - Configure the root account
        - mysql_secure_installation 
        
3. MySQL Configuration for Mirror disk (Node1)

    - Create the database directory.
        - mkdir -p /mnt/md1/mysql

    - Coping MySQL data from default location to Mirror Disk.
    
      - systemctl status mysqld
      - systemctl stop mysqld
      - sudo rsync -av /var/lib/mysql /mnt/md1/mysql
      
4. Perform the below steps on both the Nodes.

    - Initialize the MySQL 8.0(internal version:8.0.17)
        - Configure the MySQL Configuration file (/etc/my.cnf).
          > [mysqld]  
          > datadir=/mnt/md1/mysql  
          > socket=/mnt/md1/mysql/mysql.sock  
          > [client]  
          > socket=/mnt/md1/mysql/mysql.sock  
            - No other configuring required.

          or
              
      - Initialize the MySQL 8.0(internal version:8.0.21)
        - Configure the MySQL Configuration file (/etc/my.cnf.d/mysql-server.cnf).
          > [mysqld] 
          > datadir=/mnt/md1/mysql

          > socket=/mnt/md1/mysql/mysql.sock 
          
        - Configure the MySQL Client Configuration file (/etc/my.cnf.d/client.cnf).
          > [client]

          > port=3306
          
          > socket=/mnt/md1/mysql/mysql.sock 
            - No other configuring required.        
     
5. MySQL Setup (Node1)
        
    - Run the MySQL service.
        - systemctl start mysqld

    - Create database 
         - mysql -u root -p
            - log in to root
         - CREATE DATABASE db_test;
            - Create database named db_test.
                
    - Create user and passward for executing database. 
         - CREATE USER 'test_user'@'localhost' IDENTIFIED BY 'aaaaaa';

    - Grant permissions.
         - GRANT ALL PRIVILEGES ON db_test.* TO 'test_user'@'localhost';

    - Apply the Configuration file.
         - FLUSH PRIVILEGES;

    - Create database
         - mysql -u test_user -p
            - Log in to test_user.
         - use db_test
            - Specify using database.
         - create table user(id int, name varchar(10));
            - Create table.
         - create index id_index on user(id);
            - Create index.
         - insert into user values(1, 'Yamada');
         - insert into user values(2, 'Suzuki');
            - Insert values.
         - select * from user;
            - Confirm the database.

              |id|name|
              |---|---|
              |1|Yamada|
              |2|Suzuki|
              
    - Stop MySQL service
             - systemctl stop mysqld

6. MySQL Setup (Node2)

    You have to move failover group on the Node2. Configure MySQL on the Node2.
    - Start MySQL service.
        - systemctl start mysqld
        
    - Configure the root account
        - mysql_secure_installation
      
    - Confime the database
         - mysql -u test_user -p
              - Log in to "test_user"
         - use db_test
              - Specify 
         - select * from user;
            - Confirm the database.

              |id|name|
              |---|---|
              |1|Yamada|
              |2|Suzuki|
              
    - Stop MySQL service.
         - systemctl stop mysqld
            
 7. Configure the EXPRESSCLUSTER
 
      - Add the exec resource and configure. 
       
        In Config mode of the Cluster WebUI, Add the exec resource to control MySQL.  
          - Configure the start.sh and stop.sh
            -  In the case of start.sh -> Immediately after "$CLP_DISK" = "SUCCESS", add the "systemctl start mysqld"
            -  In the case of stop.sh  -> Immediately after "$CLP_DISK" = "SUCCESS", add the "systemctl stop mysqld"
           
      - Add the MySQL monitor resource
          - Configure the folloing parameters

              |parameter|value|
              |---|---|
              |Monitor(common) > Monitor Timing > Target Resource|setting the exec resource name|
              |Monitor(special) > Database Name|db_test|
              |Monitor(special) > User Name|test_user|
              |Monitor(special) > password|aaaaaa|
              |Monitor(special) > Library Path|/usr/lib64/mysql/libmysqlclient.so.21|
              
      - In Config mode of the Cluster WebUI, execute Apply the Configuration File.
      

8. Verification

    - Confirm that we can access the database where it failover group is running.
