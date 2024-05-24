# Wordpress with Docker
Components
- [mysql](https://hub.docker.com/_/mysql) / [mariadb](https://hub.docker.com/_/mariadb) (alternative)
- [wordpress](https://hub.docker.com/_/wordpress), [wp-cli](https://hub.docker.com/_/wordpress) (optional)
- [PHPMyAdmin](https://hub.docker.com/_/phpmyadmin) (optional)


## Settings
- [.env](./.env) - change examplary user-names/passwords, external WP-host
- [docker-compose.yml](./docker-compose.yml) - check external ports


## Operation
```sh
docker-compose up -d                        # start
docker-compose --env-file my.env up -d      # specify alternative .env file
docker-compose ps                           # check running containers
docker-compose pull                         # pull updated versions from dockerhub
docker-compose down                         # stop
```


# PHPMyAdmin
use "root" and [`DB_ROOT_PASSWORD`](./.env) to login.


## Troubleshoot
Error
> mysqli::real_connect(): (HY000/2002): Connection refused
> mysqli::real_connect(): php_network_getaddresses: getaddrinfo failed: Name or service not known

It might take some time (couple of minutes) for the DB to start. You can try to ping DB-container from inside PMA-container:
```sh
docker-compose ps                       # get name of PMA-container, e.g. 'wordpress_pma'
docker exec -it wordpress_pma bash      # start terminal-session from inside PMA-container

# while inside container ..
apt update && apt install -y iputils-ping   # install 'ping'
ping db                                     # try to ping DB-container
```


# TODO
- Implement "healthcheck" to wait for DB-container to become ready, see references:
  - reference: https://stackoverflow.com/questions/70263411/docker-compose-how-to-wait-for-a-file-to-appear-or-a-folder-to-appear-before-a
- check that **mariadb** container is working


# References
- https://wordpress.org/support/topic/wordpress-site-with-docker-behind-apache-reverse-proxy-keeps-redirecting-from-do/
- https://stackoverflow.com/questions/65343396/docker-compose-phpmyadmin-php-network-getaddresses-getaddrinfo-failed-temporar
- https://geshan.com.np/blog/2023/08/docker-compose-environment-variables/
