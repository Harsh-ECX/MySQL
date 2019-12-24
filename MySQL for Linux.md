CLUSTERPRO X for Linux MySQL HowTo
===

はじめに
---
本ガイドでは、MySQL 8.0 と CLUSTERPRO を使用した環境の構築手順や設定例を紹介します。  
CLUSTERPRO X の詳細については、[こちら](https://jpn.nec.com/clusterpro/clpx/index.html)をご参照ください。


構成
---
本構成では、2node構成のミラーディスク型クラスタ(以下 Server1 / Server2) を構築します。  
CLUSTERPROで管理する ミラーディスク内に、MySQLのデータベースを構築することで、MySQL の可用性向上を目指します。

### 使用ソフトウェア
- MySQL 8.0(内部バージョン:8.0.17)

- CLUSTERPRO X 4.1 for Linux (内部バージョン：4.1.2-1)
- CLUSTERPRO X Replicator for Linux
- CLUSTERPRO X Database Agent for Linux

### クラスタ構成
- グループリソース
  - execリソース
  - フローティング IP リソース
  - ミラーディスクリソース
- モニタリソース
  - フローティング IP モニタリソース
  - ミラーコネクトモニタリソース
  - ミラーディスクモニタリソース
  - mysqlモニタリソース

CLUSTERPRO 環境下での MySQLの設定
---
CLUSTERPRO環境下でMySQLを設定する場合、非クラスタ環境の場合と以下の点が異なりますのでご注意ください。
- ユーザデータベースは、必ず CLUSTERPRO で管理するミラー用ディスクに格納する必要があります。
  データベースクラスタの作成や、データベースの作成は、現用系サーバからのみ行います。


構築手順
---
1. CLUSTERPRO の設定  
    - 以下のような 2ノードクラスタ構成環境を想定し、説明を行う。

    ###### クラスタ環境
    ||サーバ1(現用系)|サーバ2(待機系)|
    |---|---|---|
    |サーバ名|Server1|Server2|
    |実IPアドレス|192.168.1.1|192.168.1.2|  
    |クラスタパーティション|/dev/sdb1|/dev/sdb1|
    |データパーティション|/dev/sdb2|/dev/sdb2|
    
    ###### フェイルオーバグループ情報  
    |設定項目|設定例|
    |---|---|
    |グループ名|failover1|
    |フェイルオーバポリシ| Server1 -> Server2 |
    |フローティングIPアドレス|192.168.1.10|
    |ミラーディスクリソース(マウントポイント)|/mnt/md1|
    
    - CLUSTERPRO の WebUI を使用し、MySQLの運用に使用するフェイルオーバグループを作成します。  
      フェイルオーバグループには以下のリソースが必要となります。
      - フローティングIPリソース  
      - ミラーディスクリソース  
    
      上記のリソースを追加する手順については、CLUSTERPRO X 4.1 for Linux インスタンス&設定ガイド  
      「第5章 クラスタ構成情報を作成する」 をご参照ください。  
     フェイルオーバグループを作成し、クラスタ構成情報をアップロードしてサーバに反映させた後、  
     フェイルオーバグループを現用系サーバで起動します。

2. 両サーバにMySQLのインストール（rpm）
    - 以下の順番でrpmをインストールする
     1. mysql-community-common-8.0.17-1.el7.x86_64.rpm
     2. mysql-community-libs-8.0.17-1.el7.x86_64.rpm
     3. mysql-community-client-8.0.17-1.el7.x86_64.rpm
     4. mysql-community-server-8.0.17-1.el7.x86_64.rpm

3. サーバ共通 MySQLの設定
    - MySQL サービスの 自動起動を OFF にする
      - systemctl disable mysqld
    
    - MySQL 初期設定
        - MySQLの設定ファイル(/etc/my.cnf)を以下に設定する
          > [mysqld]  
          > datadir=/mnt/md1/mysql  
          > socket=/mnt/md1/mysql/mysql.sock  
          > [client]  
          > socket=/mnt/md1/mysql/mysql.sock  
            - 他の箇所はデフォルト設定でOK
     
3. Server1(現用系)でのMySQLの設定
    - データベースディレクトリの作成
        - mkdir -p /mnt/md1/mysql
    
    - MySQL サービスの起動
        - systemctl start mysqld

    - 管理者アカウントの設定
        - mysql_secure_installation
            - rootパスワード等の変更
    - データベースの作成
         - mysql -u root -p
            - root でログイン
         - CREATE DATABASE db_test;
            - db_test という名前の データベースを作成
                
    - データベースを実行するユーザ、パスワードを作成
         - CREATE USER 'test_user'@'localhost' IDENTIFIED BY 'aaaaaa';

    - 作成したユーザに権限を付与
         - GRANT ALL PRIVILEGES ON db_test.* TO 'test_user'@'localhost';

    - 設定の反映
         - FLUSH PRIVILEGES;

    - データベースの作成
         - mysql -u test_user -p
            - test_user でログイン
         - use db_test
            - 使用するdatabaseの指定
         - create table user(id int, name varchar(10));
            - tableの作成
         - create index id_index on user(id);
            - indexの作成
         - insert into user values(1, 'Yamada');
         - insert into user values(2, 'Suzuki');
            - 値の挿入
         - select * from user;
            - 以下のdatabaseが構築されたことを確認

              |id|name|
              |---|---|
              |1|Yamada|
              |2|Suzuki|
              
    - MySQL サービスの停止
         - systemctl stop mysqld

4. Server2(待機系)でのMySQLの設定  
    フェイルオーバグループを待機系サーバに移動します。待機系サーバにてMySQLの設定を行います。  
    - MySQL サービスの起動
        - systemctl start mysqld
        
    - 管理者アカウントの設定
        - mysql_secure_installation
            - rootパスワード等の変更
      
    - データベースの確認
         - mysql -u test_user -p
              - test_user でログイン
         - use db_test
              - 使用するdatabaseの指定
         - select * from user;
            - 以下のdatabaseが構築されたことを確認(databaseがミラーリングされていることを確認)

              |id|name|
              |---|---|
              |1|Yamada|
              |2|Suzuki|
              
    - MySQL サービスの停止
         - systemctl stop mysqld
            
 5. CLUSTERPROに設定を追加
      - EXECリソースの追加  
        CLUSTERPRO WebUI を使用して、MySQLの起動・停止用スクリプトを実行するEXECリソースを追加します。  
          - start.bat / stop.bat を 修正
            -  start.bat の場合 -> "$CLP_DISK" = "SUCCESS" の直後に systemctl start mysqld と追記
            -  stop.bat の場合 -> "$CLP_DISK" = "SUCCESS" の直後に systemctl stop mysqld と追記
      - MySQL モニタリソースの追加
          - 以下のパラメータを修正

              |設定項目|値|
              |---|---|
              |監視(共通) > 監視タイミング > 対象リソース|execリソース名|
              |監視(固有) > データベース名|db_test|
              |監視(固有) > ユーザ名|test_user|
              |監視(固有) > パスワード|aaaaaa|
              |監視(固有) > ライブラリパス|/usr/lib64/mysql/libmysqlclient.so.21|
              
      - 設定の反映を実施
      

6. 動作確認
    - CLUSTERPRO の フェイルオーバグループが起動している箇所で database にアクセスできることを確認
