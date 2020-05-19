# Pequenas customizações 

- PHP - Versão e extensões Habilitadas
    - PHP 7.4 (Repositório original ainda utilizando versão 7.3)
    - php_curl
    - php_fileinfo
    - php_ldap
    - php_mbstring
    - php_mysqli
    - php_openssl
    - php_pdo_mysql
    - allow_url_include = On
- Mariadb
    - [Solucionado erro do container mariadb](#erro_container_mariadb)
- http:// localhost (apenas um capricho)
    - Modificado nginx/sites/default.conf para apontar nginx/public/ quando acessar http://localhost. Página exibe versão do PHP utilizado.

# Começando Laradock e iniciando os containers

- Baixe docker para o seu sistema: https://www.docker.com/ (no windows, use docker desktop)
- Clone o repositório: 

```
$ git clone https://github.com/gonribeiro/laradock.git
```

- Crie o ".env" a partir do "env-example"

- No terminal (CMD, PowerShell ou outros), acesse a pasta do Laradock e inicie os containers

```
$ docker-compose up -d nginx mariadb redis workspace php-fpm
```

Caso seja a primeira vez que o executa, o docker irá baixar as imagens de cada serviço. Isso poderá levar muito tempo - Vá fazer um lanche enquanto isso ;)

- OBS: No windows, caso apareça "deseja compartilhar volume?", clique em "sim" ou "compartilhar".
- Troubleshooting: [Consulte a solução caso tenha problemas para executar os containers com conta do ADDS](#erro_iniciar_container_volume_compartilhado)

# Criando um projeto Laravel no Laradock

- No terminal, ainda dentro da pasta do Laradock, execute o workspace

Pelo workspace você executa comandos artisan, composer, gulp, etc durante o seu desenvolvimento... Ou seja, aqui será seu local de desenvolvimento.

```
$ docker-compose exec workspace bash
```

Dentro do workspace, utilize o composer para criar seus projetos Laravel.

```
$ composer create-project laravel/laravel seu_projeto --prefer-dist 
```

O projeto é criado na mesma pasta onde encontra-se a pasta do laradock <mark>(não dentro da pasta do laradock)</mark>, Exemplo:

- Desktop\
    - Laradock\
    - seu_projeto1\
    - seu_projeto2\

# Acessando um ou vários projetos Laravel (ou outro web qualquer)

- Na pasta "laradock/nginx/sites/" crie "seu_projeto.conf" a partir do "laravel.conf.example" modificando o trecho "seu_projeto" para o nome e pasta do seu projeto

```
...

server_name seu_projeto.local;
root /var/www/seu_projeto/public;
...
```

- No terminal, dentro da pasta do Laradock, reincie os containers

```
$ docker-compose restart
```

- No windows, aponte os seus projetos no arquivo "hosts".

Em “C:/Windows/System32/drivers/etc/hosts”, insira: 

```
...
127.0.0.1 seu_projeto.local
127.0.0.1 seu_projeto2.local
```

- Acesse seu projeto pelo navegador: 

https://seu_projeto.local

- <mark>Repita os mesmos passos acima para cada novo projeto</mark>
- OBS 1: Não use “.dev” como extensão para seu ambiente de desenvolvimento ou produção, é uma recomendação do laradock por problema do Google Chrome.
- OBS 2: Para se conectar ao MariaDB, use o usuário e senha "root".
- <mark>Troubleshooting:</mark> [Consulte a solução caso o ocorra o problema "MySQL connection refused"](#erro_connection_refused_db)

# Troubleshooting

- <b>Visualizando logs</b>

Recebeu algum erro e deseja visualizar os logs? Use:

```
# docker-compose logs [options] [SERVICE…]

# Exemplo:
$ docker-compose logs nginx
```

Os logs estarão disponíveis na pasta: laradock\logs\

Fonte: https://docs.docker.com/compose/reference/logs/

<a id="erro_iniciar_container_volume_compartilhado"></a><hr>

- <b>Erro "compartilhamento de volume" ao iniciar os containers no Windows</b>

PROBLEMA

Você está usando Windows, conectado com uma conta de usuário de domínio ADDS ([Active Directory Domain Server](https://pt.wikipedia.org/wiki/Active_Directory)) e os conteiners não estão subindo informando erro "compartilhamento de volume". 

SOLUÇÃO

No menu iniciar, procure por: "Editar usuários e grupos locais" > Pasta "Grupos" > adicione o seu usuário do AD nos grupos “Administradores” e “Administradores do Hyper-V” e reinicie o computador.

Fonte: https://github.com/docker/for-win/issues/2946

<a id="erro_connection_refused_db"></a><hr>

- <b>Erro de conexão com o banco de dados "MySQL connection refused"</b>

PROBLEMA

Aplicação laravel informa erro ao tentar conectar o banco de dados, mensagem "MySQL connection refused" 

SOLUÇÃO

No seu projeto, execute em qualquer lugar que possa acessar facilmente:

```
dd(Request::ip())
```

Com o ip em mãos, informe-o no "DB_HOST" no .env do seu projeto laravel

Fonte: https://laradock.io/help/#i-get-mysql-connection-refused 

<hr>

- <b>Erro ao mudar a versão do PHP após ter iniciado os containers</b>

PROBLEMA

Na primeira vez que iniciei os containers, a versão PHP que estava como padrão era 7.3. Ao modificar para 7.4 e reiniciar os containers, não atualizou a versão do serviço. 

SOLUÇÃO

Verifiquei que ficaram duas imagens instaladas, uma de cada versão. Apaguei a versão que não desejava mantendo apenas a 7.4.

Caso problema persista, "recompile" o php-fpm:

```
$ docker-compose build php-fpm
```

<a id="erro_container_mariadb"></a><hr>

- <b>Erro ao iniciar o container do MariaDB ou MySQL</b>

PROBLEMA

Após algum tempo tive problemas para subir o container do MariaDB. 

SOLUÇÃO 

(já aplicada no MariaDB nesta customização) Modificar o trecho abaixo no docker-compose.yml (O problema não era com o Laradock mas com o próprio container do mysql/mariadb)

```
mssql:
    build:
        context: ./mssql
    environment:
        - MSSQL_PID=Express
        - MSSQL_DATABASE=${MSSQL_DATABASE}
        - SA_PASSWORD=${MSSQL_PASSWORD}
        - ACCEPT_EULA=Y
    volumes:
        - ${DATA_PATH_HOST}/mssql:/var/opt/mssql
    ports:
        - "${MSSQL_PORT}:1433"
    networks:
        - backend
```

Fonte: https://github.com/laradock/laradock/issues/916 

# Fontes

- Laradock
    - https://laradock.io/introduction/
    - https://blog.especializati.com.br/utilizando-o-laradock/ 
    - https://byteslivres.com.br/blog/utilizando-o-docker-para-seu-projetos-laravel-com-laradock/ 
- Nginx SSL
    - https://zeropointdevelopment.com/how-to-get-https-working-in-windows-10-localhost-dev-environment/