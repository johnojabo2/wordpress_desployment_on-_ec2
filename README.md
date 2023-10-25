Creating comprehensive documentation for installing WordPress on an Amazon EC2 Ubuntu instance involves several steps, from setting up the instance to configuring WordPress. Below, I've outlined a detailed guide for the process:

## Installation of WordPress on Amazon EC2 Ubuntu

### Prerequisites:

- An AWS account.
- An Amazon EC2 instance running Ubuntu.
- AWS key pair for SSH access.
- A domain name (optional).

### Step 1: Create an EC2 Instance

1. Log in to your AWS Management Console.

2. Navigate to the EC2 dashboard and click "Launch Instances."

3. Choose an Amazon Machine Image (AMI). Select "Ubuntu Server" (or the latest version of Ubuntu).

4. Choose an instance type based on your needs and click "Next."

5. Configure the instance details. Set the number of instances and network configurations as needed. Click "Next."

6. Add storage to your instance based on your requirements. You can keep the default settings for most cases. Click "Next."

7. Add tags to your instance (optional). Tags help you identify instances in the AWS Management Console. Click "Next."

8. Configure security groups. Create a new security group or use an existing one. Make sure to allow SSH (port 22) and HTTP (port 80) traffic. Click "Review and Launch."

9. Review your instance settings and click "Launch."

10. Select an existing key pair or create a new one. This key pair will be used to SSH into your instance. Click "Launch Instances."

11. Your instance is now launching. Click "View Instances" to see the status.

### Step 2: Connect to Your EC2 Instance

1. Open your terminal (on Windows, you can use an SSH client like PuTTY).

2. Navigate to the directory where your AWS key pair file is located.

3. Change the key pair file's permissions for security:  
   ```
   chmod 400 your-key-pair.pem
   ```

4. Connect to your instance using SSH:  
   ```
   ssh -i "your-key-pair.pem" ubuntu@your-ec2-public-ip
   ```

### Step 3: Update the System

After connecting to your EC2 instance, run the following commands to update the system packages:

```
sudo apt update
sudo apt upgrade -y
```

### Step 4: Install Required Software

Install the necessary software for running WordPress:

```
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

### Step 5: Configure MySQL

Secure your MySQL installation and set a root password:

```
sudo mysql_secure_installation
```

During this process, you'll be asked to set the MySQL root password. Make sure to remember it.

### Step 6: Create a Database for WordPress

Log in to MySQL:

```
mysql -u root -p
```

Create a new database:

```sql
CREATE DATABASE your_wp_database;
```

Create a user for the database and set a password:

```sql
CREATE USER 'your_wp_user'@'localhost' IDENTIFIED BY 'your_wp_password';
```

Grant privileges to the user:

```sql
GRANT ALL PRIVILEGES ON your_wp_database.* TO 'your_wp_user'@'localhost';
```

Flush privileges and exit:

```sql
FLUSH PRIVILEGES;
EXIT;
```

### Step 7: Download and Configure WordPress

Download and extract WordPress:

```bash
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

Move the files to your web server directory:

```bash
sudo mv /tmp/wordpress/* /var/www/html
```

Create a WordPress configuration file:

```bash
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
```

Edit the configuration file with your database information:

```bash
sudo nano wp-config.php
```

Find the following lines and modify them:

```php
define('DB_NAME', 'your_wp_database');
define('DB_USER', 'your_wp_user');
define('DB_PASSWORD', 'your_wp_password');
```

Save and exit the text editor.

### Step 8: Configure Apache

Create an Apache configuration file for your site:

```bash
sudo nano /etc/apache2/sites-available/your-site.conf
```

Add the following configuration, replacing `your-site` with your domain name (if you have one):

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@your-site
    DocumentRoot /var/www/html
    ServerName your-site
    ServerAlias www.your-site

    <Directory /var/www/html>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save and exit the text editor.

Enable the site and the rewrite module:

```bash
sudo a2ensite your-site.conf
sudo a2enmod rewrite
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

### Step 9: Complete WordPress Installation

In your web browser, navigate to your EC2 instance's public IP or your domain name (if you have one). You'll see the WordPress installation page.

Follow the on-screen instructions to complete the WordPress setup, including creating an admin user and providing your site's information.

### Step 10: UserData Script (Optional)

Here is a UserData script that can automate the installation process. When you launch a new EC2 instance, you can include this script to perform all the necessary steps:

```bash
#!/bin/bash

# Update and upgrade the system
sudo apt update
sudo apt upgrade -y

# Install required software
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip

# Secure MySQL installation and set the root password
sudo mysql_secure_installation

# Create a MySQL database for WordPress
mysql -u root -e "CREATE DATABASE your_wp_database;"
mysql -u root -e "CREATE USER 'your_wp_user'@'localhost' IDENTIFIED BY 'your_wp_password';"
mysql -u root -e "GRANT ALL PRIVILEGES ON your_wp_database.* TO 'your_wp_user'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"

# Download and configure WordPress
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo mv /tmp/wordpress/* /var/www/html
cd /var/www/html
sudo cp wp-config-sample.php wp-config.php
sed -i "s/database_name_here/your_wp_database/" wp-config.php
sed -i "s/username_here/your_wp_user/" wp-config.php
sed -i "s/password_here/your_wp_password/" wp-config.php

# Create an Apache configuration file
echo "<VirtualHost *:80>
    ServerAdmin webmaster@your-site
    DocumentRoot /var/www/html
    ServerName your-site
    ServerAlias www.your-site

    <Directory
