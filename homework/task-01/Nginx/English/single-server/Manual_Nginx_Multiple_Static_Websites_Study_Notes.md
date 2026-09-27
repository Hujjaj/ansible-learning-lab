# Manual Nginx Deployment: Two Static Websites on Rocky Linux 9

These study notes show how to deploy the **Space Science** and **Frozen Yogurt Shop** templates manually on `node1`, without using Ansible.

## Table of Contents

1. [Lab Objective](#1-lab-objective)
2. [Manual Linux vs Ansible](#2-manual-linux-vs-ansible)
3. [Lab Environment](#3-lab-environment)
4. [Connect to node1](#4-connect-to-node1)
5. [Install and Configure Nginx](#5-install-and-configure-nginx)
6. [Transfer and Prepare the Templates](#6-transfer-and-prepare-the-templates)
7. [Method 1: URL Subdirectories](#7-method-1-url-subdirectories)
8. [Method 1 Verification](#8-method-1-verification)
9. [Method 2: Separate Hostnames](#9-method-2-separate-hostnames)
10. [Method 2 Verification](#10-method-2-verification)
11. [Troubleshooting](#11-troubleshooting)
12. [Safe Cleanup](#12-safe-cleanup)
13. [Command Summary](#13-command-summary)
14. [Key Learning Points](#14-key-learning-points)

---

## 1. Lab Objective

Deploy two static websites on one Rocky Linux 9 server:

1. Space Science
2. Frozen Yogurt Shop

Use two hosting methods:

| Method | Space Science | Frozen Yogurt Shop |
|---|---|---|
| URL subdirectories | `http://192.168.1.154/space-science/` | `http://192.168.1.154/frozen-yogurt/` |
| Separate hostnames | `http://space.nitclasses.com/` | `http://yogurt.nitclasses.com/` |

---

## 2. Manual Linux vs Ansible

The manual commands demonstrate the operations that Ansible normally automates.

| Manual Linux command | Related Ansible module |
|---|---|
| `dnf install` | `dnf` |
| `systemctl` | `service` or `systemd` |
| `firewall-cmd` | `firewalld` |
| `mkdir`, `chmod`, `chown`, `rm` | `file` |
| `cp` or `scp` | `copy` |
| `restorecon` | `command` |
| `curl` | `uri` |
| `ls`, `find`, `nginx -t` | `command` or `shell` |

Manual commands are useful for understanding the operating-system steps. Ansible makes the same work repeatable across multiple managed nodes.

---

## 3. Lab Environment

| Item | Value |
|---|---|
| Server | `node1` |
| IP address | `192.168.1.154` |
| Operating system | Rocky Linux 9 |
| Normal user | `ansibleadmin` |
| Web server | Nginx |
| HTTP port | `80` |
| Existing document root | `/var/www/lawfirm.com/html` |

> Run the server-side commands on `node1`. Run the initial `scp` commands on the machine where the ZIP files are stored.

---

## 4. Connect to node1

Connect from the control node or another SSH client:

```bash
ssh ansibleadmin@192.168.1.154
```

Become root:

```bash
sudo -i
```

Confirm the current server and user:

```bash
hostname
whoami
```

Expected values:

```text
node1
root
```

---

## 5. Install and Configure Nginx

### 5.1 Install the required packages

```bash
dnf install nginx unzip policycoreutils-python-utils -y
```

Package purposes:

| Package | Purpose |
|---|---|
| `nginx` | Serves the websites |
| `unzip` | Extracts template ZIP archives |
| `policycoreutils-python-utils` | Provides `semanage` for persistent SELinux mappings |

### 5.2 Start and enable Nginx

```bash
systemctl enable --now nginx
```

- `--now` starts Nginx immediately.
- `enable` configures Nginx to start automatically during boot.

Verify:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
nginx -t
```

### 5.3 Allow HTTP through firewalld

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-services
```

This allows incoming HTTP traffic on TCP port 80.

---

## 6. Transfer and Prepare the Templates

### 6.1 Copy both archives to node1

Run from the machine containing the ZIP files:

```bash
scp space-science.zip \
ansibleadmin@192.168.1.154:/tmp/

scp frozen-yogurt.zip \
ansibleadmin@192.168.1.154:/tmp/
```

### 6.2 Confirm the transferred files

Run on `node1`:

```bash
ls -lh /tmp/space-science.zip
ls -lh /tmp/frozen-yogurt.zip

file /tmp/space-science.zip
file /tmp/frozen-yogurt.zip
```

### 6.3 Inspect the archives

```bash
unzip -l /tmp/space-science.zip | less
unzip -l /tmp/frozen-yogurt.zip | less
```

Press `q` to leave `less`.

### 6.4 Extract the archives

```bash
mkdir -p /tmp/space-extracted
mkdir -p /tmp/yogurt-extracted

unzip /tmp/space-science.zip -d /tmp/space-extracted
unzip /tmp/frozen-yogurt.zip -d /tmp/yogurt-extracted
```

### 6.5 Locate each homepage

```bash
find /tmp/space-extracted -name index.html -print
find /tmp/yogurt-extracted -name index.html -print
```

Use the directory that contains the deployable `index.html` together with its CSS, JavaScript, image and font directories.

---

## 7. Method 1: URL Subdirectories

Both websites will use the same Nginx server block and document root, but each site will have a separate subdirectory.

### 7.1 Create the destination directories

```bash
mkdir -p /var/www/lawfirm.com/html/space-science
mkdir -p /var/www/lawfirm.com/html/frozen-yogurt
```

### 7.2 Copy the Space Science files

The known Space Science archive commonly stores the deployable files under `space-science/upload/`:

```bash
cp -a /tmp/space-extracted/space-science/upload/. \
/var/www/lawfirm.com/html/space-science/
```

### 7.3 Copy the Frozen Yogurt files

First locate its `index.html`:

```bash
find /tmp/yogurt-extracted -name index.html -print
```

Then replace `/PATH/TO/FROZEN-YOGURT-SITE` with the actual directory:

```bash
cp -a /PATH/TO/FROZEN-YOGURT-SITE/. \
/var/www/lawfirm.com/html/frozen-yogurt/
```

Do not copy only `index.html`. Its CSS, JavaScript, images and fonts must also be copied.

### 7.4 Confirm the final layout

```bash
ls -l /var/www/lawfirm.com/html/space-science/index.html
ls -l /var/www/lawfirm.com/html/frozen-yogurt/index.html
```

Required mapping:

```text
http://192.168.1.154/space-science/
                      ↓
/var/www/lawfirm.com/html/space-science/index.html
```

```text
http://192.168.1.154/frozen-yogurt/
                      ↓
/var/www/lawfirm.com/html/frozen-yogurt/index.html
```

### 7.5 Set ownership and permissions

```bash
chown -R root:root /var/www/lawfirm.com/html/space-science
chown -R root:root /var/www/lawfirm.com/html/frozen-yogurt

find /var/www/lawfirm.com/html/space-science \
-type d -exec chmod 755 {} \;

find /var/www/lawfirm.com/html/space-science \
-type f -exec chmod 644 {} \;

find /var/www/lawfirm.com/html/frozen-yogurt \
-type d -exec chmod 755 {} \;

find /var/www/lawfirm.com/html/frozen-yogurt \
-type f -exec chmod 644 {} \;
```

Directories require the execute permission so Nginx can traverse them. Static files normally require read permission.

### 7.6 Restore SELinux contexts

```bash
restorecon -Rv /var/www/lawfirm.com/html/space-science
restorecon -Rv /var/www/lawfirm.com/html/frozen-yogurt
```

---

## 8. Method 1 Verification

### 8.1 Check the HTTP status

```bash
curl -I http://localhost/space-science/
curl -I http://localhost/frozen-yogurt/
```

Expected result:

```text
HTTP/1.1 200 OK
```

### 8.2 Check the SELinux contexts

```bash
ls -ldZ /var/www/lawfirm.com/html/space-science
ls -ldZ /var/www/lawfirm.com/html/frozen-yogurt
```

Look for:

```text
httpd_sys_content_t
```

### 8.3 Browser verification

```text
http://192.168.1.154/space-science/
http://192.168.1.154/frozen-yogurt/
```

---

## 9. Method 2: Separate Hostnames

This method uses two Nginx server blocks. Both websites share `192.168.1.154:80`, but Nginx selects the website by reading the HTTP `Host` header.

### 9.1 Create separate document roots

```bash
mkdir -p /var/www/space/html
mkdir -p /var/www/yogurt/html
```

### 9.2 Copy the prepared website files

```bash
cp -a /var/www/lawfirm.com/html/space-science/. \
/var/www/space/html/

cp -a /var/www/lawfirm.com/html/frozen-yogurt/. \
/var/www/yogurt/html/
```

### 9.3 Apply ownership and permissions

```bash
chown -R root:root /var/www/space /var/www/yogurt

find /var/www/space -type d -exec chmod 755 {} \;
find /var/www/space -type f -exec chmod 644 {} \;

find /var/www/yogurt -type d -exec chmod 755 {} \;
find /var/www/yogurt -type f -exec chmod 644 {} \;
```

### 9.4 Configure persistent SELinux mappings

The paths under `/var/www/space` and `/var/www/yogurt` are custom document roots. Add persistent SELinux file-context rules:

```bash
semanage fcontext -a -t httpd_sys_content_t \
'/var/www/space(/.*)?'

semanage fcontext -a -t httpd_sys_content_t \
'/var/www/yogurt(/.*)?'
```

Apply the mappings:

```bash
restorecon -Rv /var/www/space
restorecon -Rv /var/www/yogurt
```

If a mapping already exists, modify it with `-m` instead of adding it with `-a`:

```bash
semanage fcontext -m -t httpd_sys_content_t \
'/var/www/space(/.*)?'

semanage fcontext -m -t httpd_sys_content_t \
'/var/www/yogurt(/.*)?'
```

### 9.5 Create the Space Science server block

```bash
vim /etc/nginx/conf.d/space.conf
```

Add:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name space.nitclasses.com;

    root /var/www/space/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 9.6 Create the Frozen Yogurt server block

```bash
vim /etc/nginx/conf.d/yogurt.conf
```

Add:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name yogurt.nitclasses.com;

    root /var/www/yogurt/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 9.7 Validate and reload Nginx

Always validate before reloading:

```bash
nginx -t
```

Reload only after a successful test:

```bash
systemctl reload nginx
```

Verify:

```bash
systemctl is-active nginx
ss -tlnp | grep ':80'
```

### 9.8 Configure client hostname resolution

On the Windows computer running the browser, edit this file as Administrator:

```text
C:\Windows\System32\drivers\etc\hosts
```

Add:

```text
192.168.1.154 space.nitclasses.com
192.168.1.154 yogurt.nitclasses.com
```

Flush the Windows DNS cache:

```powershell
ipconfig /flushdns
```

For a Linux or WSL client, add the same entries to:

```text
/etc/hosts
```

---

## 10. Method 2 Verification

### 10.1 Test the Host headers locally

```bash
curl -I -H 'Host: space.nitclasses.com' \
http://127.0.0.1/

curl -I -H 'Host: yogurt.nitclasses.com' \
http://127.0.0.1/
```

Both commands should return `HTTP/1.1 200 OK`.

### 10.2 Test through the browser

```text
http://space.nitclasses.com/
http://yogurt.nitclasses.com/
```

### 10.3 How Nginx selects the website

The browser sends one of these headers:

```text
Host: space.nitclasses.com
```

```text
Host: yogurt.nitclasses.com
```

Nginx compares the `Host` value with `server_name` and uses the matching server block.

---

## 11. Troubleshooting

### 11.1 `403 Forbidden`

Check the Nginx error log:

```bash
tail -n 30 /var/log/nginx/error.log
```

Check the homepage, permissions and SELinux context:

```bash
ls -lZ /var/www/space/html/index.html
namei -l /var/www/space/html/index.html
```

Common causes:

- `index.html` is missing;
- a parent directory lacks execute permission;
- the SELinux context is incorrect;
- the configured Nginx `root` points to the wrong directory.

### 11.2 `404 Not Found`

Confirm that the URL path matches the filesystem path and that `index.html` is located at the expected top level.

```bash
find /var/www -name index.html -print
nginx -T | less
```

### 11.3 Wrong website appears

Check:

- the client hosts file;
- browser cache;
- the request's `Host` header;
- Nginx `server_name` values;
- duplicate or default server blocks.

```bash
nginx -T
```

### 11.4 Nginx reload fails

```bash
nginx -t
journalctl -u nginx --no-pager -n 30
```

Correct the configuration error before attempting another reload.

### 11.5 Port 80 is already in use

```bash
ss -tlnp | grep ':80'
```

Do not start a second web server on the same address and port unless the services are intentionally configured to use different addresses or ports.

---

## 12. Safe Cleanup

> These commands permanently remove the listed lab directories. Verify every path before pressing Enter.

### 12.1 Preview the targets

```bash
ls -ld /var/www/lawfirm.com/html/space-science
ls -ld /var/www/lawfirm.com/html/frozen-yogurt
ls -ld /var/www/space
ls -ld /var/www/yogurt
ls -l /etc/nginx/conf.d/space.conf
ls -l /etc/nginx/conf.d/yogurt.conf
```

### 12.2 Remove Method 1 resources

```bash
rm -rf -- /var/www/lawfirm.com/html/space-science
rm -rf -- /var/www/lawfirm.com/html/frozen-yogurt
```

### 12.3 Remove Method 2 resources

```bash
rm -rf -- /var/www/space
rm -rf -- /var/www/yogurt

rm -f -- /etc/nginx/conf.d/space.conf
rm -f -- /etc/nginx/conf.d/yogurt.conf
```

### 12.4 Remove persistent SELinux mappings

```bash
semanage fcontext -d '/var/www/space(/.*)?'
semanage fcontext -d '/var/www/yogurt(/.*)?'
```

If a rule does not exist, `semanage` may report an error for that rule. Confirm the current custom mappings with:

```bash
semanage fcontext -l -C
```

### 12.5 Validate and reload the remaining configuration

```bash
nginx -t && systemctl reload nginx
```

The `&&` means Nginx reloads only if `nginx -t` succeeds.

### 12.6 Remove temporary files

```bash
rm -rf -- /tmp/space-extracted
rm -rf -- /tmp/yogurt-extracted
rm -f -- /tmp/space-science.zip
rm -f -- /tmp/frozen-yogurt.zip
```

### 12.7 Verify cleanup

```bash
test ! -e /var/www/space && echo "Space document root removed"
test ! -e /var/www/yogurt && echo "Yogurt document root removed"
nginx -t
```

---

## 13. Command Summary

| Command | Purpose |
|---|---|
| `scp` | Copies archives to the server through SSH |
| `file` | Identifies a file's type |
| `unzip -l` | Lists a ZIP archive without extracting it |
| `unzip` | Extracts a ZIP archive |
| `find` | Locates files and applies permissions by file type |
| `cp -a` | Copies directories while preserving their structure |
| `chown` | Changes ownership |
| `chmod` | Changes permission bits |
| `restorecon` | Applies the expected SELinux context |
| `semanage fcontext` | Defines persistent SELinux file-context mappings |
| `nginx -t` | Validates Nginx configuration syntax |
| `systemctl reload nginx` | Reloads Nginx without a full stop and start |
| `curl -I` | Retrieves HTTP response headers |
| `ss -tlnp` | Shows listening TCP sockets and processes |

---

## 14. Key Learning Points

1. A single Nginx server can host multiple static websites.
2. A URL path can map to a subdirectory under one document root.
3. Named virtual hosts can share an IP address and port.
4. Nginx uses the HTTP `Host` header to select a named virtual host.
5. `index.html` normally acts as the homepage of a directory.
6. Website assets must remain in the directory structure expected by the HTML files.
7. Linux permissions and SELinux contexts must both allow Nginx to read the content.
8. `nginx -t` should always succeed before Nginx is reloaded.
9. Manual deployment helps explain what Ansible modules automate.
10. Cleanup should remove only the resources created for the lab.
