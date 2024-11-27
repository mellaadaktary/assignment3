# Assignment 3 part 1


## Overview
The goal of this assignment is to setup a Bash script that generates a static index.html file that contains some system information. The script will be configured to run at 05:00 using a `systemd` service and timer. The HTML document will be served with a nginx web server that will run on an Arch droplet along with a firewall using `ufw`.

## Requirements:

In order to follow along with this assignment you must have this git repository cloned:

```bash
git clone https://git.sr.ht/~nathan_climbs/2420-as2-start ~
```

Once this repository has been cloned to your home, you are now ready to begin.
# Task1
### System user 
This system user has to meet these requirements:
1. username: `webgen`
2. home directory: `/var/lib/webgen`
3. a login shell appropriate for a non-login user
4. ownership of the home directory and all of its sub-directories and files

Note: This might not be the best way to create this system user but this is how I have gone about it.


To create this system user run this command:
```bash
sudo useradd --system -s /usr/bin/nologin webgen
```

To confirm this user was created you need to check the contents of `/etc/passwd` this new system user should be at the bottom

the home directory for the user will be /home/webgen.
to change the home directory to whatever you want run usermod like so:
```bash
sudo usermod -d /var/lib/webgen -m webgen
```
This will change the current home directory to what is specified after the -d flag

now make the specified directory structure using the make directory command `mkdir`

```bash 
/var/lib/webgen/ 
├── bin/ 
│ 	└── generate_index 
└── HTML/ 
	└── index.html
```

Note: you will have to move the script called generate_index from the starter code we cloned into the new bin directory you make.

The final step would be to give ownership of all webgen files and directories to the webgen system user

```bash
chown -R webgen:webgen webgen/
```

we are creating and using systems users in this step because of security reasons, also system users are made for being assigned to system resources while regular users are made for people behind a computer 

# Task2

For this task we will need to create two files:
1. generate_index.service
2. generate_index.timer

you would need to move both of them to `/etc/systemd/systems` because that's where custom unit files go

we will first make the generate_index.service file. Inside this generate_index.service file we need to include the following components
```bash
[Unit]
Description=running the other file
Requires=network-online.target


[Service]
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

The important parts of these scripts are:
1. Requires=network-online.target : which indicates that it should not run until the network-online target is active.
2. User and Group = webgen : this part makes it so only user webgen and group webgen ---can run this----
3. finally ExectStart : this part indicates the path to the place where we have our script

once we have all this included into the file we need to make sure nothing is wrong with the script and start it to make sure it works

To check that the script is good run:
```bash
systemd-analyze verify generate_index.service
```
if this returns nothing then it means that you are good

we want to make sure our script works so we will run:
```bash
sudo systemctl start generate_index.service
```

After this command you can check the status of the service by running the exact same command but replacing the start with status.

check `/var/lib/webgen/HTML` to see if there is an index.html file, if there is that means it has been successful

Now we can make the timer file:

the timer file should look something like this:
```bash
[Unit]
Description=running the service hopefully

[Timer]
OnCalendar=*-*-* 05:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

the important components of this timer file consist of:
1. OnCalendar=*-*-* 05:00:00 : which indicates that it should run everyday at 5 o clock
2. WantedBy=timers.target : this indicates that all timers that should be active after boot will be

This timer file will control our service file made previously. Once everything look similar we can run the same commands to make sure everything looks the way it should

```bash
systemd-analyze verify generate_index.timer
```

We want this timer to start and be enabled so we will run this command

```bash
sudo systemctl enable --now generate_index.timer
```

This will activate the timer file and make it so that it starts from the beginning

To check the status of your timer or service run the following command:
```bash
sudo systemctl status generate_index.<timer or service>
```


Important: Once both of these are done we will need to run `sudo systemctl daemon-reload` , this command makes the system aware of the changes 
# Task3
1. For this task we need to edit the main `nginx.conf` file to ensure the server runs as the `webgen` user
2. create a separate server block that configures Nginx to server `index.html` on port 80

To edit the main `nginx.conf` file we need to got to `/etc/nginx` once in this directory run sudo with nvim and edit the file called `nginx.conf`.

change the user at the top to webgen since that is the user we want running this server
```bash
User webgen;  <----- change this
worker_processes 1;

#error_log logs/error.log;
```
save and quit the file

we can create the separate server block file that configures Nginx. make two directories `/etc/nginx/sites-available` and `/etc/nginx/sites-enabled` 

Create a file inside `sites-available`.
`/etc/nginx/sites-available/example.conf`
```bash
server {
        listen 80;
        listen [::]:80;

        server_name 159.223.194.99;

        root /var/lib/webgen/HTML;
        index index.html;

        location / {
        try_files $uri $uri/ =404;
        }
}
```
save and quit

This is what the seperate server block looks for this nginx server, The important components of this include:
1. listen 80 : which is the port
2. `listen [::]:80;` indicates ipv6 and the port for http which is 80
3. `root /var/lib/webgen/HTML;` the place being searched for the file specified in the next line `index index.html`
4. `location /` what happens when the user requests the project root
5. the rest is just error checking

Go to the `nginx.conf` file again and append `include sites-enabled/*` at the end of the `http` block like so:
```bash
http {
    include       mime.types;

    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    include sites-enabled/*; <-------
```
save and quit

To enable the site we need to simply create a symlink:
```bash
ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/example.conf
```

after this is done we can run **`sudo nginx -t`** to test the configuration file and references inside it.

the command is successful if you something like this:
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Finally we need to restart nginx.service to enable everything:

```bash
sudo systemctl restart nginx.service
```

Now everything should be running smoothly.

Why is it important to use a seperate server block file instead of modifying the main `nginx.conf` file directly?

# Task4
For this task we are going to configure and set up the `ufw` firewall 
we need to allow ssh connections and set a connection limit

First we have to install `ufw`
```bash
sudo pacman -S ufw
```

now that its installed we need to make sure THE SERVICE IS RUNNING NOT THE FIREWALL ITSELF.

to start the service and enable it we can run the following commands 
```bash
sudo systemctl start ufw.service
```

once this is successful we can run 
```bash
sudo systemctl enable ufw.service
```

to check that everything has been sucessful run 
```bash
sudo ufw status verbose
```
this command lets you see the status of the firewall, which in this case should be inactive 

WARNING: DO NOT ENABLE FIREWALL WITHOUT SSH ACCESS

now make sure you have permission to get past your firewall
```bash
sudo ufw allow SSH
```
and just for good caution also run 
```bash
sudo ufw allow 22
```
To limit ssh connections and spam we can run 
```bash
sudo ufw limit ssh
```

to make sure your have access run `sudo ufw status verbose` again and you should see a table with users who are allowed to get past the firewall.

now we can start the firewall:
```bash
sudo ufw enable
```
if everything has gone according to plan you should not lose access to your droplet and have access to everything still.

# Task5

![something](./views/something.png)




# References:
1. https://documentation.suse.com/smart/systems-management/html/systemd-working-with-timers/index.html 
2. https://man.archlinux.org/man/systemd-analyze.1
3. https://wiki.archlinux.org/title/Nginx
4. https://wiki.archlinux.org/title/Uncomplicated_Firewall
5. https://www.geeksforgeeks.org/how-to-run-nginx-for-root-non-root/
6. https://serverfault.com/questions/812584/in-systemd-whats-the-difference-between-after-and-requires
7. https://stackoverflow.com/questions/21983554/iptables-v1-4-14-cant-initialize-iptables-table-nat-table-does-not-exist-d
8. https://stackoverflow.com/questions/14972792/nginx-nginx-emerg-bind-to-80-failed-98-address-already-in-use
9. https://wiki.archlinux.org/title/Systemd/Timers