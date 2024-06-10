# Very good tutorials' notes

https://www.youtube.com/playlist?list=PL1LQwTE3lBhSnaL7j90AUJyvC9mFCKhZm <sup>Very Good</sup>

## Tutorial 1 - Install Laravel from a composer container

- https://www.youtube.com/playlist?list=PL1LQwTE3lBhSnaL7j90AUJyvC9mFCKhZm Episode 1

### Basic

1. Getting composer image (https://hub.docker.com/_/composer) - most simple: `docker run composer`
2. Delete containwr when no longer needed: `docker run --rm composer`
3. Map volume: `docker run --rm -v $(pwd):/app composer`
4. Set the working directory in the container: `docker run --rm -v $(pwd):/app -w /app composer` . Any commands you pass into the container will be run from the `/app` directory.
5. Install Laravel: `docker run --rm -v $(pwd):/app -w /app composer composer create-project laravel/laravel composer4laravel`
6. Use your user instead of `root`: `docker run -u $(UID):$(UID) --rm -v $(pwd):/app -w /app composer composer create-project laravel/laravel composer4laravel`

This is slow because composer is downloading each dependency from scratch, in this newly created filesystem, without cache.

### Fast

https://hub.docker.com/r/clevyr/prestissimo install Composer dependencies in parallel

1. Install `prestissimo`: `docker pull clevyr/prestissimo`
2. Install the whole thing: `docker run -u $(UID):$(UID) --rm -v $(pwd):/app -w /app clevyr/prestissimo composer create-project laravel/laravel composer4laravel`

### Faster

Map composer container's cache to our localhost's cache.

As seen in https://github.com/composer/docker/blob/master/1.10/Dockerfile#L40 , the composer container's cache is `/tmp`

1. Map volume: `docker run -u $(UID):$(UID) -v $HOME/.cache/composer:/tmp --rm -v $(pwd):/app -w /app clevyr/prestissimo composer create-project laravel/laravel composer4laravel`

### Other Links

- Laravel documentation: https://laravel.com/docs/7.x/deployment#nginx
- Docker documentation: https://docs.docker.com/
- Composer: https://getcomposer.org/
- PHP official Docker Image: https://hub.docker.com/_/php
- Nginx official Docker Image: https://hub.docker.com/_/nginx
- MySQL official Docker Image: https://hub.docker.com/_/mysql
- Redis official Docker Image: https://hub.docker.com/_/redis

## Tutorial 2 - Locally, with `docker-compose`

- https://www.youtube.com/playlist?list=PL1LQwTE3lBhSnaL7j90AUJyvC9mFCKhZm Episode 2
  - https://github.com/tonysm/dockerforlaravel-app

- https://www.youtube.com/playlist?list=PL36CGZHZJqsWXjf4GeQBLUl7CK1dodKuC Episode 1-3
  - https://github.com/aschmelyun/docker-compose-laravel

### Dockerfile explained

```
FROM php:7.4-fpm 
```
- https://hub.docker.com/_/php , fpm flavor
- https://stackoverflow.com/questions/54270656/difference-between-apache-vs-fpm-in-php-docker-image/54270756

```
RUN apt-get update \
  && apt-get install --quiet --yes --no-install-recommends \ 
    libzip-dev \
    unzip \
  && docker-php-ext-install zip \     
```
- Install composer's dependencies.
- `--quite` means not verbose. `--yes` means answer yes for everything during installation. `--no-install-recommends` means not installing recommended dependencies.

```
RUN apt-get update \
  && apt-get install --quiet --yes --no-install-recommends \
    libzip-dev \
    unzip \
  && docker-php-ext-install pcntl zip pdo pdo_mysql \
  && pecl install -o -f redis-5.1.1 \
  && docker-php-ext-enable redis
```
- Also install DB drivers
- Install Redis (has to be via pecl)
  - Use the php image's `docker-php-ext-enable` helper to enable the Redis dependency. 
    - `docker-php-ext-install` installs and enables, but pecl only installs without enabling.

- Copy this binary: `/usr/bin/composer` from this composer image: https://hub.docker.com/_/composer to this place in our container: `/usr/bin/`
- https://dev.to/jonesrussell/install-composer-in-custom-docker-image-3f71

#### User and permissions:

```
RUN groupadd --gid 1000 appuser \
  && useradd --uid 1000 -g appuser \
     -G www-data,root --shell /bin/bash \
     --create-home appuser

USER appuser
```
- For comparison - This is how you would normally create a new user: `useradd appuser -m -c "comment" -G sudo -s /bin/bash` 
  - `-c` for comment, `-m` to create user's home directory and `-s /bin/bash` to define user's shell.

- Make `UID` and `GID` more dynamic: 
  - If building the image directly: `docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) -t <IMAGE NAME> .` 
    ```
    RUN groupadd --gid ${GID} appuser \
      && useradd --uid ${UID} -g appuser \
         -G www-data,root --shell /bin/bash \
         --create-home appuser
    ```
    - https://faun.pub/set-current-host-user-for-docker-container-4e521cef9ffc 
      - https://medium0.com/m/global-identity?redirectUrl=https%3A%2F%2Ffaun.pub%2Fset-current-host-user-for-docker-container-4e521cef9ffc
  - If via docker-compose: `UID=${UID} GID=${GID} docker-compose up`
    ```
    services:
      servicename:
        image: startingimage
        user: "${UID}:${GID}"
    ```
    - https://stackoverflow.com/questions/56844746/how-to-set-uid-and-gid-in-docker-compose  

- https://github.com/atabegruslan/Others/blob/master/Virtual/docker.md#root-priviledge-issue

### Misc notes

- Docker-Compose Container Name vs Image: 
  - https://docs.docker.com/compose/compose-file/compose-file-v3/#container_name 
    - > Use the command `docker exec -it <container name> /bin/bash` to get a bash shell in the container. Or directly use `docker exec -it <container name> <command>` to execute whatever command you specify in the container.
    - The container name you define in `docker-compose.yaml` `container_name: xxx` is the name you can later use to get into the container `docker exec -it xxx`
    - The tag name that you build images with `docker build -t yyy .` is the name you'll refer to in `docker-compose.yaml` `image: yyy`
  - https://docs.docker.com/compose/compose-file/compose-file-v3/#image
    - > If the image does not exist, Compose attempts to pull it, unless you have also specified build, in which case it builds it using the specified options and tags it with the specified tag.
- Docker-Compose Entrypoint: https://docs.docker.com/compose/compose-file/compose-file-v3/#entrypoint
- Docker-Compose Alias: https://gist.github.com/mborodov/cb95fc4e355d28ea9660b432eac7c4d8

### Rest of the major changes:

https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/Illustrations/IMPROVE_docker_laravel_commit.pdf

## Tutorial 2.5 (i) - Networks

### Networks

- https://www.youtube.com/playlist?list=PL36CGZHZJqsWXjf4GeQBLUl7CK1dodKuC Episode 2
- https://docs.docker.com/compose/networking/#specify-custom-networks
  - > By default Compose sets up a single network for your app. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name.
  - Unless you make your own custom network:
    ```
    version: "3"

    services:
      aaa:
        ...
        networks:
          - nnn
      bbb:
        ...
        networks:
          - nnn

    networks:
      nnn:
    ```

### Multiple apps

- https://www.youtube.com/playlist?list=PL36CGZHZJqsWXjf4GeQBLUl7CK1dodKuC Episode 4
  - Different names and ports (`container_name: name_unique` & `ports: -"{unique_port_number}:80"` in `docker-compose.yml`)
    - `.env`'s database credentials can be the same, because each project's docker network is isolated (Even when the custom defined network name is the same).
  - Frontend and Backend comminucation
    1. First things first, it's best to use Laravel 7+'s https://laravel.com/api/8.x/Illuminate/Support/Facades/Http.html (though it still needs `composer install guzzlehttp/guzzle`)
    2. Your local machine has access to both frontend and backend apps, but those apps in their respective docker container networks are isolated from eachother. So we need to use a **Docker Bridge Network**: https://docs.docker.com/network/bridge/
    3. For the backend app's `docker-compose`:
      ```
      version: "3"

      services:
        aaa:
          ...
          networks:
            - nnn
            - app-shared
        bbb:
          ...
          networks:
            - nnn
            - app-shared

      networks:
        nnn:
        app-shared:
          driver: bridge
      ```
    4. Run `docker-compose up -d --build` on the backend app to get the name of the bridge network `Creating network "xxx" with driver "bridge"`
    5. For the frontend app's `docker-compose`:
      ```
      version: "3"

      services:
        aaa:
          ...
          networks:
            - nnn
            - xxx
        bbb:
          ...
          networks:
            - nnn
            - xxx

      networks:
        nnn:
        xxx:
          external: true
      ```

## Tutorial 2.5 (ii) - Cron

- https://www.youtube.com/playlist?list=PL36CGZHZJqsWXjf4GeQBLUl7CK1dodKuC Episode 6
  - https://github.com/aschmelyun/laravel-scheduled-tasks-docker

### Normally in local Linux

`crontab –e` and paste https://laravel.com/docs/8.x/scheduling#running-the-scheduler

### In Docker php fpm alpine

Paste https://laravel.com/docs/8.x/scheduling#running-the-scheduler into a new `crontab` file then in `Dockerfile`: `COPY ./crontab /etc/crontabs/root`. Then run `docker-compose exec -d {The php fpm alpine service} crond -f`.

To stop the cron: Run `docker-compose exec {The php fpm alpine service} ps aux` to get process list, then run `docker-compose exec {The php fpm alpine service} kill {process number}`

### Via a cron docker service

`docker-compose.yml`
```
version: '3'

services:

  ...

  cron:
    build:
      context: .
      dockerfile: cron.dockerfile
    volumes:
      - ./:/var/www/html
```

`cron.dockerfile`
```
FROM php:7.4-fpm-alpine

RUN docker-php-ext-install pdo pdo_mysql

COPY crontab /etc/crontabs/root

CMD ["crond", "-f"]
```

`docker-compose up -d cron`

Or better: 

Change `/dev/null` to `/dev/stdout` in crontab file ( https://laravel.com/docs/8.x/scheduling#running-the-scheduler ), then run `docker-compose up cron`. This way you will see cron's output in the console.

## Tutorial 3 - To Production

- https://www.youtube.com/playlist?list=PL1LQwTE3lBhSnaL7j90AUJyvC9mFCKhZm Episode 3
  - https://github.com/tonysm/dockerforlaravel-app

- https://www.youtube.com/playlist?list=PL36CGZHZJqsWXjf4GeQBLUl7CK1dodKuC Episode 5
  - https://github.com/aschmelyun/docker-compose-laravel

### Differences in production

- Better not to map volumes, so the image should have everything it need within it (except some private things)
- Change build steps for dependencies
- Better to use https://hub.docker.com/_/php 's `$PHP_INI_DIR/php.ini-production`, which in turn needs OPcache for custom configs.
  - https://www.scalingphpbook.com/
  - Example OPcache: https://laravel-news.com/php-opcache-docker
  - OPcache config detailed guide: https://tideways.com/profiler/blog/fine-tune-your-opcache-configuration-to-avoid-caching-suprises
- Make custom nginx image
- Have built images pushed to DockerHub
- Stick with default ports, eg: use port 80 for http, don't use 8080, 8000, etc...
- More un-guessable passwords
- Setup server
  - Add a non-root user with sudo priviledges on Server `useradd laravel`, `usermod -aG sudo laravel`
  - Create the directory on server where our website will reside
  - Setup firewall to close off all unused ports
  - Upload files to server
    1. `rsync -avzh --exclude={"/node_modules/*","/vendor/*"} . {username}@{ip address}:/{directory}` (flags: a means all files, v means verbose, z means compressed while uploading, h means display any binaries in human-readable format) https://en.wikipedia.org/wiki/Rsync
    2. Via Repo
  - Have Docker setup on Server: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04

### Dockerfile explained

```
# Composer dependencies.
FROM composer AS composer-build

WORKDIR /var/www/html

COPY composer.json composer.lock /var/www/html/

RUN mkdir -p /var/www/html/database/{factories,seeds} \
  && composer install --no-dev --prefer-dist --no-scripts --no-autoloader --no-progress --ignore-platform-reqs
```
- `composer install` needs these files: `composer.json` & `composer.lock` and these folders: `database/factories/` & `database/seeds/` (because of https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/Illustrations/IMPROVE_docker_laravel_commit.pdf `composer.json#L39-L40`)
- `--no-scripts` means don't execute those: https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/Illustrations/IMPROVE_docker_laravel_commit.pdf `composer.json#L50-L61`
- Don't run autoload-dump neither. Only just download dependencies at this stage

```
# NPM dependencies.
FROM node:12 AS npm-build

WORKDIR /var/www/html

COPY package.json package-lock.json webpack.mix.js /var/www/html/
COPY resources /var/www/html/resources/
COPY public /var/www/html/public/

RUN npm ci
RUN npm run production
```
- `npm install` needs these files: `package.json` & `package-lock.json`. 
- Installing and compiling assets needs `webpack.mix.js`, `/resources/` & `/public/` (https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/Illustrations/IMPROVE_docker_laravel_commit.pdf `webpack.mix.js#L14-L15`)
- If you build the docker image now, the image will be empty inside. `.dockerignore` prevented eg `/vendor`, `/node_modules` etc to be copied from local directory into docker image, and we haven't put anything else into it yet. (The above 2 stages downloaded dependencies and compiled assets all in the local directory)

```
# Actual production image.
FROM php:7.4-fpm

WORKDIR /var/www/html

RUN apt-get update \
  && apt-get install --quiet --yes --no-install-recommends \
    libzip-dev \
    unzip \
  && docker-php-ext-install pcntl opcache zip pdo pdo_mysql \
  && pecl install -o -f redis-5.1.1 \
  && docker-php-ext-enable redis
```
- Get required php extensions (same as in local, except `opcache`)

```
# Use the default production configuration
RUN mv $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini

# Override with custom opcache settings
COPY ./docker/opcache.ini $PHP_INI_DIR/conf.d/
```
- Different server settings
  - Need `opcache`

```
COPY --from=composer /usr/bin/composer /usr/bin/composer

COPY --chown=www-data --from=composer-build /var/www/html/vendor/ /var/www/html/vendor/
COPY --chown=www-data --from=npm-build /var/www/html/public/ /var/www/html/public/
COPY --chown=www-data . /var/www/html

RUN composer dump -o && composer check-platform-reqs
```
- Get Composer, copy all the files into the image, then perform composer autoload-dump
- https://nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile

#### This is a multi-stage dockerfile

- https://askinglot.com/what-is-multistage-dockerfile
- https://docs.docker.com/develop/develop-images/multistage-build/

### Custom nginx Dockerfile explained

```
# NPM dependencies.
FROM node:12 AS npm-build

WORKDIR /var/www/html

COPY package.json package-lock.json webpack.mix.js /var/www/html/
COPY resources /var/www/html/resources/
COPY public /var/www/html/public/

RUN npm ci
RUN npm run production
```
- NPM dependencies needed here too, for eg: email

```
# Nginx production.
FROM nginx:1.17

COPY ./docker/nginx_template_prod.conf /etc/nginx/conf.d/default.conf
```
- configs for production
  - Only difference: production (which will use K8) have `fastcgi_pass 127.0.0.1:9000;` instead of `fastcgi_pass php:9000;` on local (in which `web` talks to `php` internally thru docker networks)

```
COPY --chown=www-data --from=npm-build /var/www/html/public/ /var/www/html/public/
```
- Needs the entry point (`public/index.php`)

```
COPY --chown=www-data . /var/www/html
```

https://github.com/atabegruslan/Others/blob/master/Virtual/docker.md#dockerize-nginx

### Debug docker build error:  

The logs would have a hash associated with each build step.  
So you can go into the container and run commands to debug at the container's state just before error happened: `docker rum --rm -it {hash} bash`

### Rest of the major changes:

https://github.com/Ruslan-Aliyev/laravel_dockerized/blob/master/Illustrations/to_production_commit.pdf

## Tutorial 4 - Multiple nodes

### Swarms

**Preliminaries**

- https://www.youtube.com/watch?v=bU2NNFJ-UXA <sup>Multiple VMs as nodes. Starting from Step 1 below</sup>
  - https://automationstepbystep.com/
  - https://www.youtube.com/playlist?list=PLhW3qG5bs-L99pQsZ74f-LC-tOEsBp2rK
  - https://rominirani.com/docker-swarm-tutorial-b67470cf8872
    - https://medium0.com/m/global-identity?redirectUrl=https%3A%2F%2Frominirani.com%2Fdocker-swarm-tutorial-b67470cf8872

- https://www.youtube.com/watch?v=3-7gZS4ePak <sup>1 machine as both master and worker. Starting from Step 4 below</sup>
  - https://takacsmark.com/docker-swarm-tutorial-for-beginners/

- https://betterprogramming.pub/how-to-differentiate-between-docker-images-containers-stacks-machine-nodes-and-swarms-fd5f7e34eb9f <sup>Terminologies</sup>

**Pre-requisites**

1. Docker 1.13 or higher
2. Install Docker Machine
  - https://docs.docker.com/machine/install-machine/#installing-machine-directly
  - https://docs.docker.com/get-started/swarm-deploy

**Steps**

1. Create 1 manager node and worker nodes: 
  ```
  docker-machine create --driver virtualbox manager1
  docker-machine create --driver virtualbox worker1
  docker-machine create --driver virtualbox worker2
  ```

2. Check machine created successfully
  ```
  docker-machine ls
  docker-machine ip manager1
  ```

3. SSH into docker machine: `docker-machine ssh manager1`

4. Initialize Docker Swarm: `docker swarm init --advertise-addr {MANAGER_IP}`. You will get the "join-token". Even simpler, `docker swarm init` will create a swarm, a master and a join-token. 

5. Let 1 node join the swarm as master by SSH into designated master node and run: `docker swarm {join-token} manager`. 
  - To see all nodes in its swarm: `docker node ls`
  - See swarm info: `docker info`, especially the "Swarm" section
  - See available command options: `docker swarm `

6. Let the other nodes join the swarm as workers by SSH into them and run: `docker swarm join --token {join-token} {ip address}:{port}`

7. Run containers on Docker Swarm. Run on master: `docker service create --replicas {number} -p 80:80 --name {service name} {image name}`. **Or** if you already have `docker-compose.yml`, then run `docker stack deploy -c docker-compose.yml {stack name}` (the swarm equivalent of `docker-compose`). To check status:
  - On master: `docker service ls`
  - On master: `docker service ps {service name}`
  - Check the service running on all nodes
  - Check on the browser by giving ip for all nodes
  - See the Stacks deployed to Swarm: `docker stack ls`
  - See the services of a Stack: `docker stack services ls`

8. Scale service up and down. On manager node: `docker service scale {service name}={another number}`. To inspect nodes, on manager node:
  - `docker node inspect {nodename}`
  - `docker node inspect self`

9. Shutdown node: `docker node update --availability drain {nodename}`

10. Update service: `docker service update --image {image name}:{version} {service name}`

11. Remove service: `docker service rm {service name}`

12. Leave the swarm
  ```
  docker-machine stop {machine name} # stop machine
  docker-machine rm machine name     # remove machine
  ```

### Kubernetes with KubeAdm (Plain Setup)

- https://www.youtube.com/watch?v=l7gC4SgW7DU&t=498s

### Kubernetes (Setup on Digital Ocean)

- https://www.youtube.com/playlist?list=PL1LQwTE3lBhSnaL7j90AUJyvC9mFCKhZm Episode 4+
  - https://github.com/tonysm/dockerforlaravel-app
  - https://github.com/tonysm/dockerforlaravel-k8s-files

## Tutorial 8 - CI

https://github.com/Ruslan-Aliyev/Docker_CI