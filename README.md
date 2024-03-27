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

```
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf config-manager --set-enabled remi
sudo dnf module enable php:remi-8.3
sudo dnf install php-{common,gmp,fpm,curl,intl,pdo,mbstring,gd,xml,cli,zip,mysqli,opcache}
sudo dnf update && sudo dnf upgrade
```

______
## Instalar o Composer
```
dnf -y install wget
wget https://getcomposer.org/installer -O composer-installer.php
php composer-installer.php --filename=composer --install-dir=/usr/local/bin 
composer --version
```

______
## Instalar Nginx

```
sudo dnf install nginx
sudo systemctl enable --now nginx
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
nginx -v
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
