# WordpressToDocker
This document is a guide on how to transfer a WordPress website to a Docker container.

## What we need
We will need a working version of **Docker** and **Docker Desktop**.
The data from the site you want to transfer is also needed.
The data should be in the form of the *wp-content* folder. You can access it by going into the files of your already existing website.
You also need the *database* file.
You can get it via WordPress Local > "Database" tab > Open AdminNeo, then export the .sql from there.

**If you only have the UpdraftPlus backups**:
You have to use **WordPress Local** to load the data into a site and get the folder from there.

## Checking the prerequisites and creating the container
You first have to launch **Docker Desktop**; you can check it's running with `docker ps`.
Then you have to get the docker-compose.yml file from this git.

The `wordpress` and `wpcli` services in the docker-compose.yml use a **bind mount** pointing directly to your local `wp-content` folder, instead of copying it in with `docker cp`. This avoids very slow copies. Make sure the path in the `volumes:` section matches where your `wp-content` folder actually is on your machine.

You can then launch the containers with `docker compose up -d`.
You may run into an issue with a port already being in use; you just have to change the number of the problematic port.
Once it's running, you can run `docker compose ps`. You should see 4 lines with the status "Up" or "running".
Wait up to 30 seconds, then go to *http://localhost:8080* to check if the WordPress installation screen appears.
You can check if the content is there by running `docker exec -it wp_site ls /var/www/html/wp-content`, which should show you some folders.

## Loading the database
You can then copy the database into the site with `docker cp "path\to\backup.sql" wp_db:/backup.sql`
And load it with `docker exec -it wp_db mysql -u wpuser -pwppassword wordpress -e "SOURCE /backup.sql"`
You have to patch the URL of the old site for it to run correctly in the container.
Find the old URL with `docker exec -it wp_cli wp option get siteurl --path=/var/www/html`
And patch it with `docker exec -it wp_cli wp search-replace 'old_url' 'http://localhost:8080' --all-tables --path=/var/www/html`

## Fixing file permissions
Files may not have the right permission. Fix this with:
`docker exec -it wp_site chown -R www-data:www-data /var/www/html/wp-content`

## Admin access
Because we loaded the old database, which contains the users & admin accounts, we may have lost admin access. You can add yourself back in if needed with:
`docker exec -it wp_cli wp user create NAME mail@example.com --role=administrator --user_pass=password --path=/var/www/html`

## Final checks
Once you're logged you may have to do some steps to rebuild the visual in case its broken
1. Go to **Settings > Permalinks** and click **Save** 
2. Go to **Elementor> Editor> Tools > Elementor cache "clear"**. This rebuilds the CSS files.
