## Node.jsを少し勉強してみた

---

## 自己紹介
- 福島隆冶(ふくしまりゅうや)
- SD2部
- 今は自宅待機中
- 社内図書管理システム作ってます

---

## なぜNode.jsなのか
- ReactとかVueとか触っていると`node`が必ず出てくる
- なんとなく使っていて、バックエンドで動作するJavaScriptくらいのイメージ
- よくわからないなら調べた
    - ちなみに[本家サイト](https://nodejs.org/ja/about/)

---

## Node.jsについて

>>>

## 出来ること
- サーバーサイドでJavaScriptを実行できる
    - V8(JavaScriptエンジン)を使用しているから
- Webサーバーの構築ができる

>>>

## 強み
- フロント/バックを同じ言語で実装出来る
- リアルタイムWebに向いてる
    - socket.io
- スケーラブル
    - 同時接続数が増えても処理速度が落ちず、高速に動作する
- C10K問題を解決出来る
    - イベントループかつノンブロッキングI/Oだから
        - Nginxもイベントループ

>>>

調べればたくさん出てくる

>>>

とりあえず

何事も触ってみないと

>>>

ということで

Webサーバー構築してみた

---

## Webサーバーを構築してみる

>>>

公式のコードをそのまま実行してみる

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World');
});

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```

>>>

まずはサーバーを起動

![サーバー起動](./images/web-server/1.png)

>>>

**http://127.0.0.1:3000/** にアクセスすると

![web-server-1](./images/web-server/2.png)

>>>

「Hello world」が表示された

すごい簡単

☺️

>>>

ルーティングもしてみる

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {

    const url = req.url;

    let resMessage = '';

    switch (url) {
        case '/hoge':
            res.statusCode = 200;
            resMessage = 'Hoge page.';
            break;
        case '/fuga':
            res.statusCode = 200;
            resMessage = 'Fuga page.';
            break;
        default:
            res.statusCode = 404;
            resMessage = 'Default page.';
    }
    res.setHeader('Content-Type', 'text/plain');
    res.end(resMessage);
});

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});

```

>>>

**http://127.0.0.1:3000/hoge** にアクセスすると

![](./images/web-server/3.png)

>>>

**http://127.0.0.1:3000/fuga** にアクセスすると

![](./images/web-server/4.png)

>>>

**http://127.0.0.1:3000/** にアクセスすると

![](./images/web-server/5.png)

>>>

- `/hoge`
- `/fuga`
- `/`
- それぞれ、ルーティングが制御出来た

>>>

## その他
- 他にも外部API叩いてみたり、RestfullなAPIサーバー構築してみたり
    - expressとかFWあるから使うと簡単に構築できる
- やりたかったけど、時間掛かるからまた今度


---

## もうちょっとNodeを触ってみる

>>>

そうだ、チャットアプリ作ってみよう

>>>

## 実行画面

![sample-1](./images/chat-sample/sample-1.gif)

[参考にしたハンズオン記事](https://qiita.com/riku-shiru/items/ffba3448f3aff152b6c1)

>>>

## 実装したソース

app.js
```javascript
var express = require('express');
var app = express();
var http = require('http').Server(app);
const io = require('socket.io')(http);
const PORT = process.env.PORT || 7000;

app.get('/' , function(req, res){
    res.sendFile(__dirname+'/index.html');
});

io.on('connection',function(socket){
    socket.on('message',function(msg){
        console.log('message: ' + msg);
        io.emit('message', msg);
    });
});

http.listen(PORT, function(){
    console.log('server listening. Port:' + PORT);
});
```

>>>

index.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>socket.io chat</title>
    <script src="/socket.io/socket.io.js"></script>
    <script src="https://code.jquery.com/jquery-1.11.1.js"></script>
</head>
<body>
<ul id="messages"></ul>
<form id="message_form" action="#">
    <input id="input_msg" autocomplete="off" /><button>Send</button>
</form>
<script>
    var socketio = io();
    $(function(){
        $('#message_form').submit(function(){
            socketio.emit('message', $('#input_msg').val());
            $('#input_msg').val('');
            return false;
        });
        socketio.on('message',function(msg){
            $('#messages').append($('<li>').text(msg));
        });
    });
</script>
</body>
</html>
```

>>>

- クライアント→サーバーで値を送信
- サーバーでイベントが発火
    - サーバー→クライアントでそのまま値がブロードキャストされる
- クライアントでイベントが発火
    - 受け取ったメッセージを表示

>>>

## もうちょっと作り込んでみた
- メッセージをfirestoreに登録
- ちょっとだけ画面を装飾
- 名前ぐらい付けてあげる

>>>

実行画面

![sample-2](./images/chat-sample/sample-2.gif)

>>>

良い感じ！

>>>

### せっかくだからデプロイしてみた

>>>

firebase使ってるし

そのままホスティングでデプロイしちゃおう！

>>>

- index.jsはfunctionsフォルダに
- index.htmlはpublicフォルダに格納したのはいいが

>>>

- index.htmlからnode_modulesのsocket.ioのライブラリが読み込めない！
- index.jsも実行されてない感じがする

>>>

めんどくさい

>>>

firebaseでデプロイちょっと厳しかったからHeroku使ってみよう！

>>>

Heroku CLIを使って適当にデプロイ！

>>>

出来た！

[アクセス](https://guarded-temple-43209.herokuapp.com/)はこちらから

![sample-3](./images/chat-sample/sample-3.png)

>>>

今回はこれで終了

---

### ご静聴ありがとうございました！
