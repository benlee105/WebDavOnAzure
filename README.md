# WebDavOnAzure
Setup WebDav server on Azure
Inspired by:
https://reintech.io/blog/installing-configuring-webdav-server-ubuntu-22 &&
https://www.blackhillsinfosec.com/deploying-a-webdav-server/

## Step 1: Create VM on Azure
```
In Azure, create a Virtual Machine with Ubuntu.
Leave all settings in Azure as default then click Review + Create
Download the SSH key when prompted
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/246498dc-874f-4dff-a2d0-587d1e4f3e52)

## Step 2: SSH into VM
```
SSH into the server using the downloaded SSH key
ssh -i C:\Users\benle\Downloads\webdav_key.pem azureuser@52.139.216.120
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/77b100d0-9fe5-40a0-a005-3e62d977b125)

## Step 3: Install apache2 on VM
```
Run the following commands:
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
```
sudo nano /etc/apache2/sites-available/000-default.conf

Add the following configuration **inside** the <VirtualHost *:80> block:
<Directory /var/www/webdav>
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
  Dav On
    <LimitExcept GET HEAD OPTIONS PROPFIND>
      Deny from all
    </LimitExcept>
    Satisfy all
  </Location>
</Directory>

Close nano with CTRL+X
```

## Step 5: Restart Apache
```
sudo systemctl restart apache2
```

## Step 6: Secure WebDav Access via iptables
```
In Ubuntu VM, allow 80 and 443 traffic in:

sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
sudo service iptables save
```

## Step 7: Secure WebDav Access via Azure route
```
Limit access to the WebDav server on Azure
Networking > Add inbound rule
Source is your victim IP
Destination your server (or any is fine for now)
Ports 80 and 443
Priority any number is fine
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/d9df2f7b-968b-40e9-b5ca-9c24dfbeb91b)


## Troubleshoot: If unable to access files in WebDav folder
```
ls -la /www/apache
You will face issues if file permissions are "root root", it MUST be "www-data www-data"
```
![image](https://github.com/benlee105/WebDavOnAzure/assets/62729308/d93152de-ebaa-414b-a7ed-825e82aa3364)

## Troubleshoot: Uploading files to WebDav server
```
Create an Azure data container, upload files into it, set permission as Blob
Then use wget on the Ubuntu server!
Then chown to set permission to www-data e.g.

sudo chown www-data:www-data *.exe
```
