
# Package Laravel Project in Docker

## Local Example

```
git clone https://github.com/Ruslan-Aliyev/laravel_dockerized.git
cd laravel_dockerized

docker-compose run --rm php cp .env.example .env
docker-compose run --rm php chmod -R 777 storage
docker-compose run --rm php composer install 
# To map composer cache for faster speed by using these flags: -v $HOME/.cache/composer:/tmp -e COMPOSER_HOME=/tmp
docker-compose run --rm php php artisan key:generate

docker-compose run --rm node npm install # Or: ./vendor/bin/sail npm install

docker-compose up -d --build # ( = docker-compose build && docker-compose up -d) # Or: ./vendor/bin/sail up

# For DB
docker-compose exec php php artisan migrate # Or: docker-compose run —rm php php artisan migrate # Or: ./vendor/bin/sail artisan migrate
docker-compose exec php php artisan db:seed

# For frontend UI
docker-compose run —rm php composer require laravel/ui
docker-compose exec php php artisan ui vue --auth
docker-compose run --rm node npm install # Or: ./vendor/bin/sail npm install

###
OR:
docker-compose exec php bash
> composer require laravel/ui
> php artisan ui vue --auth  
> npm install
###

```

`localhost` is your php website,  
`localhost:8089` is your phpmyadmin,   
`root` and `{no password}` is your phpmyadmin login.

## Without Docker Example

For comparison:

```
composer create-project laravel/laravel  projectname
# Or 
# composer global require laravel/installer
# ~/.composer/vendor/laravel/installer/bin/laravel new projectname

# composer install
# cp .env.example .env
# php artisan key:generate
php artisan migrate
php artisan db:seed

# For frontend UI
composer require laravel/ui
php artisan ui vue --auth  
php artisan ui bootstrap
npm install
npm run dev

php artisan serve
```

## Production Example

```
docker build -t atabegruslan/docker4laravel-php:0.0.1 -f ./docker/php_prod.Dockerfile .
docker build -t atabegruslan/docker4laravel-nginx:0.0.1 -f ./docker/nginx_prod.Dockerfile .

# Push to DockerHub:
docker login --username=atabegruslan
docker push atabegruslan/docker4laravel-php:0.0.1
docker push atabegruslan/docker4laravel-nginx:0.0.1

docker-compose up -d --build
```

## Notes

- `docker-compose run —rm` starts new container then deletes it once composer finishes running that command.
- `docker-compose exec` runs command inside an existing container.
- Sail run inside docker container.
- Regular equivalent commands runs on localhost.

### Redis

https://laravel.com/docs/8.x/redis  
```
docker-compose exec redis redis-cli
> ping
PONG
```
https://beyondco.de/blog/the-ultimate-guide-to-php-artisan-tinker  
```
docker-compose exec php php artisan tinker
>>> Illuminate\Support\Facades\Redis::ping()
True
>>> Illuminate\Support\Facades\Redis::set('counter', 1)
True
>>> Illuminate\Support\Facades\Redis::get('counter')
"1"
```

## Tutorials

- https://webomnizz.com/containerize-your-laravel-application-with-docker-compose/amp <sup>Intro</sup>
- https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/good_notes.md <sup>Very Good</sup>
- https://help.clouding.io/hc/en-us/articles/360010679999-How-to-Deploy-Laravel-with-Docker-on-Ubuntu-18-04
- https://www.youtube.com/watch?v=2exXt7XLPBQ
- https://www.youtube.com/watch?v=Ip2btxIUdNw
- https://github.com/othyn/docker-compose-laravel
- Sail: https://github.com/atabegruslan/Others/blob/master/Development/laravel.md#sail
- Laradock: https://github.com/atabegruslan/Others/blob/master/Development/laravel.md#laradock