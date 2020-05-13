#### 2020/05/10 更新

# EC2(Amazon Linux 2)上でALB(Application Load Balancer)とHTTPSを利用したwordpress運用方法

※EC2(Amazon Linux 2)にApacheとwordpressがインストールされていることが前提

→[EC2 + wordpressのインストールはここから](wordpress_installation.md)

## 01. 独自ドメイン取得

AWSのコンソールにログインし、Route53を選択。

任意のドメインを申請する(※詳しい内容は忘れたため省略)

## 02. SSL証明書の発行

ここではAWSサービスのACM(Amazon Certificate Manager)を利用する。

AWSのコンソールにログインし、[Certificate Manager] > [証明書のリクエスト]を選択。

↓

Route53で取得した独自ドメイン(サブドメイン含む)を追加して[次へ]を選択。

↓

検証方法は、[DNSの検証]にチェックを入れて[次へ]を選択。

↓

タグ名は任意で追加して[確認]を選択。(タグを追加しなくても問題ない)

↓

[検証とリクエスト]を選択。

↓

リクエストしたドメイン名の左にある▸をクリックし、表示されたドメインの▸をさらにクリックする。

↓

[Route53でのレコードの作成]ボタンが表示されるのでクリック。
(※Route53にあるDNS設定のCNAMEレコードに、証明書のリクエストした情報が自動で追加される。)

一定時間経過後、状況が検証保留中から発行済みに変わっていることが確認できればok (この時、更新資格は使用不可のままだった気がする)

## 03. ALB(Application Load Balancer)の作成

サービスからEC2を選択し、左のメニューからロードバランサーを選択。

↓

[ロードバランサーの作成]をクリックして、ALBの[作成]を選択。

任意の名前を記入し、~~[リスナーの追加]をクリックしてHTTPSを追加する。~~

ロードバランサーのプロトコルをHTTP->HTTPSに変更する。

↓

アベイラビリティゾーンに最低2つのサブネットを指定して[次の手順]を選択。

↓

セキュリティグループ設定では、~~HTTPS化するwordpressインスタンスが所属しているセキュリティグループ名を選択して[次の手順]をクリック。~~

[新しいセキュリティグループを作成する]にチェックを入れ、タイプにHTTPSを選択して[次の手順]をクリック。

↓

ルーティング設定では、新しいターゲットグループを選択して任意の名前を入力して[次の手順]をクリック。

↓

ターゲットの登録では、wordpressインスタンスを選択してポート80で[登録済みに追加]をクリック。

~~次に、ポート443で同様に[登録済みに追加]をクリック。~~

↓

最後にすべての設定内容を確認して相違がなければ[作成]をクリック。


## 04. セキュリティグループの設定

[EC2] > [セキュリティグループ]へ移動し、先ほど新しく作成したセキュリティグループの[インバウンドルール]を選択。

↓

**下記の表に記載されているプロトコルのみ**追加する。

|タイプ|プロトコル|ポート範囲|ソース|説明-オプション|
|--|--|--|--|--|
|HTTP|TCP|80|(※ELBのセキュリティグループ)|-|
|SSH|TCP|22|0.0.0.0/0|-|
|DNS(TCP)|TCP|53|0.0.0.0/0|-|
|HTTPS|TCP|443|0.0.0.0/0|-|

※この時、[EC2] > [インスタンス]のwordpressインスタンスに**このセキュリティグループのみ**関連付けられていることを確認する

## 05. Aレコードの追加

[Route53] > [ホストゾーン] > [レコードセットの作成]を選択する。

↓

名前にサブドメインを入力し、タイプはA-IPv4アドレス(Aレコード)、エイリアスの「はい」にチェックを入れる。

↓

エイリアス先に先ほど作成したロードバランサーの値を選択して[作成]をクリック。



## 06. wordpressサイト全体のHTTPS化

ターミナルを起動し、wordpressインスタンスにSSH接続してwp-config.phpを編集する。

	$ sudo vi /var/www/html/wp-config.php

下記を追加

	define('WP_SITEURL', 'https://<サブ + ドメイン名>'); // WordPressアドレス
	define('WP_HOME','<サブ + ドメイン名>'); // サイトアドレス

	$_SERVER['HTTPS']='on'; // 管理画面のhttps化
	define('FORCE_SSL_LOGIN', true);
	define('FORCE_SSL_ADMIN', true);
	

※以下の記述より必ず上に追記すること。

	/** Absolute path to the WordPress directory. */
	if ( ! defined( 'ABSPATH' ) ) {
	        define( 'ABSPATH', __DIR__ . '/' );
	}

	/** Sets up WordPress vars and included files. */
	require_once ABSPATH . 'wp-settings.php';


リダイレクト設定を変更する。

	$ sudo chown -R apache:apache /var/www/html/.htaccess
	$ sudo vi /var/www/html/.htaccess

下記を追加する。
	
	<IfModule mod_rewrite.c>  
	RewriteEngine On  
	RewriteCond %{HTTP:X-Forwarded-Proto} !https [NC]  
	RewriteCond %{REQUEST_URI} !^/health.html  
	RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]  
	</IfModule>

Apacheを再起動する

	$ sudo systemctl restart httpd


ブラウザを起動してHTTPSアクセスしてURLの一番左に鍵マークが表示されていればok

