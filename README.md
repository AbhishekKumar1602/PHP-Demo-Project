# Introduction
A Sample Web Application of E-Commerce Built for Demonestrating LEMP and LAMP Server Setup.

## Install Pre-Requisites

1. **Install UFW (Uncomplicated Firewall)**

   ```bash
   sudo apt-get update
   sudo apt-get install -y ufw
   sudo ufw enable
   ```

## Install and Configure Database

1. **Install MariaDB**

   ```bash
   sudo apt-get install -y mariadb-server
   sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
   sudo systemctl start mariadb
   sudo systemctl enable mariadb
   ```

2. **Configure Firewall for Database**

   ```bash
   sudo ufw allow 3306/tcp
   sudo ufw reload
   ```

3. **Configure Database**

   ```bash
   sudo mysql
   MariaDB > CREATE DATABASE ecomdb;
   MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
   MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
   MariaDB > FLUSH PRIVILEGES;
   ```

   > On a Multi-Node Setup, Remember to Provide the IP Address of the Database Server Here.: `'ecomuser'@'web-server-ip'`

4. **Load Product Inventory Information to the Database**

   - Create the `db-load-script.sql`

   ```bash
   cat > db-load-script.sql <<-EOF
   USE ecomdb;
   CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment, Name varchar(255) default NULL, Price varchar(255) default NULL, ImageUrl varchar(255) default NULL, PRIMARY KEY (id)) AUTO_INCREMENT=1;

   INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");
   EOF
   ```

   - Run the SQL Script

   ```bash
   sudo mysql < db-load-script.sql
   ```

## Install and Configure Web Server

1. **Install Required Packages**

   ```bash
   sudo apt-get install -y nginx php-fpm php-mysql
   sudo ufw allow 80/tcp
   sudo ufw reload
   ```

2. **Configure Nginx**
	
   - Open the Nginx configuration file
   
   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   - Update the configuration to look like this:

   ```nginx
	user  www-data;
	worker_processes  auto;

	events {
	    worker_connections  1024;
	}

	http {
	    include       mime.types;
	    default_type  application/octet-stream;

	   server {
		listen 80;
		server_name example.com wwww.example.com;

		root /var/www/DemoPHPProject;
		index index.php;

		location / {
		    try_files $uri $uri/ =404;
		}

		location ~ \.php$ {
		    include fastcgi.conf;
		    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
		}
	    }
	}
   ```

3. **Start Nginx and PHP-FPM**

   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx

   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```

4. **Download Application Code**

   ```bash
   sudo apt-get install -y git
   sudo git clone git clone https://github.com/AbhishekKumar1602/PHP-Demo-Project.git
   ```

5. **Update `index.php`**

  - Update the `index.php` file to connect to the correct database server. For reference, PHP code snippet should look like this in this case of `localhost` since the database is on the same server:

   ```php
   <?php
   $link = mysqli_connect('localhost', 'ecomuser', 'ecompassword', 'ecomdb');
   if ($link) {
       $res = mysqli_query($link, "select * from products;");
       while ($row = mysqli_fetch_assoc($res)) { ?>
   <?php
       }
   }
   ?>
   ```

   > On a Multi-Node Setup, Remember to Provide the IP Address of the Database Server Here.
