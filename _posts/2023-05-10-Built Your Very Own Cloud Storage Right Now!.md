---
title: Build your Very Own Cloud Storage Right Now!
date: 2023-05-10 02:00:00 +0500
categories: [Blogs, Cloud Computing]
tags: [cloud computing, cloud, private cloud, IaaS, PaaS, cloud storage]
---

## Introduction
When working in an organization or as a freelancer, one always faces an issue with file storage. There are millions of documents, images and important files that you need to store and access from multiple locations. You can always get yourself some space in Google Cloud, AWS or Azure. However, you can never be sure with senstitive files, especially if you're working for a government (or military) organization. That's where NextCloud comes in.

NextCloud is a free and open-source software for creating your very own private cloud storage. It offers tonnes of features while being dedicated to you and your organization. We can set up NextCloud in any environment including Windows, Linux. It can even be set up on a cloud instance! For this assignment, I will be setting up NextCloud on an Ubuntu-based Compute Instance on Google Cloud Platform. 

![](/assets/NextCloud/0.png)

## Task 1. Setting up NextCloud

### Pre-requisites

Update and upgrade your operating system
```
sudo apt-get update && sudo apt-get upgrade
```

Install the dependencies needed to configure and run NextCloud
```bash
sudo apt install apache2 mariadb-server libapache2-mod-php8.1
sudo apt install php8.1-gd php8.1-mysql php8.1-curl php8.1-mbstring php8.1-intl
sudo apt install php8.1-gmp php8.1-bcmath php-imagick php8.1-xml php8.1-zip
```

### Setting up backend database

Start the MySQL server and log into the server console as the root user
```bash
sudo systemctl start mysql
sudo mysql -u root -p
```

Create an admin user for your NextCloud server and grant it privileges for the "nextcloud" database.<br/><br/>
&#42; Change "abdullahirfan2001" to your username and "password" to your password.
```sql
CREATE USER 'abdullahirfan2001'@'localhost' IDENTIFIED BY 'password';
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'abdullahirfan2001'@'localhost';
FLUSH PRIVILEGES;
quit;
```
### Downloading and configuring NextCloud

Download NextCloud from the official server and unzip the downloaded archive.
```bash 
wget https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
unzip nextcloud*
```

Copy the unzipped folder to the ```/var/www/``` directory so that we can configure it to run on our ```Apache``` server.
```bash
sudo cp -r nextcloud /var/www/
```

Open the Apache configuration file for NextCloud...
```bash
sudo nano /etc/apache2/sites-available/nextcloud.conf
```
...and add the following lines to it
```bash
Alias /nextcloud "/var/www//assets/NextCloud/"

<Directory /var/www//assets/NextCloud/>
  Require all granted
  AllowOverride All
  Options FollowSymLinks MultiViews

  <IfModule mod_dav.c>
    Dav off
  </IfModule>

</Directory>
```

Enable the nextcloud website
```bash
sudo a2ensite nextcloud.conf
```

Enable additional modules needed to run NextCloud without any errors
```bash
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod env
sudo a2enmod dir
sudo a2enmod mime
```

Restart the Apache server to make the changes effective
```bash
sudo systemctl restart apache2
```

Enable SSL for secure communication to your cloud and reload the Apache server
```bash
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo service apache2 reload
```

Giving ownership of the NextCloud website to Apache
```bash
sudo chown -R www-data:www-data /var/www//assets/NextCloud/
```

### Installation Wizard

Navigate to ```https://{PUBLIC_IP}//assets/NextCloud/``` to access the installation wizard. For ```Database Name```, put in ```nextcloud``` and for ```Database host```, put in ```localhost:3306```. Enter the username and password you previously set up in your database and you're good to go.

![](/assets/NextCloud/1.png)

Congratulations, you have successfully set up your very own private cloud! Here is the screen you should now be able to see.Click on the folder-looking icon on the top to access your cloud storage.<br/>

![](/assets/NextCloud/2.png)

### Adding Files

To add a file to your very own cloud storage, click on the "+" button...<br/>

![](/assets/NextCloud/3.png)

...and select the file you want to upload.<br/>

![](/assets/NextCloud/4.png)

Now, you will be able to see your uploaded file in the directory listing. Click on your file to view it.<br/>

![](/assets/NextCloud/5.png)


## Task 2. Describe your cloud service model and deployment model

**Deployment Model:** Private Cloud<br/>
This cloud is private since it is deployed on your own infrastructure and is only usable by people/organizations that you have authorized.<br/><br/>
**Service Model:** IaaS and PaaS<br/>
The general opinion about cloud storage is that it is IaaS (Infrastructure as a Service) because the users are being provided with storage infrastructure. However, NextCloud can be classified as PaaS (Platform as a Service) since it provides a complete platform with security, user management and division of storage. I would classify  this service model as a hybrid of IaaS and PaaS.

## Task 3. Accessing NextCloud from a mobile device
Download the NextCloud client
- [Android](https://play.google.com/store/apps/details?id=com.nextcloud.client&hl=en&gl=US)
- [iOS](https://apps.apple.com/us/app//assets/NextCloud/id1125420102)

Enter the URL i.e. ```https://{PUBLIC IP}//assets/NextCloud/``` and click the arrow button.<br/>

![](/assets/NextCloud/ios/1.jpeg)

Grant access to your device to log into your NextCloud.<br/>

![](/assets/NextCloud/ios/5.jpeg)

Enter your login details<br/>

![](/assets/NextCloud/ios/4.jpeg)

Finally, grant access to the device for logging in with your account.<br/>

![](/assets/NextCloud/ios/3.jpeg)

Congratulations! You just accessed your very own private cloud from your mobile device.

![](/assets/NextCloud/ios/2.jpeg)

## Task 4. Accessing NextCloud from the Desktop
[Download](https://nextcloud.com/install/) the Desktop client for NextCloud and install it (a system reboot may be required).

![](/assets/NextCloud/pc/1.png)

Press the login button and enter the URL of your NextCloud server.

![](/assets/NextCloud/pc/2.png)

Since our SSL certificate is not registered with a valid Certificate Authority, we will get a prompt. Tick the box that says "Trust this certificate..." and proceed.

![](/assets/NextCloud/pc/3.png)

This will take you to your web browser to ask you to grant access to the desktop client. Press the login button.

![](/assets/NextCloud/pc/4.png)

Enter your login information to authorize the desktop client.

![](/assets/NextCloud/pc/5.png)

Finally, click on "Grant Access". 

![](/assets/NextCloud/pc/6.png)

Configure the synchronization settings for your private cloud with your desktop and proceed. 

![](/assets/NextCloud/pc/7.png)

Navigate to the NextCloud directory in your file explorer and you will be able to see the file you previously uploaded to your private cloud.

![](/assets/NextCloud/pc/9.png)

## Task 5. Creating a test user for Sir/TA

Go to the NextCloud web interface and log in as an administrator. Click on the user icon in the top-right of the portal.

![](/assets/NextCloud/6.png)

Click on "Users".

![](/assets/NextCloud/7.png)

Click on "New User".

![](/assets/NextCloud/8.png)

Finally, set the login and personal details for the new user and click on "Add a new user". Now, the new user will be able to login with their credentials.<br/><br/>
Test credentials for checking purposes are as follows:
- **Email**: ```mudassar.aslam@nu.edu.pk```
- **Password**: ```AbdullahIrfan```

![](/assets/NextCloud/9.png)

## Conclusion
NextCloud is probably one of the best choices for setting up private cloud storage for your organization and even personal use. It provides built-in features such as identity and access management, data seperation and more. Keep all your important data stored on a central server and access it anywhere in the world, on any device.




