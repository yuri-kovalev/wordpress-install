# Wordpress install
Установка [Wordpress](https://wpackagist.org) с помощью [Composer](https://getcomposer.org)

Версия сборки: ```1.0.0```

# Переменные
Все переменные используемые в этой документации

* ```%repository_url%``` - Путь репозитория где расположена Ваша тема;
* ```%package_name%``` - Название пакета Вашей темы для установки;
* ```%project_folrder%``` - Название директории где находиться проект;
* ```%company%``` - Имя пользователя или компании где распологается репозиторий с темой.



# Установка
Склонируйте этот репозиторий:
```
$ git clone https://github.com/u-kovalev/wordpress-install.git %project_folder%
```
Измените в ```composer.json``` путь - ```%repository_url%``` репозитория где расположена Ваша тема
```sh
"repositories":[
    {
      "type": "composer",
      "url": "https://wpackagist.org"
    },
    {
        "type": "git",
        "url": "%repository_url%"
    }
]
```
Добавьте новые или удалите ненужные плагины, а также добавьте пакет - ```%package_name%``` Вашей темы для установки.
Все плагины находятся в [WordPress Packagist](https://wpackagist.org)
```
"require": {
    "php": ">=5.6",
    "oscarotero/env": "^1.0",
    "vlucas/phpdotenv": "^2.0.1",
    "johnpbloch/wordpress": "*",
    "%package_name%": "*",
    ...
}
```
Замените ```%company%``` на имя пользователя или компании где распологается репозиторий с темой.
Пример: ```https://github.com/u-kovalev/wordpress-install.git``` - здесь ```u-kovalev``` = ```%company%```
```
"extra": {
    "wordpress-install-dir": "app/web/core",
    "installer-paths": {
      "app/content/themes/{$name}/": ["vendor:%company%"]
    }
}
```

Для установки выполните следующие команды :
```
$ cd %project_folrder%
$ composer install
```
Измените настройки в `app/web/.env`.

Структура папок
```
│── composer.json
│── readme.md
│── .gitignore
│── .env.example
└── app - весь проект
    ├── content - вместо wp-content
    │   ├── plugins
    │   ├── themes
    │   └── uploads
    └── web - корень сайта
        ├── environments - настройки окружения
        │   ├── development.php
        │   └── production.php
        ├── core - сам WordPress
        ├── wp-config.php
        └── index.php
```



# Настройки для [Nginx](http://nginx.org)

Файл ```site.ru.conf```
```
server {
        server_name site.ru *.site.ru;
        listen 192.0.78.9:80;
        
        #ip 192.0.78.9 - wordpress.com просто указано для примера
        

        root /var/www/site.ru/html/app/web;

        access_log /var/www/site.ru/logs/site.ru.access.log;
        error_log /var/www/site.ru/logs/site.ru.error.log;

        location ~* ^/wp-content/(.*)$ {
            alias /var/www/web.ru/html/app/content/$1;
        }

        location ~* ^/wp-?((admin|includes).*)$ {
            try_files $uri /core/wp-$1?$args;
        }

        include restrictions.conf;
        include wordpress.conf;
}
```
Файл ```restrictions.conf```
```
# Global restrictions configuration file.
# Designed to be included in any server {} block.</p>
location = /favicon.ico {
    log_not_found off;
    access_log off;
}

location = /robots.txt {
    allow all;
    log_not_found off;
    access_log off;
}

# Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
# Keep logging the requests to parse later (or to pass to firewall utilities such as fail2ban)
location ~ /\. {
    deny all;
}

# Deny access to any files with a .php extension in the uploads directory
# Works in sub-directory installs and also in multisite network
# Keep logging the requests to parse later (or to pass to firewall utilities such as fail2ban)
location ~* /(?:uploads|files)/.*.php$ {
    deny all;
}
```
Файл ```wordpress.conf```
```
# WordPress single blog rules.
# Designed to be included in any server {} block.
# This order might seem weird - this is attempted to match last if rules below fail.
# http://wiki.nginx.org/HttpCoreModule

location / {
    index index.php;
    try_files $uri $uri/ /index.php?$args;
}

# Add trailing slash to */wp-admin requests.
#rewrite /wp-admin$ $scheme://$host$uri/ permanent;

# Directives to send expires headers and turn off 404 error logging.

location ~* ^.+.(xml|ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|css|rss|atom|js|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
    access_log off; log_not_found off; expires max;
}

# Uncomment one of the lines below for the appropriate caching plugin (if used).
# include global/wordpress-wp-super-cache.conf;
# include global/wordpress-w3-total-cache.conf;
# Pass all .php files onto a php-fpm/php-fcgi server.

location ~ .php$ {
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_split_path_info ^(.+\.php)(/.*)$;
    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    fastcgi_param DOCUMENT_ROOT $realpath_root;
}
```