# Installing a LAMP Stack and Wordpress on Debian Linux

## Overview
This guide walks you through installing, setting up, and configuring a new WordPress website on a Debian-based Linux system with a LAMP (Linux, Apache, MariaDB, PHP) stack. By the end of this guide, you will have a functioning WordPress site running on Apache, backed by a MariaDB (MySQL-compatible) database, and ready for initial configuration through the WordPress web interface. This guide assumes you have a valid domain for encryption purposes, and that you have correctly configured DNS for that domain.

## Audience
This guide is intended for more advanced Linux users who are comfortable using the command line and have basic familiarity with system administration concepts. It is suitable for:

- Developers setting up a local or remote WordPress instance
- System administrators deploying a small WordPress site
- Technical users learning how WordPress fits into a LAMP stack
- Linux hobbyists who would enjoy hosting their own website

This guide does not assume prior experience with WordPress.

## Prerequisites
Before starting, you should have:

- A Debian-based Linux system (Debian 11/12 or compatible)
- Root or sudo access
- An active internet connection
- A domain name with correctly configured DNS

## System Requirements

- Apache 2.x
- MariaDB or MySQL
- PHP 8.x with required extensions
- At least 1 GB of RAM (2 GB recommended)
- Approximately 1 GB of available disk space

## Step 1: Update the System
Before we begin installing our new website on our Linux server, a good first practice would be to apply any new system updates. This will ensure that you are up to date with current security patches and that we are using the latest package versions available from the Debian repository. Run the following commands at the terminal window:

```
sudo apt update
sudo apt full-upgrade
```

Afterwards, reboot your system to load any new system software that has been installed. Note this is only strictly necessary if the system kernel has been updated.

```
sudo reboot
```

## Step 2: Install Apache
Apache is the web server that will serve content to the visitors of your website. It is highly recommended that you install Apache directly from the Debian software repository.

```
sudo apt install apache2
```

When installation is complete, check to see if the server is running with the following command:

```
sudo systemctl status apache2
```

Furthermore, you can check to see if the server is serving pages by opening a web browser and visiting the server's IP address or hostname:

```
http://example.com
```

If Apache is installed and running correctly, you should see the Apache2 default page. At this point, Apache is installed and ready to serve the WordPress site.

### Configure Apache
Next, you will configure the Apache web server. Note that throughout this guide, "example.com" will be used as a placeholder. Be sure to replace this with your own domain where applicable.

Create a directory where you will store your files and give it the appropriate permissions:

```
sudo mkdir -p /var/www/example.com
sudo chown -R www-data:www-data /var/www/example.com
sudo chmod 755 /var/www/example.com
```

This will create a directory for your WordPress files, and make sure the Apache server has access to read and serve them to users without granting unnecessary access.

#### VirtualHost configuration
Create a VirtualHost file in Apache for your website:

```
sudo nano /etc/apache2/sites-available/example.com.conf
```

Add the following configuration, replacing the example domain with your own domain name or server IP address:

```
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    DocumentRoot /var/www/example.com

    <Directory /var/www/example.com>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example.com_error.log
    CustomLog ${APACHE_LOG_DIR}/example.com_access.log combined
</VirtualHost>
```

Save the file and exit the editor. 

#### Enable Apache rewrite module
Apache's rewrite module will be required by WordPress to create permalinks to blog posts, which is useful for indexing and sharing your pages. First, check to see if the rewrite module is already enabled:

```
apache2ctl -M | grep rewrite
```

If you see output similar to "rewrite_module (shared)", then the module is already running and you may move on to the next step. If not, then run the following command to enable the module before proceeding:

```
sudo a2enmod rewrite
```

#### Enable the new website
Next, we will issue commands to enable the new site, disable the default site, enable the rewrite module (a requirement for WordPress), and reload the Apache server. 

```
sudo a2ensite example.com.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

Again, we can check that the server is running with systemctl:

```
sudo systemctl status apache2
```

### Securing your website with Let's Encrypt
It is recommended that all websites use HTTPS, especially those in production environments. Let's Encrypt is a free service that provides signed certificates to website owners that can be automatically installed and renewed.

Begin by installing the certbot software:

```
sudo apt install certbot python3-certbot-apache
```

Once installed, issue the following command to request a certificate for your domain:

```
sudo certbot --apache -d example.com,www.example.com -m yourname@example.com 
```

You will be prompted to agree to the terms of service and asked if you want to redirect all HTTP traffic to HTTPS, which is recommended.

Verify that HTTPS is working by visiting your website:

```
https://example.com
```

If configured correctly, your browser should indicate that the connection is secure, typically with a lock icon in the address bar.

#### Automatic Certificate Renewal
Certificates from Let's Encrypt are valid for 90 days, and the certbot utility automatically installs a system timer to renew your certificates. You can verify the renewal timer with:

```
sudo systemctl list-timers | grep certbot
```

You should see output similar to:

```
Thu 2026-01-29 07:07:26 PST    23min Wed 2026-01-28 17:53:22 PST      12h ago certbot.timer                certbot.service
```

If desired, you can test the renewal process manually with:

```
sudo certbot renew --dry-run
```

## Step 3: Install MariaDB (MySQL)
Next, we will install MariaDB, the backend database server for our WordPress site.

```
sudo apt install mariadb-server
```

Once the installation is complete, verify that the MariaDB service is running:

```
sudo systemctl status mariadb
```

If the service is running correctly, the status output should show active (running).

### Secure MariaDB
MariaDB has a helper script that will walk you through the process of applying standard security practices to your database server.

```
sudo mysql_secure_installation
```

During this process, you will be prompted to:

 - Set a password for the MariaDB root account
 - Remove anonymous users
 - Disable remote root login
 - Remove the test database
 - Reload privilege tables

It is recommended that you answer "yes" to each prompt unless you have a specific reason otherwise.

### Create the WordPress Database

Next, log into the database shell as the root user:

```
sudo mysql
```

Then, execute the following commands. Be sure to note the username for your WordPress database and also make sure to use a strong password:

```
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'strong_password_here';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

The database and user required by WordPress are now configured.

## Step 4: Installing PHP
WordPress is written in PHP, and requires several extensions to function properly. Thankfully the necessary software is located in the package manager:

```
sudo apt install php php-mysql php-curl php-gd php-xml php-mbstring php-zip php-intl
```

Once installation is complete, you can verify that PHP has been installed by running:

```
php -v
```

This should output the currently installed PHP version. To check that PHP is working with Apache, create a file in your web root directory:

```
sudo nano /var/www/example.com/info.php
```

Add the following to the file:

```
<?php
phpinfo();
```

Save the file, and then change the owner so that the server can read it:

```
sudo chown www-data:www-data /var/www/example.com/info.php
```

Check the file on your web browser:

```
https://example.com/info.php
```

If PHP is configured properly, you will see a PHP information page. Once verification is complete, it is vital that you remove this file for security purposes:

```
sudo rm /var/www/example.com/info.php
```

## Step 5: Download and Configure WordPress
Next, you will download the latest WordPress software and copy it to your web directory. First, change to the "/tmp" directory and use the following command to download WordPress:

```
cd /tmp
curl -O https://wordpress.org/latest.zip
```

Decompress the file:

```
unzip latest.zip
```

If the "unzip" command does not exist, install it with:

```
sudo apt install unzip
```

### Basic WordPress Configuration
Change into the new WordPress directory:

```
cd wordpress
```

Move the sample config into place, and edit the file:

```
mv wp-config-sample.php wp-config.php
nano wp-config.php
```

Look for the following near the top of the configuration file:

```
// ** Database settings - You can get this info from your web host ** //                                                       
/** The name of the database for WordPress */                                                                                  
define( 'DB_NAME', 'database_name_here' );
/** Database username */                                                                                                       
define( 'DB_USER', 'username_here' );
/** Database password */                                                                                                       
define( 'DB_PASSWORD', 'password_here' );
```

Change the values for DB_NAME, DB_USER, and DB_PASSWORD to those used in Step 3.

Next look for the section about Salt values. 

```
define( 'AUTH_KEY',         'put your unique phrase here' );                                                                   
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );                                                                   
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );                                                                   
define( 'NONCE_KEY',        'put your unique phrase here' );                                                                   
define( 'AUTH_SALT',        'put your unique phrase here' );                                                                   
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );                                                                   
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );                                                                   
define( 'NONCE_SALT',       'put your unique phrase here' ); 
```

WordPress Salts add site-specific randomness to password hashing and session data, helping protect against credential reuse, hash cracking, and session hijacking. Visit the following URL to obtain a Salt key and add your unique key to the configuration file:

```
https://api.wordpress.org/secret-key/1.1/salt/
```

### Moving the WordPress Software into Place

Now copy the contents of the "wordpress" directory to your webserver's root directory, and make sure permissions are set so that Apache can read the files and for security purposes:

```
sudo cp -r /tmp/wordpress/* /var/www/example.com/.
sudo chown -R www-data:www-data /var/www/example.com/
sudo find /var/www/example.com -type f -exec chmod 644 {} \;
sudo find /var/www/example.com -type d -exec chmod 755 {} \;
```

## Step 6: Complete the WordPress Installation

At this point, you should have a functioning WordPress server. We will just need to log into the web interface at `https://www.example.com/wp-admin` and finalize the setup. During this step, you will choose a site title, admin username, and admin password. Be sure to use a strong password for your admin account and do not reuse any credentials that you used to create your WordPress database in MariaDB. Also be sure to use a real email address for the admin user so that you can receive system emails from your WordPress server.

## Verifying the Installation
The WordPress dashboard is able to show you if there are any Apache or PHP errors in your installation. WordPress has impressive documentation, and the Dashboard will often offer suggestions and links to help.

Try creating your first post on your website and verify if the permalink to your newly created post works. This will ensure that the Apache rewrite module is working correctly.

## Common Issues and Troubleshooting
### HTTPS Issues
If you are having issues with setting up HTTPS with Let's Encrypt, ensure that your domain name resolves to the server’s public IP address before running Certbot. If Certbot fails, Apache configuration errors are the next most common cause. Check Apache status and logs for details.

### Apache Issues
If the server is running according to systemctl but not serving pages, make sure that ports 80 (HTTP) and 443 (HTTPS) are allowed through your firewall software. A common firewall utility for Debian is ufw and can typically be configured to allow web traffic with the following command:

```
sudo ufw allow "WWW Full"
```

Please note that ufw is not the default firewall software on all systems and that you may have different firewall software on your server.

If your webserver loads, but you are getting permission errors, make sure that all of the files in your `/var/www/example.com` directory are owned by www-data and have appropriate permissions (755 for directories and 644 for files).

Typos can be a leading cause of errors: double-check your configuration files for Apache and in wp-config.php to make sure that everything is entered correctly.

## Conclusion: Ongoing Security and Next Steps
Congratulations on installing a LAMP stack on Debian Linux and configuring your WordPress website! All of the components that you need to publish a website for your personal use or your business are now in place. Moving forward, you will need to create your content and maintain good administrative habits to keep your WordPress site safe and secure.

You may remove the working files you placed in the /tmp directory:

```
sudo rm -r /tmp/wordpress
```

The best thing you can do to keep your WordPress install secure moving forward is to make sure that both your Linux server and WordPress install stay up-to-date on the latest security patches. Run `apt update` and `apt full-upgrade` to keep your Debian install updated and check the WordPress dashboard for any updates to the WordPress software, plugins, and themes. Limiting the number of themes and plugins is another way to keep your install more secure.

Additional security software such as `fail2ban` can be installed to further harden your Linux installation.

