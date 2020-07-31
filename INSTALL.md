# Cheatsheet

Demonstro aqui como compilar e gerar os HTMLs estáticos dos Cheat Sheets.

## Instale as dependências
```
# echo deb http://nginx.org/packages/mainline/ubuntu/ `lsb_release --codename --short` nginx > /etc/apt/sources.list.d/nginx.list
# curl -s http://nginx.org/keys/nginx_signing.key | apt-key add -
# echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
# curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
# apt-get update && apt-get -y upgrade
# apt install nginx ruby-full build-essential zlib1g-dev yarn
# gem install jekyll bundler jekyll-theme-primer
```

## Configure o ambiente do usuário
```
# clone git clone https://github.com/sec4us-training/cheatsheets.git
# cd cheatsheets
# yarn install
# bundle install
# bundle update i18n
```

## Gere os arquivos HTML
```
Execute o comando abaixo para gerar todo o conteúdo do site
Este processo atualizará o /assets/packed/ e _includes/2017/critical/ com as fontes em _parcel/

yarn build
```

## Gere os arquivos HTML de produção
```
JEKYLL_ENV=production bundle exec jekyll build
```

## Copie os arquivos para o local de publicação do site
Supondo que o site está hospedado em /var/www/cheatsheets/
```
rsync -av cheatsheets/_site/* /var/www/cheatsheets/
```

## Edite o arquivo do NGINX (/etc/nginx/nginx.conf) conforme abaixo
```
user  nginx;
worker_processes  1;
 
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
 
 
events {
    worker_connections  1024;
}
 
 
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
 
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    server_names_hash_bucket_size  256;
 
    client_max_body_size 10m;
 
    log_format log_standard '$remote_addr, $http_x_forwarded_for - $remote_user [$time_local] "$request_method $scheme://$host$request_uri $server_protocol" $status $body_bytes_sent "$http_referer" "$http_user_agent" to: $upstream_addr';
 
    access_log /var/log/nginx/access.log log_standard;
    error_log /var/log/nginx/error.log;
 
    sendfile        on;
    #tcp_nopush     on;
 
    keepalive_timeout  65;
 
    #gzip  on;
 
    include /etc/nginx/conf.d/*.conf;
}
```

## Edite o arquivo do host NGINX (/etc/nginx/conf.d/default.conf) conforme abaixo
```
server {
    listen      80;
    server_name _;
 
   location / {
        root /var/www/cheatsheets/;

        if (!-f "${request_filename}index.html") {
            rewrite ^/(.*)/$ /$1 permanent;
        }

        if ($request_uri ~* "/index.html") {
            rewrite (?i)^(.*)index\.html$ $1 permanent;
        }   

        if ($request_uri ~* ".html") {
            rewrite (?i)^(.*)/(.*)\.html $1/$2 permanent;
        }

        try_files $uri.html $uri $uri/ /index.html;
   }
 
}
```

## Recarregue a config do NGINX
```
# nginx -s reload
```

## Agradecimentos
Agradeço ao [@rstacruz](https://ricostacruz.com/) criador do site [devhints.io](https://devhints.io/) pelo trabalho e apoio neste processo de criação do site.


## Fontes de estudo 
https://gist.github.com/rickharrison/6410194
https://www.digitalocean.com/community/tutorials/how-to-get-started-with-jekyll-on-an-ubuntu-vps
https://yuan3y.com/2017/09/make-jekyll-serve-a-systemd-service/
https://github.com/rstacruz/cheatsheets/blob/master/CONTRIBUTING.md
