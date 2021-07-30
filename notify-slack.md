## notify_slackを利用して、alpの集計結果をSlackで通知する

### notify_slackのインストール
```
wget https://github.com/catatsuy/notify_slack/releases/download/v0.4.8/notify_slack-linux-amd64.tar.gz
tar zxvf notify_slack-linux-amd64.tar.gz
./notify_slack -version
```

### Slackアプリの作成（作成済)
- URLどこに書いてあんねん問題
slack->Activate Incoming WebhooksをONにすると出現

### notify_slackの設定ファイルの作成
```
vi /etc/notify_slack.toml

[slack]
url = "https://hooks.slack.com/services/**"
token = "xoxp-xxxxx"
channel = "#hoge"
username = "tester"
icon_emoji = ":rocket:"
interval = "1s"
```
記載場所が見つけづらいよ
- url
slack->Features->Incoming Webhooks->Webhook URLs for Your Workspace->Webhook URL
- token
slack->Features->OAuth & Permissions->OAuth Tokens for Your Workspace

### 動作確認
```
echo hello | ./notify_slack
```

### alpインストール〜集計まで済ませる

### 集計したログのhtmlファイルを作成
```
mkdir -p /www/data/logs
echo "<html><body><pre style=\"font-family: 'Courier New', Consolas\">" > /www/data/logs/alp.html
cat /var/log/nginx/access.log | ./alp ltsv -m "/upload/[0-9a-z]+.jpg,/items/[0-9]+.json,/new_items/[0-9]+.json,/users/[0-9]+.json,/transactions/[0-9]+.png,/static/*" >> /www/data/logs/alp.html
```
※ /www/data/logs/alp.htmlに書き込めない場合は権限を変更する必要があるかも

### 静的ファイルの配信設定
作成したhtmlファイルをローカルのブラウザから確認できるようにするためnginxの設定を変更する。
```
vim /etc/nginx/conf
```

```
// 中略
// 以下のように/logs/xxxx.htmlでアクセスできるようにしておく
    server {
        listen 8080;

        location / {
            proxy_pass http://localhost:8000;
        }

        location /logs/ {
            root /www/data;
            index alp.html;
        }
    }
```

設定が完了したらnginxの設定ファイルを再読み込みする
```
nginx -s reload
```

### 動作確認
ローカルのブラウザからhttp://<nginxが起動しているIPアドレス>:8080/logs/alp.htmlにアクセスする。
集計結果が表示されている。はず。

### htmlファイルをSlackに通知
`echo "http://172.28.128.3:8080/logs/alp.html" | ./notify_slack`





