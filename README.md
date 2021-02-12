# Deploy Dart Server-side App on Ubuntu VPS:

### Create, Compile and Run Your Project On The Local Machine
1. Install the Dart SDK on your local machine by following the official docs here: https://dart.dev/get-dart
2. Create your project: `dart create your_project_name`
3. Compile your project in native binary using: `dart compile exe bin/your_project_name.dart` and you should get a file with   **.exe** extension (your_project_name.exe)
4. Confirm that your app is running by: `./your_project_name.exe` and visiting [http://localhost:PORT](http://localhost:PORT)

If everything is ok, then move on to the next part.

### On The VPS
1. Upload the **exe** file to your VPS. Let's say on `/home/user/projects` directory.
2. Optionally you can confirm that it's also working on your server by running `./your_project_name.exe` and visiting [http://your_vps_ip:PORT](http://your_vps_ip:PORT)*
#### Configure systemd Service
1. Create a systemd service by `sudo nano /etc/systemd/system/your_project_name.service` and pasting the following:

```
[Unit]
Description=My Dart Server App

[Service]
User=user
WorkingDirectory=/home/user/projects
ExecStart=/home/user/projects/your_project_name.exe
Restart=always

[Install]
WantedBy=multi-user.target
```
2. Enable the service: `sudo systemctl enable your_project_name`
3. Restart systemd daemon: `sudo systemctl daemon-reload`
4. Start the service: `sudo service your_project_name start`
5. Check if everything is ok and your service is running properly: `sudo service your_project_name status`

#### Configure nginx Server Block and Proxy Pass
1. Create a nginx server block by `sudo nano /etc/nginx/sites-available/your_project_name.conf ` and pasting the following:

```
server {
  listen 80;
  server_name your_domain_name;
  root /home/user/projects;

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_pass http://127.0.0.1:PORT;
    proxy_http_version 1.1;
  }
}

```
2. Enable the server block: `sudo ln -s /etc/nginx/sites-available/your_project_name.conf /etc/nginx/sites-enabled/your_project_name.conf`
3. Test your nginx configuration: `sudo nginx -t`
4. Restart nginx: `sudo service nginx reload`

That's it! Now you can visit `your_domain_name` to see your shiny new server-side dart app in full effect on the web. 
Additionally, you can secure your app using **Let's Encrypt SSL** using **certbot**: `sudo certbot -d your_domain`

Here's the live example: [http://getxserver.smj.xyz/](http://getxserver.smj.xyz/)

* You may need to change the file permission(`sudo chmod 777 your_project_name.exe`) and disable firewall or open the specified port.