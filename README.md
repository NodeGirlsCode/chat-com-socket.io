![logo do node girls](https://i.imgur.com/4QATtIp.png)

# Chat com Node.js e Socket.io

Vamos explicar direitinho como funciona a criação e hospedagem de um chat em Node.js com Socket.io, baseado na documentação básica online Socket.io.

## Instalação

E aí, mãos na massa?

### Pré-requisitos

Você precisa ter o Node.js instalado em sua máquina e o Git também :) Qualquer um dos dois está disponível facilmente na internet. Aqui (https://nodejs.org/en) seria o Node e no meu caso que uso Windows, está aqui o Git Bash: https://gitforwindows.org! Se você tem Linux, corre aqui: https://git-scm.com/download/linux :D

### Começou!

* Criamos uma pasta para o projeto e dentro um arquivo ```package.json``` com as seguintes informações:

```
{
  "name": "chat-com-socket-io",
  "version": "0.0.1",
  "description": "chat com socket.io e node",
  "dependencies": {}
}
```

* Agora vamos instalar o Express, que é um framework para Node que oferece uma série de recursos bacanas que servem de base para uma aplicação web.

``` npm install --save express@4.15.2 ```

* Crie um arquivo ```index.js``` e lance o seguinte código:

``` 
var app = require('express')();
var http = require('http').Server(app);

app.get('/', function(req, res){
  res.send('<h1>Olá mundo</h1>');
});

http.listen(3000, function(){
  console.log('ouvindo na porta *:3000');
});
```

Como você pode ver acima, ```app``` é uma variável que chama o Express e com base nesse app, construímos nosso web server com o protocolo HTTP. Depois, criamos uma rota para a página inicial, onde sua resposta é um Olá mundo :)

Para saber se a aplicação está funcionando ou não, colocamos no console.log um "ouvindo na porta" para que ele apareça toda vez que subirmos a aplicação. Para não precisar toda hora subir uma aplicação, podemos usar o ```nodemon```, mas eu já falo dele daqui a pouco.

Você viu que criou um ```package-lock.json``` no seu projeto? Não se preocupe, tá tudo bem!

* Clique com o botão direito em uma parte vazia da pasta do projeto e clique posteriormente em Git Bash. Dentro dele, digite: 

```node index.js```

Se deu tudo certo, vai aparecer o "ouvindo na porta 3000". Acessamos então localhost:3000 e é para estar abrindo sua página com um Olá mundo. Se quiser derrubar a aplicação, dê um CTRL+C no Git Bash.

* Agora vamos radicalizar. Primeiro, para não ficar toda hora nessa função de restartar aplicação, instalamos o nodemon:

```npm install -g nodemon```

Depois de instalado, voltamos pro Socket.io. Vamos trocar aquele código lá do Olá mundo por isso aqui:

```app.get('/', function(req, res){
res.sendFile(__dirname + '/index.html');
});
```

* Agora ao invés de aparecer um olá mundo, ele vai responder para um arquivo index.html. Mas e aí, não temos ele! Então vamos criar :)

```
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```

* Por enquanto não tem nada de Socket.io aqui, né? Mas calma! Agora é a hora da verdade:

```
npm install --save socket.io
```

* Troque todo o código da sua ```index.js``` por isso:

```
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function(socket){
  console.log('usuário entrou');
});

http.listen(3000, function(){
  console.log('ouvindo na porta *:3000');
});
```

Tá, mas o que estamos fazendo afinal? O Socket.IO se divide em duas partes: a primeira integra com um ou vários servidores HTTP em Node.js (socket.io). E a outra parte é a "biblioteca do cliente", os códigos, que carregam no browser (socket.io-client).

Agora iniciamos uma nova instância do socket.io, passando o servidor HTTP. Esse servidor lerá os eventos de conexão vindos e avisará através de um console log :)

* Agora, antes do  ```</body>``` do html, vamos adicionar esse código, chamando o Socket.io. Agora, toda vez que alguém entrar na index.html, seremos avisados no console da aplicação :)

```
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io();
</script>
```

* Para ver essa mágica acontecer, dê um ```nodemon index.js``` no Git Bash. Agora, não precisamos mais reiniciar a aplicação toda vez que acontecer uma alteração no seu código.

Subiu e está bombando, né? Agora podemos também avisar quando um usuário se desconecta: 

```
io.on('connection', function(socket){
  console.log('usuário entrou');
  socket.on('disconnect', function(){
    console.log('usuário saiu');
  });
});
```

E aí, toda vez que rolar um F5 na página ou alguém sair, vai aparecer entradas e saídas de usuários.

* Vamos começar a emitir eventos pra valer pelo Socket.io? Então modifique o código antes do seu ```</body>```:

```
<script src="/socket.io/socket.io.js"></script>
<script src="https://code.jquery.com/jquery-1.11.1.js"></script>
<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
  });
</script>
```
* Na ```index.js```, adicionamos mais um código: 

```
  io.on('connection', function(socket){
    socket.on('chat message', function(msg){
      console.log('mensagem: ' + msg);
    });
  });
```

* Mas se eu der refresh na página e tento mandar mensagens, ainda não aparece nada. O que tá faltando? Só aparece no console!

Vamos modificar só um pouco o código acima:

```
  io.on('connection', function(socket){
    socket.on('chat message', function(msg){
      console.log('mensagem: ' + msg);
      io.emit('chat message', msg);
    });
  });
```
E o script do ```</body>```:

```
<script>
  $(function () {
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
    socket.on('chat message', function(msg){
      $('#messages').append($('<li>').text(msg));
    });
  });
</script>
```

* Agora sim! Se duas ou mais pessoas estiverem online na mesma hora nesse mesmo "link" (claro, quando subirmos), elas podem conversar. Claro que esse chat pode ser melhorado e podemos colocar nome das pessoas, senha, questões de segurança mais avançadas, "usuário está digitando", mas deixaremos isso para outra hora, ok?

Para subir para algum lugar este código, vocês podem utilizar a Umbler: https://help.umbler.com/hc/pt-br/articles/115001793863-Node-JS-na-Umbler

E de lá, é só sucesso :)
