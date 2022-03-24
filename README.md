# DOCKER RUBY ON RAILS PARA RASPBERRY PI

Conjunto de contenedores para subir una aplicacion de Ruby On Rails a produccion. A un servidor o en una rapsberry pi, la imagen se compilada para las plataformas de linux/amd64,linux/arm64,linux/arm/v7.

Se compone de tres contenedores, Postgres 12.2, Ruby 2.6 y Ngnix 1.17, orquestado por docker-compose

los paso a seguir

1. una vez clonado el repositorio y renombramos la carpeta al nombre de nuestro proyecto

2. copiar nuestra aplicacion dentro del directorio web por ejemplo, ejemplo de copiar a nuestra raspi

    rsync -avzh -e ssh web/ pi@192.168.1.180:~/ejemplo/web


### construir una imagen de docker de rails

docker build . -f Dockerfile_rails --build-arg RUBY_VERSION=2.6.3 -t manologcode/rails263



### instalar las gemas

```
docker run -it --rm --name rails_app -v $PWD/web:/app -v $PWD/_bundle:/usr/local/bundle manologcode/rails260 bundle install --without development test

```

### add credentials

corre el contenedor de rails run bash

```
docker run -it --rm --name rails_app -e RAILS_ENV=production -v $PWD/web:/app -v $PWD/_bundle:/usr/local/bundle manologcode/rails260 bash

```

dentro del contenedor ejecutar 

```
SECRET_KEY_BASE=production_test_key EDITOR="nano" rails credentials:edit
```

pegar el contenido con las los datos personalizados a nuestro proyecto

```
postgres:
  base: base
  user: user_app_db
  password: xxxxxxxxxxxxxxxxxxxx

```

crear la base de datos

```
docker-compose up -d


docker exec -it db_psql_app psql -U root

CREATE USER user_app_db;
ALTER USER user_app_db WITH PASSWORD 'xxxxxxxxxxxxxxxxxxxx';
CREATE DATABASE base_app OWNER user_app_db;
grant all privileges on database base_app to root;
\c base_app;
CREATE EXTENSION IF NOT EXISTS "unaccent";
\q

```

    docker-compose up -d


crear base de datos manualmente


docker exec -it rails_app sh

```
RAILS_ENV=production rails db:migrate

RAILS_ENV=production rails db:seed

RAILS_ENV=production rails assets:precompile

```

## Docker build

para forzar y crear alguno de los contenedores manualmente podemos crearlos:

docker build --no-cache -f Dockerfile_rails --build-arg RUBY_VERSION=2.6.0  --pull . -t manologcode/rails260:latest

docker build --no-cache -f Dockerfile_nginx --build-arg NGINX_VERSION=1.17.3  --pull . -t manologcode/ngnix117_rails:latest

