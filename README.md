# ACIT 2420 - Assignment 3: Nginx

## Introduction 

This repository will create and enable a .html file of syste minformation to automatically run everyday at 05:00 using a systemd service and timer. 

## Table of Contents

- [Assignment Instructions](#assignment-instructions)
    - [Task 1: System User Creation and Ownership](#task-1-system-user-creation-and-ownership)
    - [Task 2: Create .service and .timer Scripts](#task-2-create-service-and-timer-scripts)
    - [Task 3: Modify nginx.conf](#task-3-modify-nginxconf)
    - [Task 4: UFW](#task-4-setup-ufw)
    - [Task 5: Example Screenshot](#script-1-1-installation---installing-packages)
- [References](#References)

## Assignment Instructions

### Task 1: System User Creation and Ownership
---

**STEP 1**: Type the command to create a system user:

```bash
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

- ```-r```: Creates a system user.
- ```-d```: Sets the home directory.
- ```-s```: Sets login shell for user.
- ```-s /usr/sbin/nologin```: Creates a no login interaction. 

**STEP 2**: Create the Home Directory

Enter the below command to create the home directory ```/vib/lib/webgen/```:
```bash
sudo mkdir -p /var/lib/webgen
```

**STEP 3**: Create Subdirectory Files

Enter the below command to create the subdirectory ```bin``` and ```HTML```:
```bash
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
```

**STEP 4**: Clone Required Files

Enter the below command to clone Nathan Serkin's repository containing the necessary ```generate_index``` script into the working directory:
```bash
git clone https://git.sr.ht/~nathan_climbs/2420-as2-start
```

**STEP 5**: Move File, ```generate_index``` and Grant Necessary Permission

Enter the below command to move file ```generate_index``` to ```/var/lib/webgen/bin```:
```bash
sudo mv 2420-as2-start/generate_index /var/lib/webgen/bin/
```

Enter the below command to grant executable permission to ```generate_index```:
```bash
sudo chmod +x /var/lib/webgen/bin_generate_index
```

**STEP 6**: Set Ownership

Enter the below command to set ownership to webgen:
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```
**STEP 7**: Create ```index.html``` 

Enter the below command to create file ```index.html```:
```bash
sudo nvim /var/lib/webgen/HTML/index.html
```

### Task 2: Create .service and .timer Scripts
---

**STEP 1**: Create ```generate-index.service```

Enter the following command to create and immediately prompt an edit of created file, ```generate-index.service```:
```bash
sudo nvim /etc/systemd/system/generate-index.service
```

Copy and paste the following script to ```generate-index.service```:
```ini
[Unit]
Description=Generate Index Service
Wants=network-online.target
After=network-online.target

[Service]
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

> [!NOTE]
> A [Install] is not required as it is triggered by a `generate-index.timer` we will create. 


**STEP 2**: Create `generate-index.timer`

Enter the following command to create and immediately prompt an edit of created file, `generate-index.timer`:
```bash
sudo nvim /etc/systemd/system/generate-index.timer
```

Copy and paste the following script to `generate-index.timer` for it to run everyday at 05:00:
```ini
[Unit]
Description=Timer for Generate Index Service

[Timer]
OnCalendar=*-*-* 05:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```
> [!IMPORTANT]
> Test Services Prior to Proceeding

**STEP 3**: Test `generate-index.service`

Enter the following command to start `generate-index.service`:
```bash
sudo systemctl start generate-index.service
```

Enter the following command to check status of `generate-index.service`:
```bash
sudo systemctl status generate-index.service
```

Enter the following command to view the logs of the service `generate-index.service`:
```bash
sudo journalctl -u generate-index.service
```

### Task 3: Modify nginx.conf
---

**STEP 1**: Install nginx

Enter the following code to install package, `nginx`:
```bash
sudo pacman -S nginx
```

**STEP 2**: Modify `user` in `nginx.conf` using the `sudo nvim`

Replace the user line with the following code:
```bash
user webgen;
```

> [!NOTE]
> If you only specify user, it will default the primary group of the user.

**STEP 3**: Prime for Separate Server Block

Use approach `sites-enabled` and `sites-available` and create directories:
```bash
sudo mkdir -p /etc/nginx/sites-enabled
sudo mkdir -p /etc/ngxin/sites-available
```
>[!NOTE]
> Use `-p` to create parent directory or errors will occur.

**STEP 4**: Create the Separate Server Block in `sites-available`:

Enter the following command to create a new server block to host index.html:
```bash
sudo nvim /etc/nginx/sites-available/webgen.conf
```
Enter the following script into `webgen.conf`:
```ini
server {
    listen 80;
    listen [::]:80;

    server_name localhost.webgen;

    root /var/lib/webgen/HTML;
    index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
}
```

**STEP 5**: Enable Server Block

Enter the following command to symlink and enable server block:
```bash
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/ 
```

**STEP 6**: Update `nginx.conf` with `sites-enabled`

Enter the following code to access `nginx.conf`
```bash
sudo nvim /etc/nginx/sites-enabled/nginx.conf
```

Enter the below script to `nginx.conf`
```bash
http {
    ...
    include /etc/nginx/sites-enabled/*;
}
```

**STEP 7**: Test nginx Configuration

Enter the below command to test for success and errors:
```bash
sudo nginx -t
```

**STEP 8**: Restart and Start nginx

Enter the below command to restart, start, and check status of nginx:
```bash
sudo systemctl restart nginx
sudo systemctl start ngxinx
sudo systemctl status nginx
```

### Task 4: Setup UFW
---

**STEP 1**: Install UFW

Enter the following code to install package, `ufw`:
```bash
sudo pacman -S ufw
```

**STEP 2**: Allow SSH and HTTP

Enter the following command to allow SSH connection:
```bash
sudo ufw allow ssh
```

Enter the following command to allow HTTP traffic:
```bash
sudo ufw allow http
```