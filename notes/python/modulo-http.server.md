# Módulo http.server

## Espero que use no dia a dia

Esse módulo pode ser muito útil quando você precisa compartilhar algum arquivo de forma rápida e fácil. Quando invocamos o módulo `http.server`, ele utiliza o diretório atual para expor a lista de arquivos numa porta específica:

```text
$ python -m http.server

Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

Por padrão a porta é a 8000. Quando acessamos essa porta no browser ou pelo `curl`, ele mostra a lista de arquivos do diretório atual:

```text
$ curl http://localhost:8000/

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="arquivo1.txt">arquivo1.txt</a></li>
<li><a href="arquivo2.txt">arquivo2.txt</a></li>
</ul>
<hr>
</body>
</html>
```

Com isso, podemos listar o conteúdo de algum diretório e pegar os arquivos que precisamos, sem precisar de autenticação. Exemplo pelo `curl`:

```text
$ curl -sO http://localhost:8000/arquivo1.txt

$ ls -lh arquivo1.txt
-rw-r--r--  1 wvoliveira  wvoliveira     123B Aug 18 14:19 arquivo1.txt
```

Claro, nesse caso o endereço do servidor seria outro.

No browser seria a mesma coisa, o que mudaria é a forma de salvar o conteúdo: botão direito do mouse no arquivo &gt; Save Link As...

Temos vários outros apps que podemos utilizar para efetuar essa cópia de conteúdo de um lugar para o outro, como rsync, scp, cp e etc, mas com certeza você usará o módulo `http.server` algum dia.

