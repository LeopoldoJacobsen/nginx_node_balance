antes de iniciar as etapas, execute:
>>> command: docker-compose up -d


# ETAPA 1 CONFIGURAR NGINX SERVER

Como foi instalado o nginx alphine é necessario add o bash
 >>> command: docker-compose exec nginx apk add 
Para entrar no nginx
>>> command: docker-compose exec nginx bash
Add vim no nginx para editar arquivos
>>> command: apk add vim
Arquivo de configuracao do nginx, esse arquivo raramente será mexido, é configuracoes de tuning
>>> command: vim /etc/nginx/nginx.conf
O Arquivo que será configurado é o default.conf, esse arquivo é o que faz o welcome. remover tudo de #error_page
>>> command: vim /etc/nginx/conf.d/default.conf

O arquivo default fica:

```
server {
    listen          80;         //porta default
    server_name     localhost;  //domain, exemplo.com

    location / {                //quando acessar / redireciona
        root /usr/share/nginx/html
        index index.html index.html
    }
}
```

Após configurar o arquivo, pode ser rodado o comando abaixo para testar se o arquivo está ok

>>> command: nginx -t



# ETAPA 2 CONFIGURAR NODE1

Iniciar instalando bash e node
>>> command: docker-compose exec node1 apk add bash vim
Para executar o bash
>>> command: docker-compose exec node1 bash
Acessar arquivo HTML para renomear para NODE1 e ver acontecer
>>> command: vim /etc/share/nginx/html/index.HTML


# ETAPA 3 CONFIGURAR PROXY REVERSO NO SERVICE NGINX 

Entrar no nginx e mudar o location
>>> command: docker-compose exec nginx bash
>>> command: vim /etc/nginx/conf.d/default.conf
o proxy_pass listado no arquivo abaixo diz que quando tiver uma requisicao no http://localhost/ será redirecionado para o http://node1

```
server {
    listen          80;         //porta default
    server_name     localhost;  //domain, exemplo.com

    location / {                //quando acessar / redireciona
        proxy_pass http://node1;
    } 
}
```
rodar nginx -t para ver se o arquivo tem alguma falha
>>> command: nginx -t
rodar nginx -g reload para reiniciar o server
>>> command: nginx -s reload

# ETAPA 4 CONFIGURAR NODE2

nessa etapa vamos iniciar o node balance
Iniciar instalando bash e node
>>> command: docker-compose exec node2 apk add bash vim
Para executar o bash
>>> command: docker-compose exec node2 bash
Acessar arquivo HTML para renomear para NODE2 e ver acontecer
>>> command: vim /etc/share/nginx/html/index.HTML

# ETAPA 5 CONFIGURAR NODE BALANCE NO NGINX
entrar no node nginx
>>> command: docker-compose exec nginx bash
acessar arquivo de configuracao
>>> command: vim /etc/nginx/conf.d/default.conf
para configurar o nodebalance é muito simples
colocar 'upstream name{}'
```
upstream nodes{
    server node1;
    server node2;
}

server {
    listen          80;         //porta default
    server_name     localhost;  //domain, exemplo.com

    location / {                //quando acessar / redireciona
        proxy_pass http://nodes;
    } 
}
```

# ETAPA 6 - Adicionar ACCESS log do node balance

o arquivo de configuracao abaixo possibilita o log dos nodes
o nome do arquivo é a sua escolha, no caso deixei nginx-access pra ficar mais fácil de identificar

```
upstream nodes{
    server node1;
    server node2;
}

server {
    listen          80;         //porta default
    server_name     localhost;  //domain, exemplo.com

    access_log /var/log/nginx/nginx-access.log // se nao pegar o ip real, colocar main na frente

    location / {                //quando acessar / redireciona
        proxy_pass http://nodes;
        proxy_set_header X-Real-IP $remote_addr; // passa o ip do client que consultou o server
    } 

}
```

alterar arquivo nginx.conf
>>> command: vim /etc/nginx/nginx.conf
editar arquivo de log_forma e alterar o main para

```
    log_format main '$remote_addr' por '$http_x_real_ip' 
```
