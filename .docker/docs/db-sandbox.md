# DB sandbox mode

In certain cases you may want to have a DB sandbox and mess around with the DB, then roll it back.
Normally this can be achieved by backing up the DB and restoring it from the created DB dump.
With large databases (over 500MB) this operation can take a considerable amount of time (10+ minutes).

With Docker we can sandbox the DB by creating a reusable docker image from the DB container.
Spinning up such image takes less time than re-importing the DB with MySQL.

For example, a 2.6GB DB dump file takes 14 minutes to import with MySQL. (Re)launching a docker container from an image with that same imported dump takes only 2 minutes. **Your results may obviously vary.**

This is an advanced used case, so make sure you have a backup of the database before proceeding.
Follow instructions below to get started.

## Steps

1. Create a DB dump
2. Stop and remove running containers
    
    `docker-compose stop && docker-compose rm --force`

3. Comment out the `/var/lib/mysql` volume definition for the **db** service in `docker-compose.yml`
4. Restart containers

    `docker-compose up -d`

5. Import the DB dump you created in step 1.
6. Stop and commit the **db** service container (this will turn the container into a reusable docker image)
    
    `docker stop $(docker-compose ps -q db) && docker commit $(docker-compose ps -q db) <tag>`

Replace `<tag>` with any meaningful tag you'd like to use for the image. E.g. `db_backup` or `dbdata/myproject:snapshot1`

7. Edit the image definition for the *db* service in `docker-compose.yml` and replace it with the selected tag. E.g.:

    `image: dbdata/myproject:snapshot1`

8. Restart containers

    `docker-compose up -d`

You will now have a sandboxed DB container, which defaults to the DB snapshot you created in step 1 every time the db container is restarted.

## Precaution

It's not advisable to commit the container more than once in a row. Every docker image holds all parent images inside it. Thus, **with every commit the size of the resulting image will increase by full size of the DB, not its delta.**

## Disabling the sandbox mode

You will need a DB dump to revert to.
Either use the one created before enabling the sandbox mode or take another one while the **db** service contained is still up.

1. Stop and remove running containers
    
    `docker-compose stop && docker-compose rm --force`

2. Revert the changes done to the **db** service in `docker-compose.yml`
3. Restart containers

    `docker-compose up -d`

5. Import the DB dump.

Now the **db** service container is using a persistent data storage volume (`/var/lib/mysql`).