---
title: "Dockerizing My Dev Workflow"
date: 2016-07-10T16:37:03+03:00
draft: false
---

I’ve always just read about docker, for almost a year now, and recently decided to move all my development workflow to docker. I’ll briefly write about how I did this for two of the recent projects I’ve been working on, a mean stack application, and a Laravel-Postgres application.

I’m set up on a mac, and since docker is not natively supported on mac and Windows unless you have Virtual Box installed, I joined the [docker beta program](https://beta.docker.com/).

## Googling around about docker
I’m more of a hands on, learn on the job type of person, so I just went directly into using docker. I’d say though that after googling around, the most helpful resources I found were The [official docker documentation](https://docs.docker.com/get-started/), [Dan Wahlin’s Docker Magazine on Flipboard](https://flipboard.com/@dwahlin/the-docker-%26-kubernetes-magazine-vp93fvnrz) and [Nigel Pulton’s Docker Deep Dive course](https://www.pluralsight.com/courses/docker-deep-dive).

## Dockerizing the MEAN app.
The app is built on Mongo Express Angular 1.x and Node. From what I had gathered, I’d only need two docker containers. One for the App, and one for the mongo database.

#### Dockerfile
```dockerfile

FROM node:4.4.2

# Working Directory
WORKDIR /opt/app

# Add project files to WORKDIR
ADD . /opt/app

# Install the dependencies
RUN npm install -g gulp bower
RUN rm -Rf /opt/node_modules
RUN npm install
RUN mv node_modules/ /opt/
RUN gulp

# Expose the port
EXPOSE 5555

## Run app
CMD npm start
```


`npm install` can be really slow, and the folder gets deleted when the volume is mounted, so a workaround is to install node dependencies globally. That’s what the `RUN mv node_modules/ /opt/` does.

The difference with RUN and CMD is that RUN occurs during the build process of the container, while CMD occurs after the container has been built. The distinction is particularly important because if a container depends on another, some resources are usually not available during the build phase of either container, for instance, the links’ host names.

#### docker-compose.yml
```yml
version: '2'
services:

  app:
    build: .
    ports:
      - "5555:5555"
    volumes:
      - .:/opt/app
    links:
      - mongodb
    depends_on:
      - mongodb

  mongodb:
    image: mongo:3.2
    ports:
      - "27017:27017"
```


The docker-compose file is used to set up multiple containers and connect them together. You could also use it for just one container. This docker-compose defines two containers, one for the app, and the other for the MongoDB.

What to note here is that when we run `docker-compose up`, a network based on the name of my project is created. The `links` attribute of the app creates a host with that particular name (mongodb) within the network. So, if you want the MongoDB URL, you’d access it this way: `mongodb://mongodb:27017/your-db-name`.

That’s all the changes I needed to make and my Mean App was all dockerized. Running `docker-compose up`, set up everything, and my app was running on the expected port. There’s probably more I could do, but I’m just getting started.

## Dockerizing the Laravel-Postgres app
After doing the mean app, I thought the Laravel App would be straight forward (Probably a copy pasting kind of thing), It wasn’t.

Laravel has some requirements that the default PHP installation does not come with, Like the PDO for the particular database you want to use. After fiddling around with trial and errors, this is what I came up with.

#### Dockerfile
```dockerfile
FROM php:7

# Work Dir
WORKDIR /opt/app
ADD . /opt/app

# Install Dependencies
RUN apt-get update && apt-get install -y \
      libpq-dev \
    && docker-php-ext-install pdo_pgsql

RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer
RUN chmod +x /usr/local/bin/composer

CMD composer install

# Expose the port
EXPOSE 8000

# Run the app
CMD php artisan serve --host=0.0.0.0 --port=8000
```

Going through the [official docker container image for php](https://hub.docker.com/_/php/), it’s mention how to install and configure php extensions with `docker-php-ext-install` and `docker-php-ext-enable`.

The extensions are however compiled when downloaded, so we have to install their corresponding compile/helper libraries. In this case, since I’m installing the `pdo_pgsql` PHP extension, I have to also install `libpq-dev` that will assist in it’s compilation.

Notice that I use `CMD composer install` instead of `RUN composer install`. This is just to clarify that composer install is not part of the build task for the docker container. The reason for this is that I had a php artisan migrate as part of the scripts to run after composer install in my `composer.json` file.
```json
"scripts" : {
  "post-install-cmd": [
      "php artisan clear-compiled",
      "php artisan optimize",
      "php artisan migrate --force"
    ]
}
```


This task requires that we have a connection to the database. I mentioned this above, but just to iterate: During the build task, a docker container does not have access to the network created by docker-compose. As a result, during the build phase, the laravel app will not have access to the database host (postgres) host I have specified in the `docker-compose` file below.

#### docker-compose.yaml
```yaml
version: '2'
services:

  app:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/opt/app
    links:
      - database
    depends_on:
      - database

  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_USER: homestead
      POSTGRES_DB: homestead
    ports:
      - 5432
```

The other minor change I had to make was in my `.env` database host to

### .env
```
DB_HOST=database
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```


Notice the hostname `(DB_HOST)` is the same as that on my docker-compose postgres container name.

That’s all. Running my setup was as simple as `docker-compose up`.

I later found out though, that there’s a [Laravel Homestead Docker implementation](http://laradock.io/).

I’ll go back to digging deeper into docker now, there’s more to my few lines of configuration on the docker-compose file, and I just had a brief look at docker swarm, from the recently concluded dockercon.
