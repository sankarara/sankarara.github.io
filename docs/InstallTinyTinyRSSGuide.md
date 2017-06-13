# Install Tiny Tiny RSS Guide
Guide for Ubuntu Server 16.04.1 LTS  
http://www.circuidipity.com/ttrss.html  
https://blog.doenselmann.com/tiny-tiny-rss-auf-raspberry-pi-installieren/
https://davidbeath.com/posts/installing-tiny-tiny-rss-from-scratch.html  
http://www.pcwelt.de/ratgeber/So-richten-Sie-Tiny-Tiny-RSS-fuer-den-eigenen-Webserver-ein-9932083.html  

## Install Php
`sudo apt install php php-fpm php-apcu php-curl php-cli php-pgsql php-gd php-mcrypt php-mbstring php-fdomdocument`

Improve security by editing **/etc/php/7.0/fpm/php.ini** and modifying pathinfo to 0
`sudo nano /etc/php/7.0/fpm/php.ini`
Modify `cgi.fix_pathinfo=0`

Restart Php `sudo systemctl restart php7.0-fpm`

## Install Nginx
`sudo apt install nginx`
`sudo systemctl start nginx`

## Install PostgreSQL
`sudo apt install postgresql`

## Install Tiny Tiny RSS
* config PostgreSQL
  * Create a new PostgreSQL database  
    ```
    sudo -u postgres psql
    postgres=# CREATE USER "www-data" WITH PASSWORD '*insertpasswordhere*';
    postgres=# CREATE DATABASE ttrss WITH OWNER "www-data";
    postgres=# GRANT ALL PRIVILEGES ON DATABASE ttrss to "www-data";
    postgres=# \quit
    ```
  * Peer authentication
    * Later when running the TTRSS update.php script you may run into the error "Peer authentication failed for user"
      `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
      * change "peer" to "trust"
        ```
        # "local" is for Unix domain socket connections only
          local   all             all                peer => trust
        ```
    * Restart postgresql `sudo systemctl restart postgresql`
* Get TTRSS  
    `git clone https://tt-rss.org/git/tt-rss.git ttrss`  
    `cd ttrss`  
    `chmod -R 777 cache/images cache/js cache/export cache/upload feed-icons lock`  
    `cd ..`  
    `mv ttrss /var/www/html/ttrss`  
    `sudo chown -R www-data:www-data /var/www/html/ttrss`  
* config Nginx
  * Create a server block  
      `sudo nano /etc/nginx/sites-available/ttrss`  
      then  
      ```
        server {
            listen 8080;
            listen [::]:8080;

            root /var/www/html/ttrss;
            index index.php;

            access_log /var/log/nginx/ttrss_access.log;
            error_log /var/log/nginx/ttrss_error.log info;

            server_name _;

            location / {
                index           index.php;
            }

            location ~ \.php$ {
                try_files $uri = 404; #Prevents autofixing of path which could be used for exploit
                fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
                fastcgi_index index.php;
                include /etc/nginx/fastcgi.conf;
            }
        }
      ```
  * Enable block  
    `cd /etc/nginx/sites-enabled`  
    `sudo ln -s /etc/nginx/sites-available/ttrss`  
    `sudo systemctl restart nginx`
* configure TTRSS
  * Navigate to http://serverip:8080/
  * Config
    ```
      Database type: [select PostgreSQL]  
      Username: www-data  
      Password: [database password created earlier]  
      Database Name: ttrss  
      Hostname: [leave blank]  
      Port: 5432
    ```
  * Updating Feeds
    * https://tt-rss.org/gitlab/fox/tt-rss/wikis/UpdatingFeeds
    * `sudo systemctl enable /etc/systemd/system/ttrss.service`
      * https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/
      * https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units
      * Systemd For Ubuntu 16.04 it's `/lib/systemd/system/`
