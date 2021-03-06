# Harbor upgrade and database migration guide

When upgrading your existing Habor instance to a newer version, you may need to migrate the data in your database. Refer to [change log](../migration/changelog.md) to find out whether there is any change in the database. If there is, you should go through the database migration process. Since the migration may alter the database schema, you should **always** back up your data before any migration. 

*If your install Harbor for the first time, or the database version is the same as that of the lastest version, you do not need any database migration.*


**NOTE:** You must backup your data before any data migration.

### Upgrading Harbor and migrating data

1. Log in to the machine that Harbor runs on, stop and remove existing Harbor service if it is still running:

    ``` 
    cd make/
    docker-compose down
    ```

2.  Back up Harbor's current source code so that you can roll back to the current version when it is necessary.
    ```sh
    cd ../..
    mv harbor /tmp/harbor
    ```

3. Get the lastest source code from Github:
    ```sh
    git clone https://github.com/vmware/harbor
    ```
 
4. Before upgrading Harbor, perform database migration first.  The migration tool is delivered as a docker image, so you should pull the image from docker hub:

    ```
    docker pull vmware/harbor-db-migrator
    ```

5. Back up database to a directory such as `/path/to/backup`. You need to create the directory if it does not exist.  Also, note that the username and password to access the db are provided via environment variable "DB_USR" and "DB_PWD"

    ```
    docker run -ti --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql -v /path/to/backup:/harbor-migration/backup vmware/harbor-db-migrator backup
    ```

6.  Upgrade database schema and migrate data:

    ```
    docker run -ti --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql vmware/harbor-db-migrator up head
    ```

7. Change to `make/` directory, configure Harbor by modifying the file `harbor.cfg`, you may need to refer to the configuration files you've backed up during step 2. Refer to [Installation & Configuration Guide ](../docs/installation_guide.md) for more info.


8. Under the directory `make/`, run the `./prepare` script to generate necessary config files.
 
9. Rebuild Harbor and restart the registry service

    ```
    docker-compose up --build -d
    ```

### Roll back from an upgrade
For any reason, if you want to roll back to the previous version of Harbor, follow the below steps:

1. Stop and remove the current Harbor service if it is still running.

    ``` 
    cd make/
    docker-compose down
    ```
2. Restore database from backup file in `/path/to/backup` .

    ```
    docker run -ti --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql -v /path/to/backup:/harbor-migration/backup vmware/harbor-db-migrator restore
    ```

3. Remove current source code of Harbor.
    ``` 
    rm -rf harbor
    ```

4. Restore the source code of an older version of Harbor. 
    ```sh
    mv /tmp/harbor harbor
    ```

5. Restart Harbor service using the previous configuration.
    ```sh
    cd make/
    docker-compose up --build -d
    ```
    
### Migration tool reference
- Use `help` command to show instructions of the migration tool:

    ```docker run --rm -e DB_USR=root -e DB_PWD=xxxx vmware/harbor-db-migrator help```
    
- Use `test` command to test mysql connection:

    ```docker run --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql vmware/harbor-db-migrator test```

