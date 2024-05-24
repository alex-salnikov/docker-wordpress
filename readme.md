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
docker-compose logs -f                      # logs
```


# MySQL

## Healthcheck
Implement a "healthcheck", so that dependent containers wait for DB-container to become ready, example:
```yml
  db:
    healthcheck:
      #test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      #test: ["CMD", 'mysqladmin', 'ping', '-h', 'db', '-u', 'root', '-p$$MYSQL_ROOT_PASSWORD' ]
      test: ["CMD-SHELL", 'mysqladmin ping -h db -u root -p$$MYSQL_ROOT_PASSWORD || exit 1' ]

  wp:
    depends_on:
      db:
        condition: service_healthy
```

Use following command to see healthcheck results:
```sh
docker inspect --format "{{json .State.Health }}" $(docker-compose ps -q db) | jq
# {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Log": [
#     {
#       "Start": ..
#       "End": ..
#       "ExitCode": 0,
#       "Output": "mysqladmin: [Warning] Using a password on the command line interface can be insecure.\nmysqld is alive\n"
#     },
```

References
- https://github.com/compose-spec/compose-spec/blob/master/spec.md#healthcheck
- https://github.com/compose-spec/compose-spec/blob/master/spec.md#long-syntax-1
- https://stackoverflow.com/questions/42567475/docker-compose-check-if-mysql-connection-is-ready
- https://docs.docker.com/compose/compose-file/05-services/#healthcheck
- https://stackoverflow.com/questions/70263411/docker-compose-how-to-wait-for-a-file-to-appear-or-a-folder-to-appear-before-a
- https://stackoverflow.com/questions/42737957/how-to-view-docker-compose-healthcheck-logs


# PHPMyAdmin
use "root" and [`DB_ROOT_PASSWORD`](./.env) to login.


## Troubleshoot
Error
> mysqli::real_connect(): (HY000/2002): Connection refused
> mysqli::real_connect(): php_network_getaddresses: getaddrinfo failed: Name or service not known

Implement [healthcheck](#healthcheck), because takes some time (about a minute) for the DB to start.

You can try to ping DB-container from inside PMA-container:
```sh
docker-compose ps                       # get name of PMA-container, e.g. 'wordpress_pma'
docker exec -it wordpress_pma bash      # start terminal-session from inside PMA-container

# while inside container ..
apt update && apt install -y iputils-ping   # install 'ping'
ping db                                     # try to ping DB-container
```


# TODO
- [x] Implement "healthcheck" to wait for DB-container to become ready (see [healthcheck](#healthcheck))
- check that **mariadb** container is working


# References
- https://wordpress.org/support/topic/wordpress-site-with-docker-behind-apache-reverse-proxy-keeps-redirecting-from-do/
- https://stackoverflow.com/questions/65343396/docker-compose-phpmyadmin-php-network-getaddresses-getaddrinfo-failed-temporar
- https://geshan.com.np/blog/2023/08/docker-compose-environment-variables/
