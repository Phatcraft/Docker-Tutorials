# Docker Slim Framework 
Hướng dẫn setup Slim Framework trên Docker

## 1. Cấu trúc của 1 project
+ Folder `app`: Folder chứa toàn bộ dự án Slim Framework.
+ `Dockerfile`: File setup 1 image riêng.
+ `docker-compose.yaml`: File cấu hình Docker Compose.

## 2. Setup
### 2.1. Tạo 1 project Slim Framework
Bạn có thể tạo 1 Slim Framework trên folder `app`.

### 2.2. Tạo `Dockerfile`
`Dockerfile` nhằm tạo 1 image custom từ `php:8.4-rc-fpm`, bao gồm:
+ Cài đặt các package cần thiết để cài đặt `composer`.
+ Load toàn bộ Slim project vào folder.
+ Cài đặt các dependencies cho project.

Source code `Dockerfile`
````
FROM php:8.4-rc-fpm

# Install composer package
RUN apt-get update && apt-get install -y git zip unzip

# Copy composer files from image
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Change workdir to /var/www/html
WORKDIR /var/www/html

# Copy app's source code to /var/www/html
COPY app/ .

# Install composer to create vendor folder
RUN composer install

CMD [ "php-fpm" ]
````
> [!TIP]
> Bạn có thể thay đổi vị trí project khác `/var/www/html` nếu muốn.

### 2.3. Tạo `docker-compose.yaml`
File `docker-compose.yaml` nhằm setup các container để hệ thống làm việc hiệu quả.<br>
Hệ thống bao gồm:
+ `app`: Được tạo bằng custom image qua `Dockerfile`, là nơi host chủ đạo project.
+ `nginx`: Container web server.

Source code `docker-compose.yaml`
````
services:
  # Phatcraft Archive App 
  app:
    build:
      context: .
      dockerfile: docker/Dockerfile
    container_name: app
    volumes:
      - ./app:/var/www/html
      - vendor:/var/www/html/vendor
  
  # Nginx Server
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./app:/var/www/html
      - ./docker/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

volumes:
  vendor:
````
Trong đó, `vendor` là 1 volume được tạo nhằm tránh việc mất folder `vendor` khi `composer install`.<br>
Bạn có thể thay đổi cấu trúc file cho phù hợp.

### 2.4. Tạo `default.conf`
File `default.conf` được dùng để config nginx

````
server {

    listen 80;
    root /var/www/html/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass   app:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        include        fastcgi_params;
    }
}
````
Bạn có thể custom header, thiết lập https tại file này.
