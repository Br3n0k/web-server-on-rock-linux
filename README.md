![Brazil](https://raw.githubusercontent.com/stevenrskelton/flag-icon/master/png/16/country-4x3/br.png "Brazil")

# Implementando um ambiente 

Um passo a passo rápido de construir um ambiente para servidor WEB com Git, PHP 8.3, MySQL(MariaDB), NodeJS e Nginx funcional no Rocky Linux.

- [Atualização do Sistema](#atualização-do-sistema)
- [Instalação do Git](#instalação-do-git)
- [Instalar o PHP 8.3](#instalar-o-php)
- [Instalar o Composer](#instalar-o-composer)
- [Instalar Nginx](#instalar-nginx)
- [Instalar o MySQL/MariaDB](#instalar-o-mariadb)
- [Instalar o NodeJS](#instalar-o-node)

______
## Atualização do Sistema

```
sudo dnf update && sudo dnf upgrade && sudo dnf upgrade --refresh
```
______
## Instalação do Git

```
sudo dnf install git
```
______
## Instalar o PHP

1. Faça o download e instalação do PHP
```
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf config-manager --set-enabled remi
sudo dnf module enable php:remi-8.3
sudo dnf install php-{common,gmp,fpm,curl,intl,pdo,mbstring,gd,xml,cli,zip,mysqli,opcache,soap,json,mysqlnd} -y
sudo dnf update && sudo dnf upgrade
```

2. Inicie e Habilite o PHP para inicialização automatica
```
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

3. Edite no arquivo /etc/php-fpm.d/www.conf
```
listen.owner = nginx
listen.group = nginx
```

4. Edite no arquivo /etc/php.ini
```
date.timezone = UTC
cgi.fix_pathinfo=1
```

5. Restart no serviço do PHP
```
systemctl restart php-fpm
```

______
## Instalar o Composer

1. Faça o download do wget
```
dnf -y install wget
```

2. Faça o download do composer
```
wget https://getcomposer.org/installer -O composer-installer.php
```

3. Instale o Composer
```
php composer-installer.php --filename=composer --install-dir=/usr/local/bin 
```

4. Adicione as permissões sobre o arquivo do composer
```
chmod +x /usr/local/bin/composer
```

5. Verifique se tudo deu certo confirmando a versão do composer
```
composer --version
```

______
## Instalar Nginx

1. Download e instalação do nginx
```
sudo dnf install nginx
nginx -v
```

2. Inicialização e Habilitação do start automatico do nginx
```
sudo systemctl enable --now nginx
sudo systemctl start nginx
```

3. Liberando o serviço e as portas no firewall
```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
```

4. Reload no firewall para reconhecer as novas regras
```
firewall-cmd --reload
```

______
## Instalar o MariaDB

```
dnf install mariadb-server
systemctl enable mariadb  OU dnf module enable mariadb:10.5
systemctl start mariadb
mysql_secure_installation
```

## Instalar o Node

Sobre a versão
> [!IMPORTANT]
> Aqui estamos instalando a última versão LTS do NodeJS, mas você pode instalar a que achar melhor só trocando a versão. Mais [informações aqui](https://github.com/nodesource/distributions/blob/master/README.md#debinstall).

1. Faca o download do NodeJS

```sh
sudo dnf update -y
curl -fsSL https://rpm.nodesource.com/setup_21.x | sudo bash -
sudo dnf install -y nodejs
node -v
npm -v
```

2. Configurando o NVM (Node Version Manager)

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
command -v nvm
nvm install node # for the latest version
nvm install 16.14.0 # for a specific version
```

3. Instalando o Yarn (Opcional)

```sh
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo rpm --import https://dl.yarnpkg.com/rpm/pubkey.gpg
sudo dnf install yarn
```

## Observações
Notas
> [!IMPORTANT]
> Alguns problemas podem surgir com o SElinux para quando mudar o diretorio da pasta de publicação do nginx, o comando abaixo permite você liberar o diretorio para publicação
```sh
chcon -R -t httpd_sys_content_t /home/www/seusite
```

>Você pode optar por utilizar portas diferentes, e somente a liberação no firewall não é o suficiente, você deve listar essa porta como http para que o nginx possa escrever no socket
```sh
semanage port -a -t http_port_t  -p tcp 8090
```

> Se o servidor for para um framework Laravel é preciso:

1. Verificar as permições nas pastas do laravel com usuario utilizado para o nginx
```
sudo -R nginx:nginx /srv/www/hosts/brendown.tech/public_html/storage/
sudo -R nginx:nginx /srv/www/hosts/brendown.tech/public_html/bootstrap/cache/
sudo -R 0777 /srv/www/hosts/brendown.tech/public_html/storage/
sudo -R 0775 /srv/www/hosts/brendown.tech/public_html/bootstrap/cache/
sudo chown -R :www-data /srv/www/hosts/brendown.tech/public_html
sudo chmod -R 777 /srv/www/hosts/brendown.tech/public_html/storage
sudo chmod -R 777 /srv/www/hosts/brendown.tech/public_html/database
```

2. Crie um arquivo de configurações para o laravel
```
/etc/nginx/conf.d/laravel.conf
```
```
server {
       listen 80;
       server_name laravel.yourdomain.com;
       root        /var/www/html/laravelsite/public;
       index       index.php;
       charset utf-8;
       gzip on;
	      gzip_types text/css application/javascript text/javascript application/x-javascript  image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
        location / {
        	try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php {
                include fastcgi.conf;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/run/php-fpm/www.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```

3. Verifique a estrutura do arquivo de configurações do nginx
```
nginx -t
```

4. Restart nos serviços do nginx e php
```
systemctl restart php-fpm
systemctl restart nginx
```
