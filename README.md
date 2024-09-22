# WEB Stack Implementation (L.E.M.P Stack in AWS)
A Web stack project using Linux, Nginx, MySQL and PHP


Here’s a step-by-step guide on how to set up a LAMP stack (Linux, Nginx, MySQL, PHP) on an AWS EC2 t2.micro instance with Ubuntu 24.04 LTS for a DevOps project.

## Prerequisites

1. AWS account and a virtual server with Ubuntu server OS (instance of t2 micro).
2. Install [Git Bash](https://git-scm.com/downloads).
3. SSH key pair for connecting to ec2 instance vial GitBash terminal.


## Step 1 - Get your instance up and running

1. Sign up to AWS console following this [instruction](https://console.aws.amazon.com/).
2. After successfully signing up, launch a new instance.
3. Select a region close to you and launch an instance of t2.micro family with Linux ubuntu server.
4. Create a new .pem priavte key(If you don't have one already) and save it securely.
5. Configure your security groups by ensuring that the following ports are allowed:

   SSH (port 22) – for remote access
   
   HTTP (port 80) – for web traffic
   
   HTTPS (port 443) – for secure web traffic (optional)

6. Launch your ec2 instance

image 1

## Step 2 - Connect to the ec2 instance via ssh using Git bash

1. Open your Git Bash and navigate to your downloads directory

   `cd Downloads`

   
2. Connect to the instance by running
   
   `ssh -i your-key.pem ubuntu@(your-ec2-public-ip)`

   image 2
   Made an initial error of not including .pem while trying to ssh my instance using key pair 


## Step 3 - Installing the Nginx web server

1. Run the following to update the package index and upgrade installed packages:

   `sudo apt update`

   `sudo apt upgrade -y`

   image 4

2. Install Nginx

   `sudo apt install nginx`
     
3
4. To verify Nginx is running well, run this command

   `sudo systemctl status nginx`

   image 5

5. Now that your server is running, check the webpage via http://(your-ec2-public-ip)

   image 6


## Step 4 - Install MySQL

1. To Install MySQL:
   
   `sudo apt install mysql-server -y`

   image 7

2. Log in to MySQL:

   `sudo mysql`

   image 8

3. It is recommended that you run a script that comes pre-installed with MySQL. This script will set up the root password, remove anonymous users, disallow remote root login, and more.

   `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

   image 9

4. Exit the MySQL shell:
   
   `mysql> exit`

  image 10

5. Start the interactive script:
   
   `sudo mysql_secure_installation` 

6. You'll be prompted to change your default password to a password of your choice, remove anonymous users, disallow root logins etc. Answer y for yes or anything else to continue without enabling.

   image 11

7. Login into MySQL with your new password(if you changed it) to ensure it works:

   `sudo mysql -p`

   image 12

8. Exit the MySQL console:
   
   `mysql> exit`


## Step 5 - Install PHP

1. Install PHP, PHP extensions for Nginx and MySQL

   `sudo apt install php-fpm php-mysql -y`

   image 13

   
2. Confirm the php version

   image 14


## Step 5 - Configure Nginx to Use PHP Processor

1. Create a new directory for your website(my website would be named projectlemp, you can use any name for yourself.)
   
   `sudo mkdir /var/www/lempproject`

2. Assign ownership of the directory with the user environment variable
   
   `sudo chown -R $USER:$USER /var/www/lempproject`

3. Open a new configuration file in NGINX sites-available directory:

   `sudo nano /etc/nginx/sites-available/lempproject`

   Paste this in your nano editor

   ```
   server {
      listen 80;
      server_name  lempproject www.lempproject;
      root /var/www/lempproject;
  
      index index.php index.html index.htm;
  
  
      location / {
          try_files $uri $uri/ =404;
      }
  
      location ~ \.php$ {
          include snippets/fastcgi-php.conf;
          fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
      }
  
      location ~ /\.ht {
          deny all;
      }
   }
   ``` 

4. Activate your configuration by linking to the config file.

   `sudo ln -s /etc/nginx/sites-available/lempproject /etc/nginx/sites-enabled/`

5. Test your configuration for synatx error

    `sudo nginx -t`

   
   image 15
   
6. Disable default NGINX host:

   `sudo unlink /etc/nginx/sites-enabled/default`

      
7. Reload NGINX to apply changes

   `sudo systemctl reload nginx`

Congrats! Your new website is now active, but the web root /var/www/lempproject is still empty. You need to create an html file in that location so we can confirm that it works perfectly.


## Step 6 - Test the website with html scripts

1. Navigate to the project path

   `cd /var/www/lempproject`

2. Create an index.html file
   
   `touch index.html`

3. Open the inde.html file.
   
   `nano index.html`

4. Copy the following:

   ```
   <!DOCTYPE html>
   <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>LempProject</title>
      </head>
      <body>
          <h1>Welcome to Your First L.E.M.P project</h1>
          <p>This is a simple HTML file for testing purposes. If you're seeing this then it works.</p>
      </body>
   </html>
   ```
   
To save the index.html file, use Ctrl + S and Ctrl + X to exit

5. Now you view your new web page via http://(your-ec2-public-ip)

   image 16



## Step 7 - Testing PHP with NGINX

1. Create a PHP test file in your document root:

   `sudo nano /var/www/lempproject/info.php`

2. Add the following content to PHP test file:

   ```
   <?php
   phpinfo();
   ?>
   ```

3.Save and exit with Ctrl+X, Y and Enter

4. You can now access this page in your browser, visit http://<your-ec2-public-ip>/info.php. You should see a PHP info page.

   image 17

5. For security reasons, remove info.php

   `sud rm /var/www/lempproject/info.php`

   image 18

   You can always regenerate this file if you need it later


   
## Step 8 - Retrieving data from MySQL with PHP

1. Connect to the MySQL console using:

   `sudo mysql -p`

2. To create a new database:

   `CREATE DATABASE `example_database`;`

3. Create a new USER and grant permissions to use the DB:

   `CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'YOUR PASSWORD';`

4. Give this user permission to use the Database:

   `GRANT ALL ON example_database.* TO 'example_user'@'%';`

5. Exit mysql shell with:

   `mysql> exit`
   

7. Test if the user has proper permissions by logging in the MySQL console:

   `mysql -u example_user -p`         

8. Enter the password for your database

9. After accessing the database, confirm you have access to the example_database

    `SHOW DATABASES;`

    image 19

11. Create a test table called todo_list from the MySQL console, run the following commands

    ```
      CREATE TABLE example_database.mytodo_list (
      item_id INT AUTO_INCREMENT,
      item_id INT AUTO_INCREMENT,
      PRIMARY KEY(item_id)
      );
    ```

12. Insert your contents into the test table. You might want to repeat the format a few times using different values.

    ```
      mysql> INSERT INTO example_database.mytodo_list (content) VALUES ("My first item");
      mysql> INSERT INTO example_database.mytodo_list (content) VALUES ("My second item");
      mysql> INSERT INTO example_database.mytodo_list (content) VALUES ("My third item");
      mysql> INSERT INTO example_database.mytodo_list (content) VALUES ("My fourth item");    
    ```
13. To confirm that the data was successfully saved to your table, run:

    `mysql> SELECT * FROM example_database.mytodo_list;`

    image 20

 14. Exit mysql console

     `mysql> exit`

15. Create a PHP script that will connect to your database and query for your content

    `nano /var/www/lempproject/mytodo_list.php`

16. Copy the following content into your mytodo_list.php

    ```
    <?php
     $user = "example_user";
     $password = "YOUR PASSWORD";
     $database = "example_database";
     $table = "mytodo_list";
  
     try {
         $db = new PDO("mysql:host=localhost; dbname=$database", $user, $password);
        $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
  
        echo "<h2>TODO</h2><OL>";
  
        foreach ($db->query("SELECT * FROM $table") as $row) {
           echo "<LI>";
           echo "<input type='checkbox' " . ($row['completed'] ? 'checked' : '') . ">";
           echo "<span>" . $row['task'] . "</span>";
           echo "<br>";
           echo "<small>" . $row['description'] . "</small>";
           echo "<small> Due: " . $row['due_date'] . "</small>";
           echo "</LI>";
         }
  
        echo "</OL>";
     } catch (PDOException $e) {
        echo "Error: " . $e->getMessage();
     }
     ?>
    ```

    Save and close the file when you're done editing.

    
17. You can now access this page in your web browser http://(your-ec2-public-ip)/mytodo_list.php

    image 20


    If you're seeing this page, it means your PHP environment is ready to connect and interact with your MySQL server
 














