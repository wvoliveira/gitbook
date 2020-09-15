# 2. Ambiente virtual

## Essencial para qualquer projeto em Python

Ambiente virtual  é bem útil quando trabalhamos com aplicações, scripts ou até mesmo testar alguma solução. Ele funciona como um ambiente fechado onde você pode alterar, destruir sem comprometer o sistema.

O ambiente virtual cria uma cópia de todos os binários necessários para utilizar a linguagem. Com isso, temos os seguintes benefícios:

* Instalar dependências sem comprometer o sistema;
* Testar com versões diferentes da linguagem. Ex.: python 2.7, python 3.5, python 3.8;
* Instalar dependências que só a minha aplicação necessita. Ex.: requests, boto;
* Instalar dependências com versões específicas. Ex.: django 3.1, flask 1.1.2;

Existem alguns gerenciadores de ambientes virtuais pelo mundo a fora que fazem várias funções, porém o python já vem com um bem simples de usar, o `venv`. 

A sintaxe é bem fácil, basta chamar o módulo `venv` seguido do nome do nosso diretório que armazenará todas as dependências para a linguagem funcionar:

```text
$ python -m venv venv
```

Nesse caso iremos criar um diretório com o mesmo nome do módulo. Agora temos que "ativar" esse ambiente:

```text
$ source venv/bin/activate
(venv) $
```

Após a ativação desse ambiente virtual, você notará que apareceu o nome do ambiente no nosso console \(venv\). Isso significa que o ambiente está ativo e pronto para ser utilizado. Caso esteja com dúvidas se realmente estamos num ambiente virtual, podemos executar um comando para verificar de qual caminho ele está executando o python:

```text
(venv) $ python
>>> import sys
>>> sys.path
['', '/usr/lib/python38.zip', '/usr/lib/python3.8', '/usr/lib/python3.8/lib-dynload', '~/venv/lib/python3.8/site-packages']
```

Na última linha podemos ver que o executável do python é `~/venv/lib/python3.8/site-packages`.

Com isso já podemos instalar nossas dependências com o pip:

```text
$ pip install flask==1.1.2 requests==2.24.0
```

Para verificar os módulos instalados no nosso ambiente virtual:

```text
$ pip list

Package      Version
------------ ---------
certifi      2020.6.20
chardet      3.0.4
click        7.1.2
Flask        1.1.2
idna         2.10
itsdangerous 1.1.0
Jinja2       2.11.2
MarkupSafe   1.1.1
pip          20.1.1
requests     2.24.0
setuptools   47.1.0
urllib3      1.25.10
Werkzeug     1.0.1
```

A maioria dos pacotes acima são dependências das bibliotecas que instalamos no passo anterior.

Para sair do ambiente virtual, basta rodar o comando abaixo:

```text
$ deactivate
```

Pronto, você já sabe utilizar ambientes virtuais.

Caso queira testar outros gerenciadores de ambientes virtuais, segue:

* [https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)
* [https://github.com/pypa/virtualenv](https://github.com/pypa/virtualenv)
* [https://github.com/theacodes/nox](https://github.com/theacodes/nox)

Todos tem seus prós e contras, basta você decidir se o que você está fazendo necessita da complexidade ou simplicidade das soluções.

Referências:

* [https://docs.python.org/3/tutorial/venv.html](https://docs.python.org/3/tutorial/venv.html)
* [https://github.com/vinta/awesome-python\#environment-management](https://github.com/vinta/awesome-python#environment-management)
* [https://docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html)

