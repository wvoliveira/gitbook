---
description: >-
  No Ansible, uma role é a forma em que podemos quebrar coleções de tarefas em
  pequenas partes, para serem utilizadas em diferentes playbooks.
---

# 1. Criar roles reutilizáveis

## Meta

A ideia é criar algo simples para mostrar como podemos reutilizar roles em diferentes projetos. Pensando nisso, vamos criar duas roles:

1. Basic: para instalação e configurção de ferramentas básicas do Linux;
2. Nginx: instalação e configuração do Nginx, que poderá ser utilizado como proxy reverso de alguma aplicação.

As duas ideias fazem total sentido na maioria dos ambientes corporativos. A "basic" poderá servir para configuração de repositórios internos, certificados internos, instalação de ferramentas que existem na maioria dos repositórios, como telnet, netcat e etc. Por outro lado, o "Nginx" é utilizado em diversas arquiteturas, principalmente como proxy reverso de alguma solução.

E para facilitar a criação e a lógica das nossas roles, vamos restringir nosso foco em 1 sistema operacional específico: CentOS 7.

Ah, tenha em mente também que estou usando um servidor remoto para servir como cobaia para este artigo. Então ficará assim: criar as roles no meu notebook e executar as tarefas em um servidor remoto.

## Role Basic

Para iniciarmos uma nova role, vamos utilizar o ansible-galaxy:

```text
$ ansible-galaxy role init basic
```

Com isso temos a seguinte estrutura:

```text
.
└── basic
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

Explicando o significado de cada pasta:

* defaults: variáveis padrões;
* files: arquivos que serão distribuidos;
* handlers: funções que poderão ser invocadas sobre alguma condição;
* meta: informações da role;
* tasks: principais funções/tarefas que serão executadas;
* templates: modelos de arquivos que serão distribuídos;
* tests: tarefas para testar as funções da role;
* vars: outras variáveis da role que tem maior valor do que as variáveis definidas na pasta "defaults". Veja na [documentação](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#playbooks-variables) para mais detalhes e como devem ser utilizadas.

Agora vamos definir as funções para essa role:

1. Atualizar os pacotes que já estão instalados no sistema operacional;
2. Instalar o `telnet` e `netcat`;
3. Alterar o conteúdo do /etc/motd.

Copie e cole o seguinte conteúdo no arquivo main.yml da pasta tasks:

```text
---
- name: atualiza os pacotes instalados no sistema
  yum:
    state: latest
    update_only: yes

- name: instala pacotes adicionais
  yum:
    name: "{{ basic_packages }}"

- name: altera o arquivo /etc/motd
  template:
    src: motd.j2
    dest: /etc/motd
    mode: '0644'
```

As 3 tarefas acima fazem exatamente o que listamos no passo anterior, porém temos que definir as variáveis e o template. Mas antes vamos explicar nos detalhes:

* O módulo `yum` servirá para atualizar todos os pacotes instalados no sistema para a última versão;
* O módulo `yum` será utilizado para instalar os pacotes listados na variável `basic_packages`;
* Por último, utilizaremos o módulo `template` para inserir o conteúdo do nosso modelo `motd.j2` para o arquivo /etc/motd.

Esses passos podem ser simples, porém segue a mesma lógica de configuração de várias soluções: instalação do pacote &gt; configuração &gt; inicialização do serviço.

Vamos configurar o que resta, começando definindo as variáveis. No arquivo main.yml da pasta defaults, cole o seguinte conteúdo:

```text
---
basic_packages: 
  - telnet
  - nc

basic_motd: "Gerenciado pelo Ansible!"
```

Agora crie um arquivo motd.j2 dentro da pasta templates com o seguinte conteúdo:

```text
{{ basic_motd }}
```

O exemplo é bem simples e serve para ser bem claro mesmo, mas podemos reforçar: quando executarmos o ansible, ele irá usar o template motd.j2 utilizando a variável _basic\_motd_ definida no arquivo main.yml da pasta defaults.

Aparentemente está tudo OK. Bora criar um arquivo para invocar nossa role. Nesse exemplo criei um arquivo com o nome site.yml no mesmo path da pasta basic \(a role que acabamos de criar\),  conforme padrão da documentação. Copie e cole o seguinte conteúdo nesse arquivo:

```text
---
- hosts: all
  become: yes
  roles:
    - basic
```

Explicando as linhas:

* Iremos rodar em todos os hosts do inventário;
* Utilizar root para previlégios administrativos;
* Invocar a role que acabamos de criar.

Lembrando que esse arquivo site.yml irá ficar no mesmo diretório da pasta basic. Assim:

```text
.
├── basic
│   ├── README.md
│   ...
└── site.yml
```

Agora vamos usar a ferramenta de linha de comando ansible-playbook para invocar esse arquivo. Como eu irei rodar esse playbook em um servidor remoto, irei passar o nome do host por linha de comando, como se fosse nosso inventáro de máquinas:

```text
ansible-playbook -i "<3.235.109.63>," site.yml -u cloud_user -kK
```

Na linha acima irei conectar no servidor remoto \(3.235.109.63\) com um usuário sem privilégios administrativos \(cloud\_user\), por isso coloquei o become: yes no arquivo `site.yml` e os parâmetros `-kK` serve para o ansible perguntar a senha do usuário e a senha do usuário com privilégios.

Por fim, o seguinte resultado deverá aparecer em sua tela:

```text
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [all] ********************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [3.235.109.63]

TASK [basic : atualiza os pacotes instalados no sistema] **********************************************************************************************
ok: [3.235.109.63]

TASK [basic : instala pacotes adicionais] *************************************************************************************************************
changed: [3.235.109.63]

TASK [basic : altera o arquivo /etc/motd] *************************************************************************************************************
changed: [3.235.109.63]

PLAY RECAP ********************************************************************************************************************************************
3.235.109.63               : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Onde mostrar `changed`, significa que algo foi alterado naquela etapa. No nosso caso os pacotes foram instalados e o arquivo /etc/motd alterado.

Podemos verificar autenticando no servidor e validando se realmente foi realizado com sucesso ou podemos fazer isso com o próprio ansible:

```text
ansible -i "3.235.109.63," all -m yum -a "name=telnet" -u cloud_user -kK
```

Depois eu explico com mais detalhe em outro post, mas resumindo: essa é uma das formas em que podemos utilizar o Ansible por linha de comando \([ad-hok](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)\), porém deverá ser utilizado com cautela, pois não é nada reutilizável dessa forma.

O resultado desse comando:

```text
3.235.109.63 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
        "1:telnet-0.17-65.el7_8.x86_64 providing telnet is already installed"
    ]
}
```

Mostrando que o telnet já está instalado.

![](../../.gitbook/assets/giphy-magic.gif)

Depois eu continuo na criação e explicação da role Nginx🤤 

## Role Nginx

Para criarmos a role para instalação e configuração do Nginx seguiremos a mesma lógica da role basic. Utilizaremos o ansible-galaxy para iniciarmos uma role do zero:

```text
ansible-galaxy role init nginx
```

A estrutura da role continua a mesma:

```text
.
└── nginx
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

Como comentei anteriormente, vamos seguir a simplicidade nas tarefas para dar enfâse a reutilização das roles. Então vamos criar as seguintes ações:

1. Instalar o Nginx;
2. Adicionar um novo arquivo index.html;
3. Iniciar e habilitar o serviço do Nginx.

Tente analisar o conteúdo abaixo, depois copie e cole no arquivo main.yml da pasta tasks:

```text
---
- name: install nginx 
  yum:
    name: nginx
    state: present

- name: insert index.html
  template:
    src: index.html.j2
    dest: /usr/share/nginx/html/index.html
    owner: root
    group: root
    mode: '0777'
  notify: restart service

- name: start service
  systemd:
    name: nginx
    state: started
```

Existem 3 tarefas conforme listamos no passo anterior. A maioria segue a mesma lógica da role basic, porém existe 1 ponto bem interessante: o notify. Ao executar a tarefa de inserir o index.html, se tiver alterações a serem feitas, ele irá reiniciar o serviço, chamando o handler "restart service".

Vamos inserir o conteúdo do handler no arquivo main.yml na pasta handlers:

```text
---
- name: restart service
  systemd:
    name: nginx
    state: restarted
```

E não podemos nos esquecer do template. Arquivo index.html.j2 na pasta templates:

```text
<!DOCTYPE HTML>
<html>
<head>
  <title>Criado pelo Ansible</title>
</head>

<body>
	<h1>Criado pelo Ansible</h1>
</body>
</html>
```

Agora vamos adicionar essa role no site.yml que criamos para invocar a role basic:

```text
---
- hosts: all
  become: yes
  roles:
    - basic
    - nginx
```

E bora executar da mesma forma que executamos a role basic:

```text
ansible-playbook -i "18.223.170.162," site.yml -u cloud_user -kK
```

Se tudo der certo, o resultado da execução será esse:

```text
...
TASK [install nginx] *************************************************************************************************************
changed: [18.223.170.162]

TASK [nginx : insert index.html] *************************************************************************************************
changed: [18.223.170.162]

TASK [nginx : start service] *****************************************************************************************************
changed: [18.223.170.162]

RUNNING HANDLER [nginx : restart service] ****************************************************************************************
changed: [18.223.170.162]

PLAY RECAP ***********************************************************************************************************************
18.223.170.162             : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

E para verificar se nossa instalação e configuração do Nginx funcionou, bora acessar a porta 80 e ver o que retorna:

```text
curl 18.223.170.162
```

Resultado:

```text
<!DOCTYPE HTML>
<html>
<head>
  <title>Criado pelo Ansible</title>
</head>

<body>
	<h1>Criado pelo Ansible</h1>
</body>
</html>
```

![](../../.gitbook/assets/giphy-brain123.gif)

Isso parece básico mas é muuuuito maneiro! 

Por fim, realizamos as seguintes tarefas com as duas roles:

1. Atualizamos os pacotes instalados no sistema operacional;
2. Instalamos pacotes bem interessantes, como telnet e netcat;
3. Alteramos o /etc/motd para quando entrarem no servidor saberem que ele é gerenciado pelo Ansible;
4. Instalamos, configuramos e iniciamos o Nginx.

Isso tudo pode ser reutilizado por outros projetos. Digamos que você queria só utilizar a parte da role basic ou somente a parte do Nginx.. Tudo é apartado.

De novo: essas podem parecer tarefas básicas, porém são muito utilizadas em diversas roles em produção por aí.

Espero que você tenha gostado.



## Referências

* [https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_reuse\_roles.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
* [https://docs.ansible.com/ansible/latest/user\_guide/intro\_adhoc.html](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)
* [https://docs.ansible.com/ansible/latest/user\_guide/become.html](https://docs.ansible.com/ansible/latest/user_guide/become.html#using-become)
* [https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_intro.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)
* [https://docs.ansible.com/ansible/latest/modules/template\_module.html](https://docs.ansible.com/ansible/latest/modules/template_module.html)
* [https://docs.ansible.com/ansible/latest/modules/yum\_module.html](https://docs.ansible.com/ansible/latest/modules/yum_module.html)
* [https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_variables.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#playbooks-variables)
* [https://docs.ansible.com/ansible/latest/user\_guide/playbooks\_reuse\_roles.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)



