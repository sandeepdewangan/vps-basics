# Virtual Private Server (VPS)

A **VPS** stands for **Virtual Private Server**.

It’s a type of hosting where a single physical server is divided into multiple “virtual” servers. Each VPS acts like its own independent machine.

**STEPS to Followed**

1. Connect to Server with OpenSSH
2. Update Server
3. Domain Setup
4. Nginx Web Server Setup

### In simple terms:

- Imagine one powerful computer split into several smaller ones
- Each smaller “slice” runs its own operating system
- You get dedicated resources (CPU, RAM, storage)
- You have more control than shared hosting

## I. Connecting to Server with SSH

**SH (Secure Shell)** is a protocol used to **securely connect to another computer/server over a network (usually the internet)**. 

**Generate Key**

`ssh-keygen`

**Connect to VPS**

`ssh root@ipaddress -i private_key_file_path`
` ssh ubuntu@13.60.173.81 -i C:\Users\sd\.ssh\ubuntu-vps-ec2.pem`

**Improving SSH Access Flow**

Under `C:\Users\sd\.ssh` create  a `config` file.

```
ServerAliveInterval 120
ServerAliveCountMax 3

Host aws_vps
    HostName 13.60.173.81
    User ubuntu
    IdentityFile C:\Users\sd\.ssh\ubuntu-vps-ec2.pem
```

**Now connect to the VPS**

`ssh aws_vps`

**Exit to disconnect**
`exit`

## II. Update VM

`apt update`

`apt upgrade`

## III. Linking Domain to VPS

Add domain `concretesstudio.com` to our VPS provider.

1. Go to VPS provider networking tab and add domain.
2. If using DigitalOcean VPS, get its Name Server (NS) details and add to the domain provider NS records.
3. Add Subdomain `test`.
4. Check on `whatsmydns.net`. it performs DNS lookup to check a domain name's current IP address and DNS record information against multiple nameservers located in different parts of the world.

DNA for `concretesstudio.com`

| Type  | Hostname                 | Value               |
| ----- | ------------------------ | ------------------- |
| A     | concretesstudio.com      | 134.209.153.196     |
| CNAME | www.concretesstudio.com  | concretesstudio.com |
| CNAME | test.concretesstudio.com | concretesstudio.com |

## IV. Nginx Web Server Setup

### Install

`sudo apt install nginx`

Check status 

`sudo systemctl status nginx.service`

**Open port for Web server**

`sudo ufw app list`

`sudo ufw allow "Nginx Full"`

`sudo ufw status verbose`

Now open http://concretesstudio.com/ and it will be directed to our VPS and default page will be displayed.

### Nginx Configurations

Global Nginx Config File

`/etc/nginx/nginx.conf`

Access and Error Log Files

`/var/log/nginx`

Sites Enabled

`/etc/nginx/sites-enabled/`

sites-enabled points to the sites available. It is just a symbolic link.

Sites Available

`/etc/nginx/sites-available/default`



### Configure First Website (concretesstudio.com)

First remove the symbolic link of default website

`sudo rm /etc/nginx/sites-enabled/default`

`sudo nginx -t`

`sudo systemctl reload nginx`

`cd /etc/nginx/sites-available/`

`sudo cp default concretesstudio.com`

`sudo nano concretesstudio.com`

```
server {
        listen 80 default_server; # Only ONE website can be the default server.
        listen [::]:80 default_server;

        root /var/www/concretesstudio.com;
        index index.html;
        server_name concretesstudio.com;
        ...
      }
```

Create a symbolic link to point our domain

`sudo ln -s /etc/nginx/sites-available/concretesstudio.com /etc/nginx/sites-enabled/`

```
 concretesstudio.com -> /etc/nginx/sites-available/concretesstudio.com
```

`sudo nginx -t`

`sudo systemctl reload nginx`

Create Index File

`sudo mkdir /var/www/concretesstudio.com`

`sudo nano index.html`

Now this index file will be served.



### Configure First Sub Domain (test.concretesstudio.com)

`sudo cp default test.concretesstudio.com`

`sudo nano test.concretesstudio.com`

```
server { 
    listen 80; 
    listen [::]:80; 
    root /var/www/test.concretesstudio.com;
    index index.html; 
    server_name test.concretesstudio.com;         
    ...  
    }
```

Create a symbolic link to point our domain

`sudo ln -s /etc/nginx/sites-available/test.concretesstudio.com /etc/nginx/sites-enabled`

Test and Reload

Create a folder `test.concretesstudio.com` under `/var/www/` and file name "index.html"

This index.html will be served when  `test.concretesstudio.com` is visited.



## V. MySQL Server Setup

**Install**

`sudo apt install mysql-server`

`sudo mysql_secure_installation`





<hr/>

## 



## Basic Commands

### systemctl

**status, start, stop, restart, reload**

`systemctl status unattended-upgrades`

`systemctl stop unattended-upgrades`

`systemctl start unattended-upgrades`

`systemctl restart unattended-upgrades`

`systemctl reload unattended-upgrades`

Reload does not stops the service, it just reloads the config file of the service.

### Reboot

`reboot`



### SSH Connection from VPS to Third Party

From Server Terminal

`ssh-keygen -t rsa`

Start the SSH Agent

`eval "$(ssh-agent -s)"`

Add Identity to use key

`ssh-add .ssh/id_rsa`

Now any third party apps can be verified using this RSA key.



**Example:**

Connect to Github, go to github and add SSH key by adding public key of server.

Get the keys `cat .ssh/id_rsa.pub`

Adding Github

`ssh -T git@github.com`

On Success:

Hi sandeepdewangan! You've successfully authenticated.



## Cron Jobs

A **cron job** is a scheduled task on Unix/Linux systems that runs commands or scripts automatically at specified times.

The quick and simple editor for cron schedule.

https://crontab.guru/

Open Crontab

`crontab -e`

Edit file add this line

`* * * * * date >> /root/time.txt`

To disable cron job, just comment or delete the line.



## Managing Users

**Create a user**

`adduser sandeep`

A group sandeep and user sandeep is create with specified password.

**Delete User**

`deluser sandeep --remove-home`

**Logging to VPS with new user**

Generate SSH Key for new user

`ssh-keygen -t rsa`

Add Public key to VPS for new user using root user

The authorized_keys contains all the keys

`cat .ssh/authorized_keys`

To Add keys for new user, go to the home directory of new user.

`mkdir .ssh`

`cd .ssh`

`nano authorized_keys`

paste the ssh public key of new user.

Change also the config file of client system.

Login using or config file

`ssh sandeep@ipaddress -i privatekey`

**Run task as super user**

Add new user to the sudo group

`adduser sandeep sudo`

Now sandeep user can act as a root user

`sudo apt update`



## Adding a Security Layer on VPS Server

**1. Preventing root user access**

`sudo nano /etc/ssh/sshd_config`

Make Changes

```
PermitRootLogin no

# restrict the use of password for any user. Use ssh key instead. If ssh key is lost, 
# reset from the console of server provider.
PasswordAuthentication no
ChallengeResponseAuthentication no

```

After that root cannot be logges in even with valid SSH key.



**2. Enable UFW Firewall**

Uncomplicated Firewall

Check status `sudo ufw status` or `sudo ufw status verbose`

lists all application profiles known to **UFW** `sudo ufw app list`

Allow the connections `sudo ufw allow OpenSSH`

Enable UFW `sudo ufw enable`



**3. Permissions**

❌ Not a good way

`sudo chmod 777 -R test`

 Now test folder has all the permissions to RWX to all users.

✔️ Correct way, Change user

`sudo chown -R sandeep test/` 



**4. Fail2Ban**

Block intruders

`sudo apt install fail2ban`

Get to the installed location

`cd /etc/fail2ban`

Open config file , this is the original file

`sudo nano jail.conf`

Create a location jail file

`sudo nano jail.local`

```
[DEFAULT]
bantime = 3h
maxretry = 3

[sshd]
enabled = true
```

`sudo systemctl restart fail2ban.service`

`sudo fail2ban-client status`

O/p: Nos of jail: 1, sshd

sshd Status

`sudo fail2ban-client status sshd`

Unban IP

`sudo fail2ban-client set sshd unbanip 192.12.2.2`
