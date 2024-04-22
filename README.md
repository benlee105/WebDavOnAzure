# WebDav Server On Azure
Setup WebDav server on Azure to access files via Windows File Explorer!  

Inspired by:  
https://reintech.io/blog/installing-configuring-webdav-server-ubuntu-22  
https://www.blackhillsinfosec.com/deploying-a-webdav-server/

## Step 1: Create VM on Azure
```
In Azure, create a Virtual Machine with Ubuntu.
Leave all settings in Azure as default then click Review + Create
Download the SSH key when prompted
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/246498dc-874f-4dff-a2d0-587d1e4f3e52)

## Step 2: SSH into VM
SSH into the server using the downloaded SSH key  
```
ssh -i C:\Users\<name>\Downloads\key.pem azureuser@<your_VM_Public_IP>
```  

## Step 3: Install apache2 on VM
Run the following commands:  
```
sudo apt update
sudo apt install apache2
sudo systemctl enable apache2
sudo systemctl start apache2
sudo a2enmod dav
sudo a2enmod dav_fs
sudo systemctl restart apache2
sudo mkdir /var/www/webdav
sudo chown -R www-data:www-data /var/www/webdav
sudo chmod -R 755 /var/www/webdav
```

## Step 4: Configure Apache
Configure Apache's config file to alias /var/www/webdav to /webdav, and enable webdav
```
sudo nano /etc/apache2/sites-available/000-default.conf
```

Add the following configuration **inside** the <VirtualHost *:80> block after any existing config:  
```
<VirtualHost *:80>
Alias /webdav /var/www/webdav
<Directory /var/www/webdav>
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
  Dav On
</Directory>
</VirtualHost>
```  
Close nano with CTRL+X, save when prompted.  

## Step 5: Restart Apache
```
sudo systemctl restart apache2
```

## Step 6: Secure WebDav Access via iptables
In Ubuntu VM, allow 80 and 443 traffic in:  

```
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
sudo service iptables save
```

## Step 7: Secure WebDav Access via Azure route
Limit access to the WebDav server on Azure:  

```
Networking > Add inbound rule
Source is your victim IP
Destination any
Ports 80 and 443
Priority any number is fine
```
![Step 7](https://github.com/benlee105/WebDavOnAzure/assets/62729308/46b126c7-0935-4bfc-8105-52604b94444c)


## Step 8: Set DNS name for server
```
In Azure, click on your server, then click on the DNS name option, it'll redirect you to your IP configuration
Then fill up the DNS Name Label (optional) with the DNS name you want, then Save.
```  
![Step 8_1](https://github.com/benlee105/WebDavOnAzure/assets/62729308/b532ab29-fe67-4030-bd6b-49281952b36e)
  
![Step 8_2](https://github.com/benlee105/WebDavOnAzure/assets/62729308/c0deee02-8be6-4932-ad50-d2e19139b335)


## Step 9: Secure webserver with HTTPS, with Let's Encrypt
SSH into your Ubuntu server then run the commands below  

```
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
```  
When certbot prompts, type in your cloudapp.azure.com domain name.  

![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/4e6f5ace-ae36-4b95-868a-51e11fe3eb44)


## Step 10: Harden SSL settings
Harden based on recommendations here https://httpd.apache.org/docs/trunk/ssl/ssl_howto.html  
```
sudo nano /etc/apache2/mods-available/ssl.conf
SSLCipherSuite ECDHE-ECDSA-AES-256-GCM-SHA384:!aNULL
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLCompression Off
SSLSessionTickets off
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/acc85449-698d-4998-84b1-039f47693e4c)


## Step 11: Upload files to /var/www/webdav
Create an Azure data container, upload files into it, set permission as Blob  
Then use wget on the Ubuntu server!  
  
```
cd /var/www/webdav
wget https://<yourstorage>.blob.core.windows.net/<folder>/<yourfile.exe>
```  

## Step 12: Set permissions in /var/www/webdav
chown to set permission to www-data  

```
cd /var/www/webdav
sudo chown www-data:www-data *.exe
sudo chown www-data:www-data *.txt
```

## Step 13: Test access to your files
Use a browser to test access to your webdav site based on Step 8.  

Also, use Windows File Explorer to test access to your webdav site:  
```
file://<webdavsite based on Step 8>
```

## Troubleshoot: If unable to access files in WebDav folder
If you're unable to access content in the WebDav server, do the steps below  

```
ls -la /var/www/webdav
```
You will face issues if file permissions are "root root", it MUST be "www-data www-data"  

![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/d93152de-ebaa-414b-a7ed-825e82aa3364)
  

To fix, use command below:  
```
cd /var/www/webdav
sudo chown www-data:www-data <file>
```  

Or if you want to apply www-data permissions based on filetype:  
```
cd /var/www/webdav
sudo chown www-data:www-data *.exe
```
