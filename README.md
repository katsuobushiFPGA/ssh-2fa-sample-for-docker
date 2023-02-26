# このリポジトリについて
Dockerを利用してSSHの2FAを試すリポジトリです。  
動作がどのようなものかを確認するために作成したため、本番運用には適しておりません。  

# 準備
```
cp .env.example .env
```
`.env` に下記を設定する。
- USER_NAME: コンテナ内にできるユーザ名
- USER_PASS: コンテナ内にできるユーザのパスワード
- SSH_PORT:  コンテナにアクセスできるポート
例:
```
USER_NAME=kbushi
USER_PASS=kbushi
SSH_PORT=2222
```

# 動作手順
1. このリポジトリをcloneします。
```
git clone https://github.com/katsuobushiFPGA/ssh-2fa-sample-for-docker.git
```
2. イメージをビルドし、立ち上げます。
```
docker compose up -d
```
3. コンテナ内に.sshファイルが生成されているので、ホストファイルにコピーします。
```
docker cp [コンテナID]:/home/[ユーザ名]/.ssh [コピー先]

例: docker cp 2d0ad2b920df:/home/kbushi/.ssh volumes/
(直下にある volumesフォルダにコピーされます。)
```
4. sshでのアクセスを試みます。
```
ssh -i [SSH格納フォルダ]/.ssh/id_rsa [ユーザ名]@localhost -p [ポート]

例: ssh -i volumes/.ssh/id_rsa kbushi@localhost -p 2222
※ USER_NAME=kbushi, SSH_PORT=2222で設定した場合
```
```
$ ssh -i volumes/.ssh/id_rsa kbushi@localhost -p 2222
Password:

USER_PASSを入力後にログインできればOK
```

5. コンテナ内に入り、`google-authenticator` をセットアップする。
```
docker-compose exec --user=[ユーザ名] gateway bash

[kbushi@2d0ad2b920df ~]$ google-authenticator

Do you want authentication tokens to be time-based (y/n) y

QRとシークレットキーが出てくるので、それらを Authenticatorアプリに入れる。
Enter code from app (-1 to skip): XXXXXX
(アプリに出てくるコードを入力する。)

Do you want to do so? (y/n) n

Do you want to enable rate-limiting? (y/n) y
```

6. 再度SSHでログインをしてみる。
パスワードと認証コードが聞かれるようになる。
```
ssh -i [SSH格納フォルダ]/.ssh/id_rsa [ユーザ名]@localhost -p [ポート]

例: ssh -i volumes/.ssh/id_rsa kbushi@localhost -p 2222
※ USER_NAME=kbushi, SSH_PORT=2222で設定した場合
```
例:
```
$ ssh -i volumes/.ssh/id_rsa kbushi@localhost -p 2222
Password:
Verification code: 182171
Last login: Sun Feb 26 07:14:09 2023
```