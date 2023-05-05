# Hack a Web Server


## Scenario
"Roseau": Hack a Web Server


## Level
Hard


## Type
Hack


## Access
Public


## Description
There is a secret stored in a file that the local Apache web server can provide.
Find this secret and have it as a /home/admin/secret.txt file.

Note that in this server the admin user is not a sudoer.

Also note that the password crackers Hashcat and Hydra are installed from packages and John the Ripper binaries have been built from source in /home/admin/john/run


## Description(日本語)
ローカルのApacheウェブサーバーが提供できるファイルに保存されている秘密があります。
このシークレットを見つけ、/home/admin/secret.txt ファイルとして持っています。

このサーバでは、adminユーザはsudoerではないことに注意してください。

また、パスワードクラッカーのHashcatとHydraはパッケージからインストールされ、John the Ripperのバイナリは/home/admin/john/runにソースからビルドされていることに注意してください。


## Test
sha1sum /home/admin/secret.txt |awk '{print $1}' returns cc2c322fbcac56923048d083b465901aac0fe8f8


## Time to Solve
30 minutes.


## OS
Debian 11


## 回答

### Basic認証情報ファイルの特定

```bash
# 接続テスト
curl localhost
# 401 Unauthorizedとあり、認証エラーであることがわかる。

# 直接ファイルの中身を確認する
ls -al /var/www/
# 抜粋した出力は以下の通り
# drwxr-xr-x  2 root root 4096 Feb 13 02:39 html


ls -al /var/www/html/
# 抜粋した出力は以下の通りで、webfileの読み取り権限がない
# -rw-r----- 1 www-data www-data  215 Feb 13 02:39 webfile

# apacheの設定確認
ls -al /etc/apache2/
# 抜粋した出力は以下の通りで.htpasswdが怪しい
# -rw-r--r--  1 root root    45 Feb 13 02:38 .htpasswd
```

接続認証に関連しそうなファイルを探す。
```cat /etc/apache2/sites-enabled/000-default.conf```を実行すると下記が出力され、/var/www/htmlにアクセするにはBasic認証で、認証情報は/etc/apache2/.htpasswdに書かれていることが分かった。
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined


        <Directory "/var/www/html">
                AuthType Basic
                AuthName "Protected Content"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>
</VirtualHost>
```

```bash
# Basic認証のハッシュファイルの表示
cat /etc/apache2/.htpasswd
# 出力結果は下記の通りで、carlosというユーザに対して、パスワードはハッシュ化されていることがわかる
# carlos:$apr1$b1kyfnHB$yRHwzbuKSMyW62QTnGYCb0
```

### Basic認証のユーザ名とパスワードの特定

```bash
cd /home/admin/

# パスワードクラッカーでのパスワード特定
john/run/john /etc/apache2/.htpasswd
# 出力結果の抜粋は下記の通りで、ユーザー名がcarlosで、パスワードがchaletと分かった
# Enabling duplicate candidate password suppressor
# chalet           (carlos)

# 別コマンドでもパスワード確認
htpasswd -bv /etc/apache2/.htpasswd carlos chalet
# 出力結果
# Password for user carlos correct.

# Basic認証でアクセスできるか確認
curl -u carlos:chalet localhost
# 出力結果は省略するが、認証成功

# ファイルの取得
curl -u carlos:chalet localhost/webfile --output webfile

# ファイルの中身を確認
cat webfile
# 出力結果は省略するが、バイナリファイルの可能性が高い
```

### 暗号化ファイルの復号化

```bash
cd /home/webfile

# ファイルの中身の特定
file webfile
# 出力結果は下記の通りで、zipファイルであることが分かった
# webfile: Zip archive data, at least v1.0 to extract

# zipファイルの解凍
unzip webfile
# 出力結果は下記の通りで、パスワードがかかっていて解凍できない
# Archive:  webfile
# [webfile] secret.txt password:

# zipファイルのパスワードハッシュ情報を取得
john/run/zip2john webfile > webfile.hash
cat webfile.hash
# 出力結果
# webfile/secret.txt:$pkzip$1*2*2*0*1d*11*aac6e9af*0*44*0*1d*14e0*7307b809c29d0e4602770428f8f469ba44ff98065855e557f76e29e8d1*$/pkzip$:secret.txt:webfile::webfile

# パスワードクラッカーでのパスワード特定
john/run/john webfile.hash
# 出力結果の抜粋は下記の通りで、パスワードがandesと分かった
# Enabling duplicate candidate password suppressor
# andes            (webfile/secret.txt)

# パスワードを入力して解凍
unzip webfile

# テストコマンドの実行
sha1sum /home/admin/secret.txt | awk '{print $1}'
```
