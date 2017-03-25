This repository makes easy to build endpoints automatically using [api blueprint](https://apiblueprint.org/) documentation. 

To create endpoints automatically is using [apimock](https://github.com/localmed/api-mock) library.

## How-to build de environment

You can build a local environment using virtualbox, you have a vagrant file in the repository ready to up a new server. This vagrant file is using ansible to provision the server.

Provision will install [nginx](https://www.nginx.com/) and [docker](https://www.docker.com).

Nginx is used to facilitate the usage of the mocks, redirecting port 300x to port 80. And docker is used to create a new container for each documentation file and make endpoints available.

A new container for each documentation file is needed because apimock only accepts one documentation file for process.

## How-to start the endpoints

To start using the endpoints you need to install docker and nginx in a server (modify inventory file adding the ip of the server).

```
$ ansible-playbook playbook.yml -i inventory
```

Enter with ssh to new server and execute:

```
$ cd /var/app/app && docker-compose up
```

When finish you will have n docker containers with the endpoints corresponding with your documentation files accessible with http://example1.api.dev/message.


## How-to create new API mocks:

To create endpoints automatically you need to follow four easy steps:


### Add a new documentation file 

Create new documentation file using api blueprint format and save it to folder src/doc.


### Make this file available to automatically create the endpoints

To make this documentation file available you must add a new service in docker-compose file: src/app/docker-compose.yml

You must add a new service adding the following lines under services. Modify the port of host to avoid collisions. In this example the port of the host will be 3001, that port will redirect to port 3333 on the container.

```
example2:
    build: ./node
    command: api-mock /var/app/doc/example2.md --port 3333
    ports:
      - "3001:3333"
    volumes:
      - /var/app/doc:/var/app/doc
```
Adding this service you will have new endpoint. You can call it using port http://0.0.0.0:3001/{action}


### Facilitate the usage of the endpoint

To facilitate the usage of the endpoints you can add a new virtualhost to nginx to map port 3001 to port 80. To do it you must edit nginx provisioning role to add another virtualhost to the template.

Edit file provisioning/nginx/templates/virtualhosts.j2 and add another server:

```
server {
    listen 80;

    server_name example2.api.dev;

    location / {
        proxy_pass http://0.0.0.0:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

The port of proxy_pass must be the same of docker-compose new service.

Now you can call the new endpoint using http://example2.api.dev/{action}

### Reload endpoints with new files

To reload docker containers with new endpoits you must stop containers, restart nginx (if necessary) and start containers again.


