# TOR-Linux-HiddenService
How to host a site on TOR (dark web)

***Build TOR from source***
```
https://github.com/scriptzteam/TOR-Linux-Build
```

***All these steps are for nginx, if you want to run apache2 do NOT forget to disable mod status and edit the security.conf file.***
```
a2dismod status

Module status disabled.
To activate the new configuration, you need to run:
systemctl restart apache2

nano /etc/apache2/conf-available/security.conf
|--> ServerTokens Prod 
|--> ServerSignature Off
|--> TraceEnable Off
```

***The Hidden Service (nginx server)***

***We need to edit the Tor configuration file to enable our hidden service. First we will make a backup of this configuration file.***
```
sudo cp /etc/tor/torrc /etc/tor/OLD.torrc
```

***Then edit the configuration file.***
```
sudo nano /etc/tor/torrc
```

***By default all Tor client services, relays, and hidden services are commented out and disabled. Let’s active the hidden service. Find the section for hidden services. It will look something like this.***
```
############### This section is just for location-hidden services ###
## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.
#HiddenServiceDir /var/lib/tor/hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServiceDir /var/lib/tor/other_hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServicePort 22 127.0.0.1:22
```

***Uncomment the following lines.***
```
#HiddenServiceDir /var/lib/tor/hidden_service/
#HiddenServicePort 80 127.0.0.1:80
```

***The hidden services section should now look like this.***
```
############### This section is just for location-hidden services ###
## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
#HiddenServiceDir /var/lib/tor/other_hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServicePort 22 127.0.0.1:2
```

***Restart Tor***
```
sudo service tor restart
```

***And check the Tor status***
```
sudo systemctl status tor
```

***Couple of files should have generated by Tor. First is a hostname file. Open it up to get your .onion address.***
```
sudo cat /var/lib/tor/hidden_service/hostname
```

***Generated file contained SOMELONGSTRING.onion. Your file should contain something similar. The other file is a private and public key. Open it up and take a look.***
```
sudo ls -lrt /var/lib/tor/hidden_service/
```

***With these files two files you can move your server to a new machine if eventually necessary. Copy these file and keep them secure.***

***Nginx is a good web server for this project. Install Nginx.***
```
sudo apt install nginx
```

***Edit the main Nginx configuration file to disable undesirable information sharing.***
```
sudo nano /etc/nginx/nginx.conf
```

***Inside the http block add the following***
```
server_name_in_redirect off;
server_tokens off;
port_in_redirect off;
```

***Then restart the Nginx server.***
```
sudo systemctl restart nginx
```

***Make a directory to hold our files for the web server.***
```
sudo mkdir /var/www/dark_web
```

***Make and edit an index.html file for your site.***
```
sudo vi /var/www/dark_web/index.html
```

***Inside just put anything. We don’t need actual html, just something kinda unique for right now.***

***Welcome to your dark web page***

***Set the permissions so that Nginx can access the files.***
```
sudo chmod 755 /var/www/dark_web
```

***Remove the default site.***
```
sudo rm /etc/nginx/sites-enabled/default
sudo rm /etc/nginx/sites-available/default
```

***Make a new site in the sites-available directory.***
```
sudo nano /etc/nginx/sites-available/dark_web
```

***Inside add the following replacing the root and server_name values for your instance.***
```
server {
	listen 127.0.0.1:80;
	root /var/www/dark_web/;
	index index.html;
	server_name SOMELONGSTRING.onion;
}
```

***Add this site the the site_enabled.***
```
sudo ln -s /etc/nginx/sites-available/dark_web /etc/nginx/sites-enabled/
```

***Then restart the Nginx server.***
```
sudo systemctl restart nginx
```

***If you want to run php (on nginx) then there is simple sample config, replace the php7.X with your php version ( you can get version by running php-v command)***
```
server {
    listen 127.0.0.1:80;
    server_name SOMELONGSTRING.onion;

    root /var/www/dark_web/;
    index index.php index.html index.htm;
    server_name_in_redirect off;
    server_tokens off;
    port_in_redirect off;

    location / {
        allow 127.0.0.1;
        deny all;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.X-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
