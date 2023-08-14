# ASP.NET Core on Linux
So you're looking to host your asp.net application on a Linux server. In this documentation we will go through the steps when setting up and using it.

## Table of contents
* [Prerequisites](#prerequisites)
* [Configuring Nginx](#configuring-nginx)
* [Setting up app service](#setting-up-app-service)
* [Pointing domain to server](#pointing-domain-to-server)
* [Setting up Cloudflare](#setting-up-cloudflare)


# Preqrequisites
We'll need to install a couple of things to set up the environment we need.

* Nginx
```console
sudo apt update
sudo apt install nginx
```
* .NET Core
```console
sudo apt install aspnetcore-runtime-6.0
```

# Configuring Nginx
Lets start by editing out default config through nano.
```console
sudo nano /etc/nginx/sites-available/default
```

Here we will remove everything and paste this in (make sure to change example.com to your domain or delete if none).
```nginx
server {
    listen        80;
    server_name   example.com *.example.com;
    
    location / {
        proxy_pass         http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
```
Then press ``Ctrl + S`` to save and ``Ctrl + X`` to close.

# Setting up app service
This is an essential step to allow our ASP.NET instance to run indefinitely.
Lets start off by creating a new app service. Run this in your console.
```console
sudo nano /etc/systemd/system/MyApp.service
```
After entering the editor, paste this into the file.
```yaml
[Unit]
Description=My webapp
[Service]
WorkingDirectory=/var/www/WorkingDirectory
ExecStart=/usr/bin/dotnet /var/www/WorkingDirectory/MyApp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
SyslogIdentifier=mvcnew
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```
Now save by pressing ``Ctrl + S``, then ``Ctrl + X`` to close.

Next, we'll enable the newly created service.
```console
sudo systemctl enable MyApp.service
```

Then finally, we'll start it.
```console
sudo systemctl start MyApp.service
```

If you wan't to check the status of the app while its running, you can use the status command.
```console
sudo systemctl status MyApp.service
```

# Pointing domain to server
This is a rather simple task and should be an option universally.\
Go to your domain provider, find your record manager and add this record:\
```
Record: A
Host: @
Value: SERVER IP
```
After this, wait for DNS propagation to complete. This may take several hours.

# Setting up Cloudflare
> Todo.
