Resumidamente, o laradock é um ambiente de desenvolvimento PHP completo para o docker, contendo imagens docker pré-configuradas. Inicialmente o projeto foi focado em rodar os projetos Laravel , mas veio evoluindo e agora começou a suportar outros projetos PHP como Symfony, CodeIgniter, WordPress, Drupal e outros.

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
- Mariadb - [Solucionado erro do container mariadb](#erro_container_mariadb)
- Nginx e Apache usam https

# Começando Laradock e iniciando os containers

- Baixe docker para o seu sistema: https://www.docker.com/ (no windows, use docker desktop)
- Clone o repositório: 

```
$ git clone https://github.com/gonribeiro/laradock.git
```

- Na pasta clonada localize o arquivo "env-example". Faça uma cópia dele modificando seu nome para ".env".

- No terminal (CMD, PowerShell ou outros), acesse a pasta do Laradock e inicie os containers 

- OBS: Aqui você pode subir apenas os containers que precisa. Ex.: Inicie nginx + workspace + php-fpm para ter um servidor php e um ambiente de trabalho. Inicie mariadb e redis apenas se precisar deles. [Consulte a documentação do laradock para saber de outros serviços prontos para serem utilizados](https://laradock.io/documentation/). 

- OBS 2: Caso seja a primeira vez que o executa, o docker irá baixar e configurar as imagens de cada serviço. Isso poderá levar muito tempo - O grande responsável por esta demora é o "workspace". Ele será o seu ambiente de desenvolvimento. Nele você executa comandos artisan, composer, gulp, node, yarn etc (assim você não precisará ter esses programas instalados no seu PC, pode usar pelo workspace). Isso faz que ele instale tudo isso no container para você. Caso não queira alguma dessas depenências, acesse o ".env" que criou anteriormente e procure por: ``` WORKSPACE_ ``` e coloque ``` false ``` para tudo o que lhe for indesejável (informe "false" apenas para o que conhece, para não desconfigurar o ambiente indevidamente) - Lembrando que se mais tarde quiser esses programas, deverá marcar "true" e recompilar o container do "workspace" ou instalar cada programa no seu computador.

```
# Troque "nginx" por "apache2" caso queira utilizar o apache.

$ docker-compose up -d nginx mariadb redis workspace php-fpm
```

- OBS3: No windows, caso apareça "deseja compartilhar volume?", clique em "sim" ou "compartilhar".

- Troubleshooting: [Consulte a solução caso tenha problemas para executar os containers com conta do ADDS](#erro_iniciar_container_volume_compartilhado)

# Criando um projeto Laravel no Laradock

- No terminal, ainda dentro da pasta do Laradock, execute o workspace

```
$ docker-compose exec workspace bash
```

Dentro do workspace, utilize o composer para criar seus projetos Laravel.

```
$ composer create-project laravel/laravel seu_projeto --prefer-dist 
```

O projeto é criado na mesma pasta onde encontra-se a pasta do laradock (não dentro da pasta do laradock), Exemplo:

- Desktop\
    - Laradock\
    - seu_projeto1\
    - seu_projeto2\

# Acessando um ou vários projetos Laravel (ou outro web qualquer)


- Se estiver usando Nginx:
    - Na pasta "laradock/nginx/sites/" crie "seu_projeto.conf" a partir de "laravel.conf.example" modificando o trecho "seu_projeto" para o nome e pasta do seu projeto

```
server {
    # Redirect for https
    ...
    server_name    seu_projeto.local;
    ...
}

server {
    # For https
    ...
    server_name seu_projeto.local;
    root /var/www/seu_projeto/public;
    ...
}
```

- Se estiver usando Apache:
    - Na pasta "laradock/apache2/sites/" crie "seu_projeto.conf" a partir de "sample.conf.example" modificando o trecho "seu_projeto" para o nome e pasta do seu projeto

```
<VirtualHost *:80>
   ServerName seu_projeto.local
   Redirect / https://seu_projeto.local
   ...

<VirtualHost *:443>
   ServerName seu_projeto.local
   DocumentRoot /var/www/seu_projeto/public/
   ...
   <Directory "/var/www/seu_projeto/public/">
   ...
```

- No terminal, dentro da pasta do Laradock, reinicie os containers

```
$ docker-compose restart
```

- No windows, aponte os seus projetos no arquivo "hosts".

Em “C:\Windows\System32\drivers\etc”, insira: 

```
...
127.0.0.1 seu_projeto.local
127.0.0.1 seu_projeto2.local
```

- Acesse seu projeto pelo navegador: 
https://seu_projeto.local

- Repita os mesmos passos acima para cada novo projeto
- OBS 1: Não use “.dev” como extensão para seu ambiente de desenvolvimento ou produção, é uma recomendação do laradock por problema do Google Chrome.
- OBS 2: Para se conectar ao MariaDB, use o usuário e senha "root". [Consulte a solução caso o ocorra o problema "MySQL connection refused"](#erro_connection_refused_db)

# Happy Coding ;)

Para mais informações sobre o Laradock ou outras possibilidades: https://laradock.io/

Dica: Se estiver usando vscode, instale a extensão [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) - [Mais informações sobre a extensão](https://code.visualstudio.com/docs/containers/overview)


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

- Laradock: https://laradock.io
- Nginx SSL: https://zeropointdevelopment.com/how-to-get-https-working-in-windows-10-localhost-dev-environment/
- Apache SSL: https://github.com/laradock/laradock/issues/1316