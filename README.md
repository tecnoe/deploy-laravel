Sure. Here are the steps to install and configure PHP, Nginx, and Laravel on an Ubuntu server:

### Prerequisites
1. **Update the system packages:**
    ```bash
    sudo apt update && sudo apt upgrade
    ```

2. **Install PHP and required modules:**
    ```bash
    sudo apt install php php-fpm php-mysql php-curl php-mbstring php-zip php-gd php-xml
    ```

3. **Install Nginx web server:**
    ```bash
    sudo apt install nginx
    ```

4. **Install MySQL server:**
    ```bash
    sudo apt install mysql-server mysql-client
    ```

5. **Secure MySQL installation:**
    ```bash
    sudo mysql_secure_installation
    ```

### Creating a database for the Laravel application
1. **Create a new database for the Laravel application:**
    ```sql
    CREATE DATABASE laravel_db;
    ```

2. **Create a new user with access to the laravel_db database:**
    ```sql
    CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'laravel_password';
    ```

3. **Grant all privileges to the laravel_user user on the laravel_db database:**
    ```sql
    GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
    ```

4. **Flush privileges:**
    ```sql
    FLUSH PRIVILEGES;
    ```

### Creating a new Laravel application
1. **Install Composer:**
    ```bash
    curl -sS https://getcomposer.com/installer | php
    ```

2. **Move the Composer executable to /usr/local/bin:**
    ```bash
    sudo mv composer.phar /usr/local/bin/composer
    ```

3. **Create a new Laravel application directory:**
    ```bash
    mkdir laravel-app
    cd laravel-app
    ```

4. **Create a new Laravel application:**
    ```bash
    composer create-project laravel/laravel .
    ```

5. **Set the correct file permissions:**
    ```bash
    sudo chown -R www-data:www-data .
    ```

### Configuring Nginx
1. **Create a new Nginx server block for the Laravel application:**
    ```bash
    sudo nano /etc/nginx/sites-available/laravel_app
    ```

2. **Add the following content to the file:**
    ```nginx
    server {
        listen 80;
        server_name example.com;

        root /home/user/laravel-app/public;

        index index.php index.html;

        location / {
            try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
    ```

3. **Enable the new server block:**
    ```bash
    sudo ln -s /etc/nginx/sites-available/laravel_app /etc/nginx/sites-enabled/laravel_app
    ```

4. **Test the Nginx configuration:**
    ```bash
    sudo nginx -t
    ```

5. **Restart Nginx:**
    ```bash
    sudo systemctl restart nginx
    ```

### Customizing the Laravel application
1. **Access the Laravel application in your web browser:**
    [http://example.com](http://example.com)

2. **Configure the database connection in the .env file:**
    ```bash
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_DATABASE=laravel_db
    DB_USERNAME=laravel_user
    DB_PASSWORD=laravel_password
    ```

3. **Run the Laravel migrations to create the necessary database tables:**
    ```bash
    php artisan migrate
    ```

4. **Start the Laravel development server:**
    ```bash
    php artisan serve
    ```

5. **Access the Laravel application in your web browser:**
    [http://localhost:8000](http://localhost:8000)

### Security considerations
- Use strong passwords for your database user and Laravel application.
- Keep your Laravel application and dependencies up to date.
- Enable file caching and view caching in your Laravel application.
- Consider using a web application firewall (WAF) to protect your Laravel application from common web attacks.
- Regularly backup your Laravel application and database.

### Fuentes
- [jaranguda.com/cara-install-laravel-7-dan-ssl-di-ubuntu-20-04-focal-fossa/](https://jaranguda.com/cara-install-laravel-7-dan-ssl-di-ubuntu-20-04-focal-fossa/)
- [github.com/allebb/hooker](https://github.com/allebb/hooker)
- [www.digitalocean.com/community/tutorials/how-to-install-laravel-with-nginx-on-an-ubuntu-12-04-lts-vps](https://www.digitalocean.com/community/tutorials/how-to-install-laravel-with-nginx-on-an-ubuntu-12-04-lts-vps)

### Folder permissions for Laravel 10 project anotations. 

Just to state the obvious for anyone viewing this discussion.... if you give any of your folders 777 permissions, you are allowing ANYONE to read, write and execute any file in that directory.... what this means is you have given ANYONE (any hacker or malicious person in the entire world) permission to upload ANY file, virus or any other file, and THEN execute that file...

IF YOU ARE SETTING YOUR FOLDER PERMISSIONS TO 777 YOU HAVE OPENED YOUR SERVER TO ANYONE THAT CAN FIND THAT DIRECTORY. Clear enough??? :)

There are basically two ways to setup your ownership and permissions. Either you give yourself ownership or you make the webserver the owner of all files.

Webserver as owner (the way most people do it, and the Laravel doc's way):

assuming www-data (it could be something else) is your webserver user.

```bash
sudo chown -R www-data:www-data /path/to/your/laravel/root/directory
```

if you do that, the webserver owns all the files, and is also the group, and you will have some problems uploading files or working with files via FTP, because your FTP client will be logged in as you, not your webserver, so add your user to the webserver user group:

```bash
sudo usermod -a -G www-data ubuntu
```

Of course, this assumes your webserver is running as www-data (the Homestead default), and your user is ubuntu (it's vagrant if you are using Homestead).

Then you set all your directories to 755 and your files to 644... SET file permissions
```bash
sudo find /path/to/your/laravel/root/directory -type f -exec chmod 644 {} \;
```
   
SET directory permissions

```bash
sudo find /path/to/your/laravel/root/directory -type d -exec chmod 755 {} \;
```
Your user as owner

I prefer to own all the directories and files (it makes working with everything much easier), so, go to your laravel root directory:

```bash
cd /var/www/html/laravel >> assuming this is your current root directory
sudo chown -R $USER:www-data .
```

Then I give both myself and the webserver permissions:

```bash
sudo find . -type f -exec chmod 664 {} \;   
sudo find . -type d -exec chmod 775 {} \;
```
Then give the webserver the rights to read and write to storage and cache

Whichever way you set it up, then you need to give read and write permissions to the webserver for storage, cache and any other directories the webserver needs to upload or write too (depending on your situation), so run the commands from bashy above :

```bash
sudo chgrp -R www-data storage bootstrap/cache
sudo chmod -R ug+rwx storage bootstrap/cache
```

Now, you're secure and your website works, AND you can work with the files fairly easily
