# WordpressToDocker

This document will be the guide on how to transfer a WordPress website to a Docker container.

## What we need
We will need a working version of **Docker** and **Docker Desktop**

The data from the site you want to transfer is also needed.
The data should be in the form of the *wp-content* folder.
You also need the *database* file. 
You can get it with WordPress Local > "Database" tab > Open AdminNeo and get the .sql from here

You can access it by going in the files of your already existing website.

**If you only have the updraft backups**:
You have to use **WordPress Local** to load the data into a site and get the folder.

## Checking the prerequisite and creation of the container
You first have to launch **Docker Desktop**, you can check with `docker ps`

Then you have to get the docker-compose.yml file from this git.

You can then launch the container with `docker compose up -d`
You may run into some issue with the port being already used, you just have to change the number of the problematic port.
Once it is executed, you can run `docker compose ps` . You should see 4 lines with the status up or running.

Wait up to 30 seconds and you can then go to *http://localhost:8080* to check if there is a WordPress installation interface.

You can check if the content is here by running `docker exec -it wp_site ls /var/www/html/wp-content` which should show you some folders. 

## Loading the database and giving you access
You can then copy the database into the site with `docker cp "path\to\backup.sql" wp_db:/backup.sql`
And load it with `docker exec -it wp_db mysql -u wpuser -pwppassword wordpress -e "SOURCE /backup.sql"`

You have to patch the url of the old site if you want it to run well in the container. 
Find the old url with `docker exec -it wp_cli wp option get siteurl --path=/var/www/html`
And patch it with `docker exec -it wp_cli wp search-replace 'old_url 'http://localhost:8080'` --all-tables --path=/var/www/html
