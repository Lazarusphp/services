# Engurn Installer : Docker Compose

## What is Engurn 
Engurn Installer Pronounced Engine is a collection of scripts all installed in either a singular or a collective way by using either docker run or docker compose commands.

## Docker Compose setup.
The Following example code is based on prebuild Scripts which can be found under the nginx and php folders.

Image names are as follows itzzpadre:php-fpm and itzzpadre/nginx:php-fpm

Establish the version of the docker-compose file.


    version: "3.9"

<hr>

### Establishing the Volumes and Networks


    volumes:
    Hosting:
        external: true
    networks:
    web:
        external: true
    backend:
        external: true

these Snippets tell docker that the containers are to be used externally and can be shared between container stacks (outside of the docker compose file).

if you do not have the created networks or volumes use the following command in docker


    docker network create web
    docker network create backend
    docker volume create Hosting

if you already have a volume and networks created that you wish to use please simply rename the networks and volumes. 
<hr>
### Establishing the Services 

The Snippet below tells docker which containers you wish to run on instantiation of the docker-compose file

    services:
    web:
        image: itzzpadre/nginx:php-fpm
        hostname: ${domain}
        container_name: ${site_name}-web
        ports:
        - 80:80
        networks:
        - web
        - backend
        volumes:
        - Hosting:/var/www/
        - ./conf.d:/etc/nginx/conf.d
        restart: always
    php-fpm:
        image: itzzpadre/php-fpm
        networks:
        - backend
        container_name: "${site_name}-php"
        volumes:
        - Hosting:/var/www
        restart: always

### Nginx

the first service we will tell docker to run is the nginx service called ${site_name}-web with a hostname of ${domain} using the networks web and backend which we specified as external above on port 80

this also allows access to shared volumes which can will overwrite any content on the docker containers file system.

Hosting:/var/www allows the user to store and manage/edit the files locally without having to enter the container to do any form of editing.

./Conf:/etc/nginx/conf.d

this script i have put in place in order to hold all web config files in case of a container failure or rebuild, upon creationall config files will be added to thecorrect folder

### the php-fpm service
the second service which is require as part of the package, without it nginx will fail to load properly due to a misconfiguration with the php server 

like before the php fpm it will tell the compose file it needs to create a container called ${site_name}-php with hostname ${domain} using the network backend as this is the only network needed for the two containers to connect, this also allows isolation between containers.

In regards to volumes this must share the same volume as nginx in order for php to serve the server.

<hr>
Please note that ${site_name} and ${domain} are variables taken from an env file if you wish to just name them within the docker compose file simply replace ${domain} ${site_name} or add your own container name and domain.

### Creating a .env file
as part of the package (downloaded from git) the file comes with a .env.example file which needs to be renamed to .env

this file containes variables which are passed to the docker-compose file in this case site_name and domain

    domain= Your domain name
    site_name=Your site name

the domain althought not needed helps to give the server a hostname (I use it to link php applications together using a domain) the sitename just allows quick and easy naming convention if the site uses the same name.

In my own config i used a reverse proxy service called <a href='https://traefik.io'>Traefik</a> 
which is where the site_name and domain name became useful as i chose to use seperate containers per site.


## How to use and install containers with docker-compose

once you are happy with your setup for the server simply enter or copy and paste the following command 

    docker-compose up -d 

adding --force-recreate will force the container to recreate if it encounters errors.

add --remove-orphans will remove any text or flags and will boot up with the amended changes. as docker compose typically will recreate from cache.

## Additional Flags
Please see below the following flags that can be utilised with this container stack

### workdir
by default the working directory is set to /vaw/www if you wish to change the working directory use -workdir this is not the same as the config file directory this is simply the directroy you will start in when you boot into the container.

### depends_on
this is added into the current stack is used to prevent the services booting up if it cannot find the required container inside the stack.

    -depends_on:
        - php-fpm

this will prevent the container from booting up.

### Enviromental variables
if you wish to pass optional variables to the server let use credentials as an example you can simply add the following.

    -enviroments
        - USER="Username"
        - PASS="Password"

if you then call echo $USER or echo $PASS the value entered would be displayed as this has now been saved within the container.

<hr>


<!-- Explain about labels -->
### the following code snippet before is used with the reverse proxy package called traefik <a href="https://doc.traefik.io/traefik/providers/docker/">Traefik Documentation</a> for more information on labels <a href='https://docs.docker.com/reference/cli/docker/container/run/#label'>Docker documentation</a>

if you wish to use traefik reverse proxy please attach the snippet below to your webserver section of the docker compose

this is used to forward and track the ports to the correct container
```
    labels:
    - "traefik.enable=true"
    - "traefik.http.routers.${site_name}_web.rule=Host(`${domain}`)"
    - "traefik.http.routers.${site_name}_web.entrypoints=web"
    - "traefik.http.routers.${site_name}_websecure.rule=Host(`${domain}`)"
    - "traefik.http.routers.${site_name}_websecure.entrypoints=websecure"
    - "traefik.http.routers.${site_name}_websecure.tls.certresolver=myresolver"
    - "traefik.docker.network=web"
```
without the use of a reverse proxy you can only forward port 80 to one container and would need to rely on forwarding different ports to port 80 per container.

this is obviously dependant on your server setup and how many ips, servers or subnets you have at your dispsal
