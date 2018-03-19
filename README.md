** This is an updated version of the original documentation: https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-for-apache

## Install nginx
To start off, we need to install and configure nginx which will serve the front end of our site.

Let’s download it from apt-get:

```
sudo apt-get install nginx
```

Once it has downloaded, you can go ahead and configure the virtual host to run on the front end.

There are a few changes we need to make in the configuration.

Configure nginx

Open up the nginx configuration.

```
sudo nano /etc/nginx/sites-available/awesome
```

The following configuration will set you up to use nginx as the front end server. It is very similar to the default set up, and the details are under the configuration.

```
server {
        listen   80;

        root /var/www/;
        index index.php index.html index.htm;

        server_name awesome.com;

        location / {
        try_files $uri $uri/ /index.php;
        }

        location ~ \.php$ {

        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:8080;

         }

         location ~ /\.ht {
                deny all;
        }
}
```

The following changes were implemented in the configuration:

The root was set to the correct web directory
index.php was added on the index line
try_files attempts to serve whatever page the visitor requests. If nginx is unable, then the file is passed to the proxy
proxy_pass lets nginx the address of the proxied server
Finally the "location ~ /\.ht {" location block denies access to .htaccess files, if Apache's document root concurs with nginx's one
This configuration sets up a system where all extensions with a php ending are rerouted to the apache backend which will run on port 8080.

Activate the virtual host.

```
sudo ln -s /etc/nginx/sites-available/awesome /etc/nginx/sites-enabled/awesome
```

Additionally, delete the default nginx server block.

```
sudo rm /etc/nginx/sites-enabled/default
```

The next step is to install and configure apache.

## Install Apache
With nginx taken care of, it’s time to install our backend, apache.

```
sudo apt-get install apache2
```

Since nginx is still not turned on, Apache will start running on port 80.

Configure Apache
We need to configure apache to take over the backend, which as we told nginx, will be running on port 8080. Open up the apache ports file to start setting apache on the correct port:

```
sudo nano /etc/apache2/ports.conf
```

Find and change the following lines to have apache running on port 8080, accessible only from the localhost:

```
NameVirtualHost 127.0.0.1:8080
Listen 127.0.0.1:8080
```

Save and Exit.

Subsequently, open up a new virtual host file, copying the layout from the default apache file:

```
sudo cp /etc/apache2/sites-available/default /etc/apache2/sites-available/awesome
sudo nano /etc/apache2/sites-available/awesome
```

The main issue that needs to be addressed here is that the virtual host needs to be, once again, running on port 8080 (instead of the default 80 given to nginx).

The line should look like this:

```
<VirtualHost 127.0.0.1:8080>
```

Make sure your Document Root is correct. Save and exit the file and activate that virtual host:

```
sudo a2ensite awesome
```

Before we start testing anything out, we need to equip apache with php. Go ahead and install it now:

```
sudo apt-get install php
```

Restart both servers to make the changes effective:

```
sudo service apache2 restart
sudo service nginx restart
```

Finish Up
We have set up the VPS with nginx running on the front end of our site and apache processing php on the back end. Loading our domain will take us to our site’s default page.

We can check that information is being routed to apache is working by running a common php script.

Go ahead and create the php.info file:

sudo nano /var/www/info.php
Paste the following lines into that file:

<?
phpinfo( );
?>
Save and exit.
