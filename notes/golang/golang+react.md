---
description: Altamente escalável + interfaces magníficas
---

# Golang + React

Há um tempo atrás eu descobri o quão fácil é unir esses dois elementos sensacionais: [Golang](https://golang.org/) + [React](https://reactjs.org/). Então nesse post irei mostrar como você pode fazer em sua casa, no conforto do seu lar, tomando aquela coca-cola ou aquela colorado bem gelada ou qualquer coisa, água talvez?

Eu sempre tive curiosidade de como as empresas conseguem criar interfaces bonitas, funcionais, sensacionais etc e como escalam de forma rápida essas aplicações.. Eis então que descobri uma das formas, a rapidez do Golang e a facilidade de reutilizar váários componentes prontos em seu front-end.

Eu sei, existem vários frameworks, bibliotecas que podem ser utilizadas, mas como não sou especialista em nenhuma delas - e realmente meu conhecimento é bem razo - eu vou mostrar as tecnologias que tenho interesse. Mas tenha em mente que você consegue utilizar qualquer framework com o Golang, então se você conhece Angular, por que não unir Golang com Angular? Se liga [aqui](https://auth0.com/blog/developing-golang-and-angular-apps-part-1-backend-api/) e [aqui](https://medium.com/@anshap1719/getting-started-with-angular-and-go-setting-up-a-boilerplate-project-8c273b81aa6).

Voltando ao Golang + React, acompanha comigo:

Vamos criar uma pasta para o nosso projeto, nesse caso dei o nome de gore \(Go de Golang e Re de React. É, nada original\):

```bash
$ mkdir gore
$ cd gore
```

Agora vamos iniciar nosso módulo para controlar nossas dependências:

```bash
$ go mod init gore
```

E bora usar o [creat-react-app](https://create-react-app.dev/) para iniciar nosso projeto em React:

```bash
$ npx create-react-app web
$ cd web
```

Simm, o front-end ficará numa pasta chamada `web`. E caso você não tenha o npx instalado, atualize o seu [npm](https://www.npmjs.com/get-npm), pois a partir do 5.2 esse [CLI](https://en.wikipedia.org/wiki/Command-line_interface) já vem instalado por padrão.

Vamos iniciar o projeto para ver se está tudo OK com o app em React:

```bash
$ npm start
```

Caso o seu browser não abriu de forma automática, tente acessar o endereço [http://localhost:3000/](http://localhost:3000/). Caso o símbolo do React esteja rodando nesse link, é que o app está funcionando.

Agora vamos jogar esse projeto que está em React para dentro do Golang. Transformar todos os arquivos js, css, html em variáveis acessíveis dentro do Golang. Afinal, tudo é bytes, certo?

Podemos fazer isso com o alguns módulos: [go-bindata](https://github.com/containous/go-bindata), [pkger](https://github.com/markbates/pkger), [packr](https://github.com/gobuffalo/packr), [statik](https://github.com/rakyll/statik) e vários outros. Nesse caso iremos utilizar o [pkger](https://github.com/markbates/pkger). Bora instalar ele:

```bash
$ cd ..
$ go get github.com/markbates/pkger/cmd/pkger
```

E agora crie um arquivo `main.go` para ser nosso arquivo principal do nosso projeto em Golang e insira o seguinte conteúdo:

```go
package main

import (
	"io"
	"log"
	"net/http"

	"github.com/gorilla/mux"
	"github.com/markbates/pkger"
)

func main() {
	r := mux.NewRouter()

	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)

		f, err := pkger.Open("/public/index.html")
		if err != nil {
			w.Write([]byte(err.Error()))
		}
		io.Copy(w, f)
		defer f.Close()
		return
	})

	dir := http.FileServer(pkger.Dir("/public"))
	r.PathPrefix("/public").Handler(http.StripPrefix("/public", dir))

	log.Fatal(http.ListenAndServe(":3000", r))
}

```

Resumindo: iremos exibir o conteúdo do nosso arquivo principal do app em React \(index.html\) em um endpoint no Golang \("/"\).

Não execute esse código!Ainda temos que buildar o nosso projeto em React:

```text
$ cd web
$ PUBLIC_URL="/public" npm run-script build
```

Essa variável `PUBLIC_URL` serve para modificarmos o local onde os componentes do app irão procurar as dependências. Nesse caso, o Golang irá expor na rota /public.

Após o build terminar, uma pasta com o nome `build` irá aparecer. Vamos mover ela para o mesmo diretório do arquivo `main.go` e renomear para `public`:

```text
$ mv build ../public
$ cd ..
```

Então nosso diretório ficou da seguinte forma:

```text
├── main.go
├── public
└── web
```

E bora executar:

```text
$ go run main.go
```

Veja que ao executar pela primeira vez, ele irá baixar todas as dependências necessárias:

```text
go: finding module for package github.com/markbates/pkger
go: found github.com/markbates/pkger in github.com/markbates/pkger v0.17.0
go: finding module for package github.com/gorilla/mux
go: found github.com/gorilla/mux in github.com/gorilla/mux v1.8.0
```

 Por isso não instalamos o pkger e nem o mux antes. Mas é uma boa instalar as dependências antes, principalmente se você estiver utilizando o VSCode ou outro editor que tenha suporte ao auto-complete.

Ao terminar de instalar as dependências, tente acessar o link: [http://localhost:3000/](http://localhost:3000/). Se aparecer a mesma tela quando iniciamos nosso projeto pelo npm, é que deu tudo certo.

Agora bora criar o binário:

```text
$ go build .
```

Irá aparecer um arquivo executável com o nome do nosso projeto `gore`, com apenas 8M. Isso porque utilizamos o [mux](https://github.com/gorilla/mux) para facilitar nossa vida, se tivéssemos utilizado o [http](https://golang.org/pkg/net/http/) default do Golang ficaria menor ainda. Já imaginou as possibilidades? Podemos utilizar tecnologias modernas no front-end e unir com a rapidez, simplicidade e elegância do Golang.

Enfim, não é só por isso. Eu sou péssimo em argumentar sobre tudo, mas talvez você acredite em mim nos próximos posts. 

O conteúdo desse projeto pode ser encontrado aqui: [https://github.com/wvoliveira/gore](https://github.com/wvoliveira/gore).

Até a próxima.



