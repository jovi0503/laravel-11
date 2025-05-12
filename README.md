Git-Clone
```sh
cd laravel-11/
```


Crie o Arquivo .env
```sh
cp .env.example .env
```


Atualize essas variáveis de ambiente no arquivo .env
```dosini
APP_NAME="laravel"
APP_URL=http://localhost:8989

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=nome_que_desejar_db
DB_USERNAME=nome_usuario
DB_PASSWORD=senha_aqui

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```


Suba os containers do projeto
```sh
docker-compose up -d
```


Acesse o container
```sh
docker-compose exec app bash
```


Instale as dependências do projeto
```sh
composer install
```


Gere a key do projeto Laravel
```sh
php artisan key:generate
```


Acesse o projeto
http://http://localhost:8000


extra:
Caso o phpmyadmin apresente o erro: 
mysql::real_connect(): (HY000/2002): php_network_getaddresses: getaddrinfo for db failed: Name or service not known
Utilize este código no docker-compose.yml:
´´´ymal
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    working_dir: /var/www/
    volumes:
      - ./:/var/www
    depends_on:
      - redis
    networks:
      - laravel

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - ./:/var/www
      - ./docker/nginx/:/etc/nginx/conf.d/
    networks:
      - laravel

  db:
    container_name: db
    image: mysql:8.0
    platform: linux/x86_64
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: ${DB_DATABASE:-laravel}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD:-root}
      MYSQL_PASSWORD: ${DB_PASSWORD:-secret}
      MYSQL_USER: ${DB_USERNAME:-laravel}
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - laravel
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 5

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    platform: linux/x86_64
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 0
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD:-root}
    networks:        
      - laravel

  redis:
    image: redis:latest
    networks:
      - laravel
      

networks:
  laravel:
    driver: bridge

volumes:
  mysql_data:
  ´´´
