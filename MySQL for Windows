MySQL with EXPRESSCLUSTER X on Windows
===

About this guide
---
This guide describes how to setup MySQL with EXPRESSCLUSTER X. 
For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


Configuration
---
In this setup, create 2 nodes (Node1 and Node2 as below) mirror disk type cluster.
Achieving MySQL High Availability By using EXPRESSCLUSTER X. 

### Software versions
- MySQL 8.0(internal version:8.0.26) 
- CLUSTERPRO X 4.3 for Windows 
- CLUSTERPRO X Replicator for Windows

### Cluster configurations
- Group resources
  - Service resource
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
- Database have to create Mirror disk that maneged by EXPRESSCLUSTER.
  
Procedure
---
1. EXPRESSCLUSTER setup  

    - We assume the following 2node cluster and explain it.

    ### cluster information
    ||Node1(Active)|Node2(Stanby)|
    |---|---|---|
    |Server name|Server1|Server2|
    |IPaddress|192.168.1.1|192.168.1.2|  
    |Cluster Partition|D:|D:|
    |Data Partition|E:|E:|
    
    ### failover group information  
    |parameter|value|
    |---|---|
    |name|failover1|
    |Startup Server| Server1 -> Server2 |
    |floating ip address|192.168.1.10|
    |mirror disk resource | E:/data/
    
    - In Config mode of the Cluster WebUI, add failover group to use MySQL.  
      You need the following resources.
      - Floating ip resource  
      - Mirror disk resource
    
     If want to know how to add the resource, please refer to "EXPRESSCLUSTER X 4.3 for Windows Installation and Configuration Guide". 
     After you add failver group and execute apply the configuration file, you start failover group by server1.  
     
2. Install MySQL on both servers

    - Install MySQL 8.0.26 on both the servers.
    - For installation procedure please refer to [this site](https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/ ).

3. MySQL Configuration for Mirror disk (Node1)

    - Create the database directory.
        - create data directory in mirror drive e.g. E:/data/MySQL

    - Coping MySQL data from default location to Mirror Disk.
    
      - Stop mysql service from services.msc.
      - Copy MySQL data from default directory (c:\ProgramData\MySQL) to Mirror disk (E:\Data\MySQL)
      
4. Perform the below steps on both the Nodes one by one after group failover move.

      - Configure the MySQL Configuration file (c:\ProgramData\MySQL\my.ini).
          > [mysqld]  
          > datadir=E:\Data\MySQL 
    
      - The original encoding is UTF-8, but it was changed to UTF-8 BOM after changing the mirror drive path in my.ini. There is two method to remove BOM encoding as follwing.
          > CMD method

            powershell -NoProfile -ExecutionPolicy Unrestricted -Command "& { $MyPath = 'my.ini';$MyFile = Get-Content $MyPath; $Utf8NoBomEncoding = New-Object System.Text.UTF8Encoding($False);[System.IO.File]::WriteAllLines($MyPath, $MyFile, $Utf8NoBomEncoding)}"
            
            This command removes BOM from UTF-8 BOM file, and also doesn't affect UTF-8 file without BOM.

    OR 

          > Text Editor Method

            BOM from my.ini can be edit other text editors (e.g. Sakura editor), then MySQL service is running fine. It can be change from any editor make sure the encoding it shloud be UTF-8 of My.ini file.

5. MySQL Setup (Node1)
        
    - Start the MySQL service to check it is working fine.
    
        - From services.msc start the MySQL Service.
        - From services.msc stop the MySQL Service.


6. Configure the MySQL Service in Cluster
 
      - Add the service resource and configure. 
       
      - In Config mode of the Cluster WebUI, Add the service resource to control MySQL.  
                                 
      - In Config mode of the Cluster WebUI, execute Apply the Configuration File.
      
      
7. Verification

    - Confirm that we can access the database where it failover group is running.
