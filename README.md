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

What is the benefit of creating a system user for this task rather than using a regular user or root?

# Task2

How will you verify that the timer is active and the service runs successfully? What commands can you run to check logs and to confirm the service's execution

# Task3

Why is it important to use a seperate server block file instead of modifying the main `nginx.conf` file directly?

How can you check the status of the nginx services and test your nginx configuration?
# Task4

How can you check the status of your firewall?


# Task5
![[Pasted image 20241123162405.png]]




# References:
