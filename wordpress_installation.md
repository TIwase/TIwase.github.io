#### 2020/05/03 追記


# EC2(Amazon Linux 2) + wordpressのインストール

構築した環境 (2020/05/03現在):

|名前|version|
|----|----|
|wordpress|5.4.1
|Apache|2.4|
|php|7.4|
|mariaDB|5.5.64|

※AWS Consoleの設定等は割愛

## 01. mariaDBのインストール

	$ sudo yum -y install MariaDB-server MariaDB-client 

mariaDBを起動する

	$ sudo systemctl start mariadb  
	$ sudo systemctl enable mariadb  
	$ sudo systemctl status mariadb 

(表示例)
> ● mariadb.service - MariaDB 10.3.20 database server                                                   
>    Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)         
>   Drop-In: /etc/systemd/system/mariadb.service.d                                                       
>            mqmigrated-from-my.cnf-settings.conf                                                        
>    Active: active (running) since Sun 2019-12-01 13:51:14 UTC; 8s ago                                  
>      Docs: man:mysqld(8)                                                                               
>            https://mariadb.com/kb/en/library/systemd/                                                  
>   Process: 3753 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited  
> , status=0/SUCCESS)                                                                                      
>   Process: 3710 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/ga  
> lera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=  
> exited, status=0/SUCCESS)                                                                                
>   Process: 3707 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited,  
>  status=0/SUCCESS)                                                                                     
>  Main PID: 3722 (mysqld)                                                                               
>    Status: "Taking your SQL requests now..."                                                           
>    CGroup: /system.slice/mariadb.service                                                               
>            mq3722 /usr/sbin/mysqld                                                                     
>                                                                                                        
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] InnoDB: 10...21  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] InnoDB: Lo...ol  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] Plugin 'FE...d.  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] InnoDB: Bu...14  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] Server soc...'.  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] Reading of...ed  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] Added new ...le  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: 2019-12-01 13:51:14 0 [Note] /usr/sbin/...s.  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal mysqld[3722]: Version: '10.3.20-MariaDB'  socket: '/v...er  
> Dec 01 13:51:14 ip-172-31-94-88.ec2.internal systemd[1]: Started MariaDB 10.3.20 database server.        
> Hint: Some lines were ellipsized, use -l to show in full.                                              

rootユーザのパスワードを設定する

	$ sudo mysql_secure_installation

(表示例)

> NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB  
>       SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!  
> 
> In order to log into MariaDB to secure it, we'll need the current  
> password for the root user.  If you've just installed MariaDB, and  
> you haven't set the root password yet, the password will be blank,  
> so you should just press enter here.  
> 
> Enter current password for root (enter for none):  
> OK, successfully used password, moving on...  
> 
> Setting the root password ensures that nobody can log into the MariaDB  
> root user without the proper authorisation.  
> 
> Set root password? [Y/n] y  
> New password:  
> Re-enter new password:  
> Password updated successfully!  
> Reloading privilege tables..  
>  ... Success!  
> 
> 
> By default, a MariaDB installation has an anonymous user, allowing anyone  
> to log into MariaDB without having to have a user account created for  
> them.  This is intended only for testing, and to make the installation  
> go a bit smoother.  You should remove them before moving into a  
> production environment.  
> 
> Remove anonymous users? [Y/n] y  
>  ... Success!  
>  
> Normally, root should only be allowed to connect from 'localhost'.  This  
> ensures that someone cannot guess at the root password from the network.  
> 
> Disallow root login remotely? [Y/n] y  
>  ... Success!  
> 
> By default, MariaDB comes with a database named 'test' that anyone can  
> access.  This is also intended only for testing, and should be removed  
> before moving into a production environment.  
> 
> Remove test database and access to it? [Y/n] y  
>  - Dropping test database...  
>  ... Success!  
>  - Removing privileges on test database...  
>  ... Success!  
> 
> Reloading the privilege tables will ensure that all changes made so far  
> will take effect immediately.  
> 
> Reload privilege tables now? [Y/n] y  
>  ... Success!  
> 
> Cleaning up...  
> 
> All done!  If you've completed all of the above steps, your MariaDB  
> installation should now be secure.  
> 
> Thanks for using MariaDB!

※全部yでもいいかもです

rootユーザでログイン

	$ mysql -u root -p

(表示例)
> Enter password:  
> Welcome to the MariaDB monitor.  Commands end with ; or \g.
> Your MariaDB connection id is 10
> Server version: 5.5.64-MariaDB MariaDB Server
> 
> Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
> 
> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

データベースの作成

	> CREATE DATABASE `wordpress-db`;
	> CREATE USER 'user'@'localhost' IDENTIFIED BY 'userpass';
	> GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "user"@"localhost";
	> FLUSH PRIVILEGES;  
	> exit

※wordpress-dbにデータベース名、userに任意のユーザ名、userpassにユーザ名の任意のパスワードを設定する

## 02. Apacheインストール
	$ sudo yum -y install httpd 
	$ httpd -version
※インストールされていることを確認

Apacheの起動

	$ sudo systemctl start httpd 
	$ sudo systemctl enable httpd 
	$ sudo systemctl status httpd

(表示例)
> ● httpd.service - The Apache HTTP Server                                                              
>    Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)           
>    Active: active (running) since Sun 2019-12-01 14:08:04 UTC; 9s ago                                  
>      Docs: man:httpd.service(8)                                                                        
>  Main PID: 4005 (httpd)                                                                                
>    Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"   
>    CGroup: /system.slice/httpd.service                                                                 
>            tq4005 /usr/sbin/httpd -DFOREGROUND                                                         
>            tq4006 /usr/sbin/httpd -DFOREGROUND                                                         
>            tq4007 /usr/sbin/httpd -DFOREGROUND                                                         
>            tq4008 /usr/sbin/httpd -DFOREGROUND                                                         
>            tq4009 /usr/sbin/httpd -DFOREGROUND                                                         
>            mq4010 /usr/sbin/httpd -DFOREGROUND                                                         
>                                                                                                        
> Dec 01 14:08:04 ip-172-31-94-88.ec2.internal systemd[1]: Starting The Apache HTTP Server...            
> Dec 01 14:08:04 ip-172-31-94-88.ec2.internal systemd[1]: Started The Apache HTTP Server.               

IPアドレス先orパブリックDNS名でアクセスしてApacheのテストページが表示されることを確認する

## 03. phpのインストール

現在のphpのバージョンを確認

	$ php -v

(表示例)
> PHP 5.4.16 (cli) (built: Oct 31 2019 18:34:05)  
> Copyright (c) 1997-2013 The PHP Group  
> Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies  

インストール可能なphpパッケージの確認

	$ sudo amazon-linux-extras list | grep php

(表示例)
> 15  php7.2                   available    \
> 17  lamp-mariadb10.2-php7.2  available    \
> 31  php7.3                   available    \
> 42  php7.4                   available    [ =stable ]

現時点で最新版のphp7.4にアップグレードする

	$ sudo amazon-linux-extras install php7.4
	$ sudo systemctl restart httpd

Apacheでphpが正常に動作することを確認

	$ cd /var/www/html
	$ sudo vi phpinfo.php

viエディタを開き、インサートモードで下記を追加する

	<?php
	phpinfo();
	?>

http://<IPアドレス先>/phpinfo.phpあるいはhttp://<パブリックDNS名>/phpinfo.phpでアクセスしてphpのテストページが表示されることを確認する

## 04. wordpressのインストール  
 
ディレクトリの移動

	$ cd /var/www/html

wordpressの公式サイトからファイルをダウンロードする 

	英語ver.
	$ wget https://wordpress.org/latest.tar.gz  
	$ tar -xzvf latest.tar.gz 
	
	日本語ver.
	$ wget http://ja.wordpress.org/latest-ja.zip  
	$ unzip latest-ja.zip 
(表示例)
> --2019-12-01 13:45:19--  https://wordpress.org/latest.tar.gz                                           
> Resolving wordpress.org (wordpress.org)... 198.143.164.252                                             
> Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.                         
> HTTP request sent, awaiting response... 200 OK                                                         
> Length: 12372564 (12M) [application/octet-stream]                                                      
> Saving to: ‘latest.tar.gz’                                                                           
>                                                                                                        
> 100%[=============================================================>] 12,372,564  30.8MB/s   in 0.4s    
>                                                                                                        
> 2019-12-01 13:45:20 (30.8 MB/s) - ‘latest.tar.gz’ saved [12372564/12372564]                          




	$ ls -l wordpress/  

(表示例)
> total 204                                                                                              
> -rw-r--r--  1 ec2-user ec2-user   420 Nov 30  2017 index.php                                           
> -rw-r--r--  1 ec2-user ec2-user 19935 Jan  1  2019 license.txt                                         
> -rw-r--r--  1 ec2-user ec2-user  7368 Sep  2 21:44 readme.html                                         
> -rw-r--r--  1 ec2-user ec2-user  6939 Sep  3 00:41 wp-activate.php                                     
> drwxr-xr-x  9 ec2-user ec2-user  4096 Nov 12 20:31 wp-admin                                            
> -rw-r--r--  1 ec2-user ec2-user   369 Nov 30  2017 wp-blog-header.php                                  
> -rw-r--r--  1 ec2-user ec2-user  2283 Jan 21  2019 wp-comments-post.php                                
> -rw-r--r--  1 ec2-user ec2-user  2898 Jan  8  2019 wp-config-sample.php                                
> drwxr-xr-x  4 ec2-user ec2-user    52 Nov 12 20:31 wp-content                                          
> -rw-r--r--  1 ec2-user ec2-user  3955 Oct 10 22:52 wp-cron.php                                         
> drwxr-xr-x 20 ec2-user ec2-user  8192 Nov 12 20:31 wp-includes                                         
> -rw-r--r--  1 ec2-user ec2-user  2504 Sep  3 00:41 wp-links-opml.php                                   
> -rw-r--r--  1 ec2-user ec2-user  3326 Sep  3 00:41 wp-load.php                                         
> -rw-r--r--  1 ec2-user ec2-user 47007 Sep 30 18:53 wp-login.php                                        
> -rw-r--r--  1 ec2-user ec2-user  8483 Sep  3 00:41 wp-mail.php                                         
> -rw-r--r--  1 ec2-user ec2-user 19120 Oct 15 15:37 wp-settings.php                                     
> -rw-r--r--  1 ec2-user ec2-user 31112 Sep  3 00:41 wp-signup.php                                       
> -rw-r--r--  1 ec2-user ec2-user  4764 Nov 30  2017 wp-trackback.php                                    
> -rw-r--r--  1 ec2-user ec2-user  3150 Jul  1 08:01 xmlrpc.php                                          

wp-config-sample.phpファイルがあることを確認

configファイルを複製し、wordpressのユーザ名とパスワードを設定する

	$ cp wordpress/wp-config-sample.php wordpress/wp-config.php  
	$ vi wordpress/wp-config.php  
	


(表示例)

> 中略...
> // ** MySQL settings - You can get this info from your web host ** //                                  
> /** The name of the database for WordPress */                                                          
> define( 'DB_NAME', 'database_name_here' );                                                                   
>                                                                                                        
> /** MySQL database username */                                                                         
> define( 'DB_USER', 'username_here' );                                                                         
>                                                                                                        
> /** MySQL database password */                                                                         
> define( 'DB_PASSWORD', 'password_here' );                                                                     
>                                                                                                        
> /** MySQL hostname */                                                                                  
> define( 'DB_HOST', 'localhost' );                                                                      
>                                                                                                        
> /** Database Charset to use in creating database tables. */                                            
> define( 'DB_CHARSET', 'utf8' );                                                                        
>                                                                                                        
> /** The Database Collate type. Don't change this if in doubt. */                                       
> define( 'DB_COLLATE', '' );                                                                            
> ...  

※database_name_hereにMySQLデータベース名、username_hereにMySQLユーザ名、password_hereにMySQLパスワードを記述する


wordpress配下にあるすべてのファイルをhtmlディレクトリへ移動し、権限を変更

	$ sudo mv wordpress/* /var/www/html/
	$ sudo rm -r wordpress
	$ sudo chown -R apache:apache /var/www/html/; ls -l

(表示例)                                                                                             
> total 216
> -rw-r--r--  1 apache apache   405 Feb  6 06:33 index.php  
> -rw-r--r--  1 apache apache 19915 Feb 12 11:54 license.txt  
> -rw-r--r--  1 root   root      20 May  2 14:40 phpinfo.php   
> -rw-r--r--  1 apache apache 10089 May  2 02:00 readme.html  
> -rw-r--r--  1 apache apache  6912 Feb  6 06:33 wp-activate.php  
> drwxr-xr-x  9 apache apache  4096 May  2 02:00 wp-admin  
> -rw-r--r--  1 apache apache   351 Feb  6 06:33 wp-blog-header.php  
> -rw-r--r--  1 apache apache  2275 Feb  6 06:33 wp-comments-post.php  
> -rw-r--r--  1 root   root    3915 May  2 14:13 wp-config.php  
> -rw-r--r--  1 apache apache  3931 May  2 02:00 wp-config-sample.php  
> drwxr-xr-x  5 apache apache    69 May  2 15:01 wp-content  
> -rw-r--r--  1 apache apache  3940 Feb  6 06:33 wp-cron.php  
> drwxr-xr-x 21 apache apache  8192 May  2 02:00 wp-includes  
> -rw-r--r--  1 apache apache  2496 Feb  6 06:33 wp-links-opml.php  
> -rw-r--r--  1 apache apache  3300 Feb  6 06:33 wp-load.php  
> -rw-r--r--  1 apache apache 47874 Feb 10 03:50 wp-login.php  
> -rw-r--r--  1 apache apache  8509 Apr 14 11:34 wp-mail.php  
> -rw-r--r--  1 apache apache 19396 Apr 10 03:59 wp-settings.php  
> -rw-r--r--  1 apache apache 31111 Feb  6 06:33 wp-signup.php  
> -rw-r--r--  1 apache apache  4755 Feb  6 06:33 wp-trackback.php  
> -rw-r--r--  1 apache apache  3133 Feb  6 06:33 xmlrpc.php  
 
再度、Apacheを再起動

	$ sudo systemctl restart httpd

http://<IPアドレス先>/wp-admin/install.phpあるいはhttp://<パブリックDNS名>/wp-admin/install.phpにアクセスしてwordpress登録画面が表示されればok

