---
layout: post
title: "Dockerising a Node.js/Express app"
image: ""
published: false
---

# Intro

# Designing and creating the database

# Building the API

# Dockerising the application

Our application consists of two separate components which are currently communicating over the localhost network:

1. Our Node.js/Express application, running on a Node.js server - we have been accessing this through our browser on localhost port 3000.
2. Our Postgres database, running on a postgres database server - our Express app connects to this on localhost port 5432.

To Dockerise our application, we will create 2 separate Docker containers, one for each of these components. The easiest way to do this is using Docker Compose.

First, lets look at our Express app.

```dockerfile
# Specify a base image
FROM node:alpine

WORKDIR "/app"
COPY ./package.json ./
COPY ./package-lock.json ./
# Install some dependencies
RUN npm install
COPY ./ ./

# Default command
CMD ["npm", "run", "dev"]
```

TODO: talk about this.

Now for our Database. We can use the existing [Postgres Image](https://hub.docker.com/_/postgres) for this. Our Express app needs to be able to talk to the database, and specifying them both as services in our docker-compose file will allow it to do so.

We add a Postgres service in our `docker-compose.yml`, using `postgres:10` as the image from which the Postgres container will be created. Now lets try and run this and see what happens. 

`docker-compose up --build`

This will create separate containers for our `api` and `postgres` services, and run them on the same network. The build flag specfieis that any images with a build context specified (TODO: correct?) should be rebuilt.

```yml
version: "3"
services:
  api:
    build: 
      context: .
    ports: 
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  postgres:
    image: "postgres:10"
    volumes:
      - "~/programming/databases/ecommerce-demo-data:/var/lib/postgresql/data"

```

We will see that we get an error, the application is unable to connect to the database. This is because our `ecommerce_demo` database does not exist inside of our postgres container.

We can solve this using volumes. We will map the Postgres data folder inside of our container to a folder on our local machine.

Commands:

`docker-compose run postgres`

`docker ps` to get ID

`docker inspect -f '{{ json .Mounts }}' [containerId] | python -m json.tool` will allow us to inspect the volumes for this container.

`docker cp ./ecommerce_demo.sql [containerId]:/home/ecommerce_db.sql`  copies db dump to container

`docker exec -it [containerId] bash` starts shell inside of container

`su - postgres` 
`createdb ecommerce_demo`
`pg_restore -d ecommerce_demo /home/ecommerce_demo.sql`

Now that we have imported our database inside of theis container, the databse will be persisted in our mapped data folder on our local machine. This will ensure the each time we run `docker-compose up` and a new `postgres` container is created, it will have access to our ecommerce_demo databse, in the state it was left last time we stopped our `postgres` service.


