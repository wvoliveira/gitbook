# 2. Consumir API com Golang

Estamos rodeados por APIs que nos abrem lacks de oportunidades para automatizar nossos trabalhos manuais do dia a dia. E agora vou te mostrar como podemos consumir uma REST API com a linguagem Golang.

Para você conseguir realizar as instruções contidas nesse post, irei utilizar uma API gratuita da internet: [https://reqres.in/](https://reqres.in/)

No [reqres](https://reqres.in/) temos tudo que uma REST API poderá ter: estrutura em JSON, paginação, autenticação e outros.

Bora criar uma pasta com o nome _reqres_ e inicializar nosso projeto:

```bash
$ mkdir reqres
$ go mod init reqres
```

No Golang, precisamos criar a estrutura da resposta da API. Então vamos escolher um endoint: [https://reqres.in/api/users](https://reqres.in/api/users)

Exemplo de retorno desse endpoint:

```javascript
{
  "page": 1,
  "per_page": 1,
  "total": 12,
  "total_pages": 12,
  "data": [
    {
      "id": 1,
      "email": "george.bluth@reqres.in",
      "first_name": "George",
      "last_name": "Bluth",
      "avatar": "https://s3.amazonaws.com/uifaces/faces/twitter/calebogden/128.jpg"
    }
  ],
  "ad": {
    "company": "StatusCode Weekly",
    "url": "http://statuscode.org/",
    "text": "A weekly newsletter focusing on software development, infrastructure, the server, performance, and the stack end of things."
  }
}
```

Agora que sabemos a estrutura da API, bora criar a estrutura no Golang. Crie um arquivo main.go na pasta reqres com o seguinte conteúdo:

```go
package main

type api struct {
	Page int `json:"page"`
	PerPage int `json:"per_page"`
	Total int `json:"total"`
	TotalPages int `json:"total_pages"`
	Data []user `json:"data"`
	Ad ad `json:"ad"`
}

type user struct {
	ID int `json:"id"`
	Email string `json:"email"`
	FirstName string `json:"first_name"`
	LastName string `json:"last_name"`
	Avatar string `json:"avatar"`
}

type ad struct {
	Company string `json:"company"`
	URL string `json:"url"`
	Text string `json:"text"`
}
```

Veja que parece bem complexo para manipular o retorno da API, mas temos que fazer isso para o Golang entender o resultado e assim conseguir manipular os dados.

Separei a estrutura do retorno da API em 3 partes:

1. O corpo da API com as chaves page, per\_page, total, total\_page, data e ad;
2. A estrutura do usuário: id, email, first\_name, last\_name e avatar;
3. A estrutura da chave "ad": company, URL e text.

As notações _json_ servem para converter de forma automática os dados com a biblioteca json do próprio Golang.

Vamos adicionar a chamada GET usando a biblioteca http:

```go
resp, err := http.Get("https://reqres.in/api/users")
if err != nil {
	log.Fatal(err)
}
```

No Golang verá que temos que repetir várias coisas, mas não ligue muito pra isso, o resultado é surpreendente.

No código acima simplesmente usamos a biblioteca [http](https://golang.org/pkg/net/http/) com o método Get passando a URL que queremos enviar a requisição. Com isso temos a resposta, mas ainda temos que tratar o conteúdo do corpo da resposta \(body\):

```go
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(body)
	}
```

Utilizei a biblioteca [ioutil](https://golang.org/pkg/io/ioutil/) para ler o conteúdo do body, que por padrão é do tipo [io.Reader](https://golang.org/pkg/io/#Reader).

Agora vamos criar uma variável atribuindo a estrutura da API que criamos no começo:

```go
content := api{}
```

E lembra da conversão automática que comentei?! Agora é a hora. Bora utilizar a biblioteca [json](https://golang.org/pkg/encoding/json/#Unmarshal) para converter o corpo da resposta para dentro da variável content:

```go
	err = json.Unmarshal(body, &content)
	if err != nil {
		log.Fatal(err)
	}
```

Veja que usamos & \(ponteiro\) para passarmos a referência da variável e não duplicar a variável dentro da função, ou seja, estamos passando o endereço da variável em memória. Se não entendeu, não tem problema, há outras pessoas que [explicam ](https://golangbot.com/pointers/)melhor.

Agora bora printar na tela o conteúdo da resposta da API utilizando a nossa estrutura atribuída a variável _content_:

```go
	fmt.Printf("Page: %v\n", content.Page)
	fmt.Printf("Per page: %v\n", content.PerPage)
	fmt.Printf("Total: %v\n", content.Total)
	fmt.Printf("Total pages: %v\n", content.TotalPages)
```

Porque não criar um loop e exibir todos os e-mails dos usuários:

```go
	for _, c := range content.Data {
		fmt.Printf("E-mail: %v\n", c.Email)
	}
```

Juntando todo o conteúdo que criamos até agora e jogando dentro da função main:

```go
func main() {
	resp, err := http.Get("https://reqres.in/api/users")
	if err != nil {
		log.Fatal(err)
	}

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(body)
	}

	content := api{}

	err = json.Unmarshal(body, &content)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Page: %v\n", content.Page)
	fmt.Printf("Per page: %v\n", content.PerPage)
	fmt.Printf("Total: %v\n", content.Total)
	fmt.Printf("Total pages: %v\n", content.TotalPages)

	for _, c := range content.Data {
		fmt.Printf("E-mail: %v\n", c.Email)
	}
}
```

Podemos rodar com o seguinte comando:

```go
$ go run .
```

Resultado:

```go
Page: 1
Per page: 6
Total: 12
Total pages: 2
E-mail: george.bluth@reqres.in
E-mail: janet.weaver@reqres.in
E-mail: emma.wong@reqres.in
E-mail: eve.holt@reqres.in
E-mail: charles.morris@reqres.in
E-mail: tracey.ramos@reqres.in
```

![](../../.gitbook/assets/giphy-brain312312321.gif)

Sem contar que você poderá criar o binário e rodar de onde você quiser. Seja Linux, MacOS ou Windows, sem nenhuma dependência do sistema operacional. Leia mais [aqui](https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04).

## Referencias

* [https://golang.org/pkg/net/http/](https://golang.org/pkg/net/http/)
* [https://golang.org/pkg/encoding/json/\#Unmarshal](https://golang.org/pkg/encoding/json/#Unmarshal)
* [https://golang.org/pkg/io/ioutil/](https://golang.org/pkg/io/ioutil/)
* [https://golang.org/pkg/io/\#Reader](https://golang.org/pkg/io/#Reader)
* [https://tour.golang.org/moretypes/1](https://tour.golang.org/moretypes/1)
* [https://reqres.in/](https://reqres.in/)

