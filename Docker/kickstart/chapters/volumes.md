---
---

# Docker Volumes

In the [Docker Images](./images.md) chapter you saw that when a container is removed, its writable layer — and any data written to it — is deleted along with it. For databases, uploaded files, or any state that must survive beyond a single container lifecycle, you need a different mechanism: **Docker Volumes**.

> **Tasks:**
>
> - [Task 1: Understanding Docker Volumes](#task-1-understanding-docker-volumes)

## Task 1: Understanding Docker Volumes

[Docker volumes](https://docs.docker.com/engine/admin/volumes/volumes/) are directories on the host file system that are not managed by the storage driver. Since they are not managed by the storage driver they offer a couple of important benefits.

- **Performance**: Because the storage driver has to create the logical filesystem in the container from potentially many directories on the local host, accessing data can be slow. Especially if there is a lot of write activity to that container. In fact you should try and minimize the amount of writes that happen to the container's filesystem, and instead direct those writes to a volume

- **Persistence**: Volumes are not removed when the container is deleted. They exist until explicitly removed. This means data written to a volume can be reused by other containers.

Volumes can be anonymous or named. Anonymous volumes have no way to be explicitly referenced. They are almost exclusively used for performance reasons as you cannot persist data effectively with anonymous volumes. Named volumes can be explicitly referenced so they can be used to persist data and increase performance.

The next sections will cover both anonymous and named volumes.

> Special Note: These next sections were adapted from [Arun Gupta's](https://twitter.com/arungupta) excellent [tutorial](http://blog.arungupta.me/docker-mysql-persistence/) on persisting data with Docker databases.

### Anonymous Volumes

Take a look at the MariaDB [Dockerfile](https://github.com/MariaDB/mariadb-docker/blob/master/11.4/Dockerfile) you will find the following line:

```dockerfile
VOLUME /var/lib/mysql
```

This line sets up an anonymous volume in order to increase database performance by avoiding sending a bunch of writes through the Docker storage driver.

> **Note:** An anonymous volume is a volume that hasn't been explicitly named. This means that it's extremely difficult to use the volume later with a new container. Named volumes solve that problem, and will be covered later in this section.

1. Start a MariaDB container

   ```console
   $ docker container run --name mariadb -e MARIADB_USER=dbuser -e MARIADB_PASSWORD=dbpass -e MARIADB_DATABASE=sample -e MARIADB_ROOT_PASSWORD=supersecret -d mariadb:11
   acf185dc16e274b2f332266a1bfc6d1df7d7b4f780e6a7ec6716b40cafa5b3c3
   ```

   When we start the container the anonymous volume is created:

2. Use `docker container inspect` to view the details of the anonymous volume

   {% raw %}

   ```console
   $ docker container inspect -f "in the {{.Name}} container {{(index .Mounts 0).Destination}} is mapped to {{(index .Mounts 0).Source}}" mariadb
   ```

   {% endraw %}

   This command will return: `in the /mariadb container /var/lib/mysql is mapped to /var/lib/docker/volumes/cd79b3301df29d13a068d624467d6080354b81e34d794b615e6e93dd61f89628/_data`

   As mentioned anonymous volumes will not persist data between containers, they are almost always used to increase performance.

3. Shell into your running MariaDB container and log into MariaDB

   ```console
   $ docker container exec --tty --interactive mariadb bash

   root@132f4b3ec0dc:/# mariadb --user=dbuser --password=dbpass
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 3
   Server version: 11.4.2-MariaDB-ubu2404 mariadb.org binary distribution

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   ```

4. Create a new table

   ```
   MariaDB [(none)]> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | sample             |
   +--------------------+
   2 rows in set (0.00 sec)

   MariaDB [(none)]> use sample;
   Database changed

   MariaDB [sample]> show tables;
   Empty set (0.00 sec)

   MariaDB [sample]> create table user(name varchar(50));
   Query OK, 0 rows affected (0.01 sec)

   MariaDB [sample]> show tables;
   +------------------+
   | Tables_in_sample |
   +------------------+
   | user             |
   +------------------+
   1 row in set (0.00 sec)
   ```

5. Exit MariaDB and the MariaDB container.

   ```
   MariaDB [sample]> exit
   Bye

   root@132f4b3ec0dc:/# exit
   exit
   ```

6. Stop the container and restart it

   ```console
   $ docker container stop mariadb
   mariadb

   $ docker container start mariadb
   mariadb
   ```

7. Shell back into the running container and log into MariaDB

   ```console
   $ docker container exec --interactive --tty mariadb bash

   root@132f4b3ec0dc:/# mariadb --user=dbuser --password=dbpass
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 3
   Server version: 11.4.2-MariaDB-ubu2404 mariadb.org binary distribution

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   ```

8. Ensure the table created previously still exists

   ```
   MariaDB [(none)]> use sample;
   Reading table information for completion of table and column names
   You can turn off this feature to get a quicker startup with -A

   Database changed

   MariaDB [sample]> show tables;
   +------------------+
   | Tables_in_sample |
   +------------------+
   | user             |
   +------------------+
   1 row in set (0.00 sec)
   ```

9. Exit MariaDB and the MariaDB container.

   ```
   MariaDB [sample]> exit
   Bye

   root@132f4b3ec0dc:/# exit
   exit
   ```

   The table persisted across container restarts, which is to be expected. In fact, it would have done this whether or not we had actually used a volume as shown in the previous section.

10. Let's look at the volume again

    {% raw %}

    ```console
    $ docker container inspect -f "in the {{.Name}} container {{(index .Mounts 0).Destination}} is mapped to {{(index .Mounts 0).Source}}" mariadb
    in the /mariadb container /var/lib/mysql is mapped to /var/lib/docker/volumes/cd79b3301df29d13a068d624467d6080354b81e34d794b615e6e93dd61f89628/_data
    ```

    {% endraw %}

    We do see the volume was not affected by the container restart either.

    Where people often get confused is in expecting that the anonymous volume can be used to persist data BETWEEN containers.

    To examine that delete the old container, create a new one with the same command, and check to see if the table exists.

11. Remove the current MariaDB container

    ```console
    $ docker container rm --force mariadb
    mariadb
    ```

12. Start a new container with the same command that was used before

    ```console
    $ docker container run --name mariadb -e MARIADB_USER=dbuser -e MARIADB_PASSWORD=dbpass -e MARIADB_DATABASE=sample -e MARIADB_ROOT_PASSWORD=supersecret -d mariadb:11
    eb15eb4ecd26d7814a8da3bb27cee1a23304fab1961358dd904db37c061d3798
    ```

13. List out the volume details for the new container

    {% raw %}

    ```console
    $ docker container inspect -f "in the {{.Name}} container {{(index .Mounts 0).Destination}} is mapped to {{(index .Mounts 0).Source}}" mariadb
    in the /mariadb container /var/lib/mysql is mapped to /var/lib/docker/volumes/e0ffdc6b4e0cfc6e795b83cece06b5b807e6af1b52c9d0b787e38a48e159404a/_data
    ```

    {% endraw %}

    Notice this directory is different than before.

14. Shell back into the running container and log into MariaDB

    ```console
    $ docker container exec --interactive --tty mariadb bash

    root@132f4b3ec0dc:/# mariadb --user=dbuser --password=dbpass
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 3
    Server version: 11.4.2-MariaDB-ubu2404 mariadb.org binary distribution

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    ```

15. Check to see if table created previously still exists

    ```
    MariaDB [(none)]> use sample;
    Database changed

    MariaDB [sample]> show tables;
    Empty set (0.00 sec)
    ```

16. Exit MariaDB and the MariaDB container.

    ```
    MariaDB [sample]> exit
    Bye

    root@132f4b3ec0dc:/# exit
    exit
    ```

17. Remove the container

    ```console
    $ docker container rm --force mariadb
    mariadb
    ```

So while a volume was used to store the new table in the original container, because it wasn't a named volume the data could not be persisted between containers.

To achieve persistence a named volume should be used.

### Named Volumes

A named volume (as the name implies) is a volume that's been explicitly named and can easily be referenced.

A named volume can be created on the command line, in a docker-compose file, and when you start a new container. They [CANNOT be created as part of the image's dockerfile](https://github.com/moby/moby/issues/30647).

1. Start a MariaDB container with a named volume (`mydbdata`)

   ```console
   $ docker container run --name mariadb \
   -e MARIADB_USER=dbuser \
   -e MARIADB_PASSWORD=dbpass \
   -e MARIADB_DATABASE=sample \
   -e MARIADB_ROOT_PASSWORD=supersecret \
   --detach \
   --mount type=volume,source=mydbdata,target=/var/lib/mysql \
   mariadb:11
   ```

   Because the newly created volume is empty, Docker will copy over whatever existed in the container at `/var/lib/mysql` when the container starts.

   Docker volumes are primitives just like images and containers. As such, they can be listed and removed in the same way.

2. List the volumes on the Docker host

   ```console
   $ docker volume ls
   DRIVER              VOLUME NAME
   local               55c322b9c4a644a5284ccb5e4d7b6b466a0534e26d57c9ef4221637d39cf9a88
   local               cc44059d23e0a914d4390ea860fd35b2acdaa480e83c025fb381da187b652a66
   local               e0ffdc6b4e0cfc6e795b83cece06b5b807e6af1b52c9d0b787e38a48e159404a
   local               mydbdata
   ```

3. Inspect the volume

   ```console
   $ docker volume inspect mydbdata
   [
       {
           "CreatedAt": "2017-10-13T19:55:10Z",
           "Driver": "local",
           "Labels": null,
           "Mountpoint": "/var/lib/docker/volumes/mydbdata/_data",
           "Name": "mydbdata",
           "Options": {},
           "Scope": "local"
       }
   ]
   ```

   Any data written to `/var/lib/mysql` in the container will be rerouted to `/var/lib/docker/volumes/mydbdata/_data` instead.

4. Shell into your running MariaDB container and log into MariaDB

   ```console
   $ docker container exec --tty --interactive mariadb bash

   root@132f4b3ec0dc:/# mariadb --user=dbuser --password=dbpass
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 3
   Server version: 11.4.2-MariaDB-ubu2404 mariadb.org binary distribution

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   ```

5. Create a new table

   ```
   MariaDB [(none)]> use sample;
   Database changed

   MariaDB [sample]> show tables;
   Empty set (0.00 sec)

   MariaDB [sample]> create table user(name varchar(50));
   Query OK, 0 rows affected (0.01 sec)

   MariaDB [sample]> show tables;
   +------------------+
   | Tables_in_sample |
   +------------------+
   | user             |
   +------------------+
   1 row in set (0.00 sec)
   ```

6. Exit MariaDB and the MariaDB container.

   ```
   MariaDB [sample]> exit
   Bye

   root@132f4b3ec0dc:/# exit
   exit
   ```

7. Remove the MariaDB container

   ```console
   $ docker container rm --force mariadb
   ```

   Because MariaDB was writing out to a named volume, we can start a new container with the same data.

   When the container starts it will not overwrite existing data in a volume. So the data created in the previous steps will be left intact and mounted into the new container.

8. Start a new MariaDB container

   ```console
   $ docker container run --name new_mariadb \
   -e MARIADB_USER=dbuser \
   -e MARIADB_PASSWORD=dbpass \
   -e MARIADB_DATABASE=sample \
   -e MARIADB_ROOT_PASSWORD=supersecret \
   --detach \
   --mount type=volume,source=mydbdata,target=/var/lib/mysql \
   mariadb:11
   ```

9. Shell into your running MariaDB container and log into MariaDB

   ```console
   $ docker container exec --tty --interactive new_mariadb bash

   root@132f4b3ec0dc:/# mariadb --user=dbuser --password=dbpass
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 3
   Server version: 11.4.2-MariaDB-ubu2404 mariadb.org binary distribution

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   ```

10. Check to see if the previously created table exists in your new container.

    ```
    MariaDB [(none)]> use sample;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Database changed

    MariaDB [sample]> show tables;
    +------------------+
    | Tables_in_sample |
    +------------------+
    | user             |
    +------------------+
    1 row in set (0.00 sec)
    ```

    The data will exist until the volume is explicitly deleted.

11. Exit MariaDB and the MariaDB container.

    ```
    MariaDB [sample]> exit
    Bye

    root@132f4b3ec0dc:/# exit
    exit
    ```

12. Remove the new MariaDB container and volume

    ```console
    $ docker container rm --force new_mariadb
    new_mariadb

    $ docker volume rm mydbdata
    mydbdata
    ```

    If a new container was started with the previous command, it would create a new empty volume.

## Next Steps

For the next step in the tutorial, head over to [Webapps with Docker - Part Two](./webapps-part2.md)
