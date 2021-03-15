# Deploy-Laravel-ubuntu-server
 Passo a passo para realizar o deploy de uma aplicação laravel em ubuntu server 18.04 LTE / Step by step to deploy a laravel application on ubuntu server 18.04 LTE 

- Passo 1: Atualizações automaticas
    - sudo apt-get update
    - sudo apt-get install -y unattended-upgrades

- Passo 2: Softwares necessarios
    - sudo apt-get install -y software-properties-common
    - sudo add-apt-repository ppa:nginx/stable
    - sudo apt-get update
    - sudo apt-get install -y nginx
    - sudo service nginx restart
    //Depois desses comandos etapa verificar se deu certo usando o dns no navegador, se tudo estiver okay deve aparecer uma tela escrita "Welcome to nginx on Debian!"
    - sudo apt-get install -y git curl wget 
    - PHP 7 (Needed for Laravel 5.7)
        - sudo apt-get purge php5-fpm
        - sudo apt-get --purge autoremove
        - sudo apt-get install -y software-properties-common
        - sudo apt-get install -y python-software-properties
        - sudo add-apt-repository ppa:ondrej/php
        - sudo apt-get update
        - Install PHP 7 on the server
            - sudo apt-get install -y php7.3 php7.3-fpm php-mysql php7.3-mysql php-mbstring php-gettext php-doctrine-dbal php-xml redis-server
            - sudo systemctl restart php7.3-fpm    
    - Composer 
        - curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
    - Mysql
        - sudo apt-get install -y mysql-server
    - Node e NPM
        - sudo apt install nodejs
        - sudo apt install npm
        - sudo apt-get update
        - sudo npm install --global yarn gulp-cli grunt-cli

- Passo 3: Como acessar seu aplicativo da Web via Git
    - cd /var/www
    - sudo git clone your repositorie link

- Passo 4: Configurando servidor 
    - sudo nano /etc/nginx/sites-available/default
    - Dentro do arquivo deixe desse jeito
       server {
    listen 80;
    server_name _;
    root "/var/www/AppName/public";

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;

    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

- Passo 5: Setando permissões
    - sudo chmod 777 -R storage bootstrap/cache

- Passo 6: Instalando dependencias
    - sudo  apt install zip unzip php-zip
    - sudo apt-get install php-curl  
    - sudo composer install
    - sudo npm install
    - sudo npm run production

- Passo 7: Configurando Mysql
    - sudo mysql -u root -p
    - create database database_name
    - Criando usuario com todos os previlegios
        - CREATE USER 'novousuario'@'localhost' IDENTIFIED BY 'password';
	- GRANT ALL PRIVILEGES ON * . * TO 'novousuario'@'localhost';
        - FLUSH PRIVILEGES;

- Passo 8: Configurando o ambiente Laravel
    - cd /var/www/AppName
    - sudo cp .env.example .env
    - nano .env #Configure o .env como desejar

- Passo 9: Configurar LetsEncrypt para SSL
    - - sudo apt-get install -y software-properties-common
    - sudo add-apt-repository ppa:certbot/certbot
    - sudo apt-get update
    - sudo apt-get install -y python-certbot-nginx

- Passo 10: Configure o host virtual do site
    - sudo nano /etc/nginx/sites-available/default
    - dentro do nano 
        - mude de:
            - server_name _;
        - para:
            - server_name example.com www.example.com;

        - mude de:
            - listen 80 default_server;
            - listen [::]:80 default_server;
        - para:
            - listen 80 default_server;
            - listen [::]:80;
   -  sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
    - sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
    

links e correções de alguns erros
https://sempreupdate.com.br/como-destravar-o-var-lib-dpkg-lock-ubuntu-debian-linux-mint/ #Corrigir a mensagem /var/lib/dpkg/lock
sudo  apt install zip unzip php-zip #Para corrigir o erro do ZIP and UNZIP na hora do composer install

Erro time out 504 (nginx)
-no arquivos nginx.conf
	adiciona dentro do bloco http o seguinte comando: fastcgi_read_timeout 300;
-sudo service nginx reload

Cron Job
* * * * * cd /var/www/Organizacional-ccbeu && php artisan schedule:run >> /dev/null 2>&1
sudo crontab -e

Para redefinição de senha
	No .env
		MAIL_DRIVER=smtp
		MAIL_HOST=smtp.gmail.com
		MAIL_PORT=587
		MAIL_USERNAME=email@gmail.com
		MAIL_PASSWORD=senha
		MAIL_ENCRYPTION=tls
	E realize a instalar do guzzle

ERRO Temporary failure in name resolution
Solução => sudo systemctl enable systemd-resolved.service

ERRO file_put_contents no storage
Solução => 
	 php artisan cache:clear
	 chmod -R gu+w storage
	 chmod -R guo+w storage
