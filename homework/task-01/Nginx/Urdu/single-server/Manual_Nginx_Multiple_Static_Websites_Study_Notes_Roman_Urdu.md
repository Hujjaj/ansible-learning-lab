# Manual Nginx Deployment: Rocky Linux 9 par Do Static Websites

Yeh study notes batate hain ke Ansible use kiye baghair `node1` par **Space Science** aur **Frozen Yogurt Shop** templates manually kaise deploy kiye jate hain.

## Fehrist (Table of Contents)

1. [Lab ka Maqsad](#1-lab-ka-maqsad)
2. [Manual Linux aur Ansible ka Muqabla](#2-manual-linux-aur-ansible-ka-muqabla)
3. [Lab Environment](#3-lab-environment)
4. [node1 se Connect Hona](#4-node1-se-connect-hona)
5. [Nginx Install aur Configure Karna](#5-nginx-install-aur-configure-karna)
6. [Templates Transfer aur Prepare Karna](#6-templates-transfer-aur-prepare-karna)
7. [Method 1: URL Subdirectories](#7-method-1-url-subdirectories)
8. [Method 1 ki Verification](#8-method-1-ki-verification)
9. [Method 2: Separate Hostnames](#9-method-2-separate-hostnames)
10. [Method 2 ki Verification](#10-method-2-ki-verification)
11. [Troubleshooting](#11-troubleshooting)
12. [Safe Cleanup](#12-safe-cleanup)
13. [Commands ka Khulasa](#13-commands-ka-khulasa)
14. [Aham Learning Points](#14-aham-learning-points)

---

## 1. Lab ka Maqsad

Ek Rocky Linux 9 server par yeh do static websites deploy karni hain:

1. Space Science
2. Frozen Yogurt Shop

Do hosting methods use honge:

| Method | Space Science | Frozen Yogurt Shop |
|---|---|---|
| URL subdirectories | `http://192.168.1.154/space-science/` | `http://192.168.1.154/frozen-yogurt/` |
| Separate hostnames | `http://space.nitclasses.com/` | `http://yogurt.nitclasses.com/` |

---

## 2. Manual Linux aur Ansible ka Muqabla

Manual commands se samajh aata hai ke Ansible background mein kin operating-system operations ko automate karta hai.

| Manual Linux command | Mutaliqa Ansible module |
|---|---|
| `dnf install` | `dnf` |
| `systemctl` | `service` ya `systemd` |
| `firewall-cmd` | `firewalld` |
| `mkdir`, `chmod`, `chown`, `rm` | `file` |
| `cp` ya `scp` | `copy` |
| `restorecon` | `command` |
| `curl` | `uri` |
| `ls`, `find`, `nginx -t` | `command` ya `shell` |

Manual commands operating-system ke steps samajhne ke liye mufeed hain. Ansible isi kaam ko multiple managed nodes par repeatable aur consistent banata hai.

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

> Server-side commands `node1` par run karein. Shuru ke `scp` commands us machine par run karein jahan ZIP files maujood hain.

---

## 4. node1 se Connect Hona

Control node ya kisi SSH client se connect karein:

```bash
ssh ansibleadmin@192.168.1.154
```

Root shell hasil karein:

```bash
sudo -i
```

Current server aur user confirm karein:

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

## 5. Nginx Install aur Configure Karna

### 5.1 Required packages install karein

```bash
dnf install nginx unzip policycoreutils-python-utils -y
```

| Package | Istemaal |
|---|---|
| `nginx` | Websites serve karta hai |
| `unzip` | Template ZIP archives extract karta hai |
| `policycoreutils-python-utils` | Persistent SELinux mappings ke liye `semanage` provide karta hai |

### 5.2 Nginx start aur enable karein

```bash
systemctl enable --now nginx
```

- `--now` Nginx ko foran start karta hai.
- `enable` Nginx ko system boot par automatically start karne ke liye configure karta hai.

Verify karein:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
nginx -t
```

### 5.3 Firewalld mein HTTP allow karein

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-services
```

Yeh incoming HTTP traffic ke liye TCP port 80 allow karta hai.

---

## 6. Templates Transfer aur Prepare Karna

### 6.1 Dono archives ko node1 par copy karein

Yeh commands us machine se run karein jahan ZIP files hain:

```bash
scp space-science.zip \
ansibleadmin@192.168.1.154:/tmp/

scp frozen-yogurt.zip \
ansibleadmin@192.168.1.154:/tmp/
```

### 6.2 Transferred files confirm karein

`node1` par run karein:

```bash
ls -lh /tmp/space-science.zip
ls -lh /tmp/frozen-yogurt.zip

file /tmp/space-science.zip
file /tmp/frozen-yogurt.zip
```

### 6.3 Archives inspect karein

```bash
unzip -l /tmp/space-science.zip | less
unzip -l /tmp/frozen-yogurt.zip | less
```

`less` se bahar aane ke liye `q` press karein.

### 6.4 Archives extract karein

```bash
mkdir -p /tmp/space-extracted
mkdir -p /tmp/yogurt-extracted

unzip /tmp/space-science.zip -d /tmp/space-extracted
unzip /tmp/frozen-yogurt.zip -d /tmp/yogurt-extracted
```

### 6.5 Har website ka homepage locate karein

```bash
find /tmp/space-extracted -name index.html -print
find /tmp/yogurt-extracted -name index.html -print
```

Woh directory use karein jisme deployable `index.html` ke saath CSS, JavaScript, images aur fonts ki directories bhi hon.

---

## 7. Method 1: URL Subdirectories

Dono websites same Nginx server block aur document root use karengi, lekin har website ki apni subdirectory hogi.

### 7.1 Destination directories create karein

```bash
mkdir -p /var/www/lawfirm.com/html/space-science
mkdir -p /var/www/lawfirm.com/html/frozen-yogurt
```

### 7.2 Space Science files copy karein

Space Science archive mein deployable files aam tor par `space-science/upload/` mein hoti hain:

```bash
cp -a /tmp/space-extracted/space-science/upload/. \
/var/www/lawfirm.com/html/space-science/
```

### 7.3 Frozen Yogurt files copy karein

Pehle uski `index.html` locate karein:

```bash
find /tmp/yogurt-extracted -name index.html -print
```

`/PATH/TO/FROZEN-YOGURT-SITE` ko actual directory se replace karein:

```bash
cp -a /PATH/TO/FROZEN-YOGURT-SITE/. \
/var/www/lawfirm.com/html/frozen-yogurt/
```

Sirf `index.html` copy na karein. Uski CSS, JavaScript, images aur fonts bhi copy hone chahiye.

### 7.4 Final layout confirm karein

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

### 7.5 Ownership aur permissions set karein

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

Directories ko execute permission chahiye taa-ke Nginx unke andar traverse kar sake. Static files ko aam tor par read permission chahiye.

### 7.6 SELinux contexts restore karein

```bash
restorecon -Rv /var/www/lawfirm.com/html/space-science
restorecon -Rv /var/www/lawfirm.com/html/frozen-yogurt
```

---

## 8. Method 1 ki Verification

### 8.1 HTTP status check karein

```bash
curl -I http://localhost/space-science/
curl -I http://localhost/frozen-yogurt/
```

Expected result:

```text
HTTP/1.1 200 OK
```

### 8.2 SELinux contexts check karein

```bash
ls -ldZ /var/www/lawfirm.com/html/space-science
ls -ldZ /var/www/lawfirm.com/html/frozen-yogurt
```

Output mein yeh context dekhein:

```text
httpd_sys_content_t
```

### 8.3 Browser se verify karein

```text
http://192.168.1.154/space-science/
http://192.168.1.154/frozen-yogurt/
```

---

## 9. Method 2: Separate Hostnames

Is method mein do Nginx server blocks use honge. Dono websites `192.168.1.154:80` share karengi, lekin Nginx HTTP `Host` header parh kar sahi website select karega.

### 9.1 Separate document roots create karein

```bash
mkdir -p /var/www/space/html
mkdir -p /var/www/yogurt/html
```

### 9.2 Prepared website files copy karein

```bash
cp -a /var/www/lawfirm.com/html/space-science/. \
/var/www/space/html/

cp -a /var/www/lawfirm.com/html/frozen-yogurt/. \
/var/www/yogurt/html/
```

### 9.3 Ownership aur permissions apply karein

```bash
chown -R root:root /var/www/space /var/www/yogurt

find /var/www/space -type d -exec chmod 755 {} \;
find /var/www/space -type f -exec chmod 644 {} \;

find /var/www/yogurt -type d -exec chmod 755 {} \;
find /var/www/yogurt -type f -exec chmod 644 {} \;
```

### 9.4 Persistent SELinux mappings configure karein

`/var/www/space` aur `/var/www/yogurt` custom document roots hain. In ke liye persistent SELinux file-context rules add karein:

```bash
semanage fcontext -a -t httpd_sys_content_t \
'/var/www/space(/.*)?'

semanage fcontext -a -t httpd_sys_content_t \
'/var/www/yogurt(/.*)?'
```

Mappings apply karein:

```bash
restorecon -Rv /var/www/space
restorecon -Rv /var/www/yogurt
```

Agar mapping pehle se maujood ho to `-a` ki jagah `-m` use karein:

```bash
semanage fcontext -m -t httpd_sys_content_t \
'/var/www/space(/.*)?'

semanage fcontext -m -t httpd_sys_content_t \
'/var/www/yogurt(/.*)?'
```

### 9.5 Space Science server block create karein

```bash
vim /etc/nginx/conf.d/space.conf
```

Yeh configuration add karein:

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

### 9.6 Frozen Yogurt server block create karein

```bash
vim /etc/nginx/conf.d/yogurt.conf
```

Yeh configuration add karein:

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

### 9.7 Nginx validate aur reload karein

Reload se pehle hamesha validation karein:

```bash
nginx -t
```

Successful test ke baad hi reload karein:

```bash
systemctl reload nginx
```

Verify karein:

```bash
systemctl is-active nginx
ss -tlnp | grep ':80'
```

### 9.8 Client hostname resolution configure karein

Windows computer par hosts file ko Administrator ke taur par edit karein:

```text
C:\Windows\System32\drivers\etc\hosts
```

Yeh entries add karein:

```text
192.168.1.154 space.nitclasses.com
192.168.1.154 yogurt.nitclasses.com
```

Windows DNS cache clear karein:

```powershell
ipconfig /flushdns
```

Linux ya WSL client par yehi entries is file mein add karein:

```text
/etc/hosts
```

---

## 10. Method 2 ki Verification

### 10.1 Host headers locally test karein

```bash
curl -I -H 'Host: space.nitclasses.com' \
http://127.0.0.1/

curl -I -H 'Host: yogurt.nitclasses.com' \
http://127.0.0.1/
```

Dono commands ko `HTTP/1.1 200 OK` return karna chahiye.

### 10.2 Browser se test karein

```text
http://space.nitclasses.com/
http://yogurt.nitclasses.com/
```

### 10.3 Nginx website kaise select karta hai

Browser in mein se ek HTTP header send karta hai:

```text
Host: space.nitclasses.com
```

```text
Host: yogurt.nitclasses.com
```

Nginx `Host` value ko `server_name` ke saath compare karta hai aur matching server block use karta hai.

---

## 11. Troubleshooting

### 11.1 `403 Forbidden`

Nginx error log check karein:

```bash
tail -n 30 /var/log/nginx/error.log
```

Homepage, permissions aur SELinux context check karein:

```bash
ls -lZ /var/www/space/html/index.html
namei -l /var/www/space/html/index.html
```

Common causes:

- `index.html` missing hai;
- kisi parent directory par execute permission nahi hai;
- SELinux context ghalat hai;
- configured Nginx `root` ghalat directory ko point kar raha hai.

### 11.2 `404 Not Found`

Confirm karein ke URL path filesystem path se match karta hai aur `index.html` expected top level par maujood hai.

```bash
find /var/www -name index.html -print
nginx -T | less
```

### 11.3 Ghalat website nazar aaye

Yeh cheezein check karein:

- client hosts file;
- browser cache;
- request ka `Host` header;
- Nginx `server_name` values;
- duplicate ya default server blocks.

```bash
nginx -T
```

### 11.4 Nginx reload fail ho

```bash
nginx -t
journalctl -u nginx --no-pager -n 30
```

Dobara reload karne se pehle configuration error correct karein.

### 11.5 Port 80 pehle se use ho raha ho

```bash
ss -tlnp | grep ':80'
```

Same address aur port par doosra web server start na karein, jab tak services ko jaan-boojh kar mukhtalif addresses ya ports use karne ke liye configure na kiya gaya ho.

---

## 12. Safe Cleanup

> Yeh commands listed lab directories ko permanently remove karti hain. Enter press karne se pehle har path verify karein.

### 12.1 Targets ko pehle preview karein

```bash
ls -ld /var/www/lawfirm.com/html/space-science
ls -ld /var/www/lawfirm.com/html/frozen-yogurt
ls -ld /var/www/space
ls -ld /var/www/yogurt
ls -l /etc/nginx/conf.d/space.conf
ls -l /etc/nginx/conf.d/yogurt.conf
```

### 12.2 Method 1 resources remove karein

```bash
rm -rf -- /var/www/lawfirm.com/html/space-science
rm -rf -- /var/www/lawfirm.com/html/frozen-yogurt
```

### 12.3 Method 2 resources remove karein

```bash
rm -rf -- /var/www/space
rm -rf -- /var/www/yogurt

rm -f -- /etc/nginx/conf.d/space.conf
rm -f -- /etc/nginx/conf.d/yogurt.conf
```

### 12.4 Persistent SELinux mappings remove karein

```bash
semanage fcontext -d '/var/www/space(/.*)?'
semanage fcontext -d '/var/www/yogurt(/.*)?'
```

Agar koi rule maujood na ho to `semanage` us rule ke liye error report kar sakta hai. Current custom mappings confirm karein:

```bash
semanage fcontext -l -C
```

### 12.5 Baqi configuration validate aur reload karein

```bash
nginx -t && systemctl reload nginx
```

`&&` ka matlab hai ke Nginx sirf us waqt reload hoga jab `nginx -t` successful ho.

### 12.6 Temporary files remove karein

```bash
rm -rf -- /tmp/space-extracted
rm -rf -- /tmp/yogurt-extracted
rm -f -- /tmp/space-science.zip
rm -f -- /tmp/frozen-yogurt.zip
```

### 12.7 Cleanup verify karein

```bash
test ! -e /var/www/space && echo "Space document root removed"
test ! -e /var/www/yogurt && echo "Yogurt document root removed"
nginx -t
```

---

## 13. Commands ka Khulasa

| Command | Maqsad |
|---|---|
| `scp` | SSH ke zariye archives server par copy karta hai |
| `file` | File ki type identify karta hai |
| `unzip -l` | Extract kiye baghair ZIP ke contents list karta hai |
| `unzip` | ZIP archive extract karta hai |
| `find` | Files locate karta aur file type ke mutabiq permissions apply kar sakta hai |
| `cp -a` | Directory structure preserve karte hue copy karta hai |
| `chown` | Ownership change karta hai |
| `chmod` | Permission bits change karta hai |
| `restorecon` | Expected SELinux context apply karta hai |
| `semanage fcontext` | Persistent SELinux file-context mapping define karta hai |
| `nginx -t` | Nginx configuration syntax validate karta hai |
| `systemctl reload nginx` | Nginx ko stop kiye baghair configuration reload karta hai |
| `curl -I` | HTTP response headers hasil karta hai |
| `ss -tlnp` | Listening TCP sockets aur processes dikhata hai |

---

## 14. Aham Learning Points

1. Ek Nginx server multiple static websites host kar sakta hai.
2. URL path ek document root ke neeche subdirectory se map ho sakta hai.
3. Named virtual hosts ek IP address aur port share kar sakte hain.
4. Nginx named virtual host select karne ke liye HTTP `Host` header use karta hai.
5. `index.html` aam tor par directory ka homepage hota hai.
6. Website assets ko wahi directory structure rakhna chahiye jiski HTML files ko zaroorat hai.
7. Linux permissions aur SELinux contexts dono ko Nginx ke liye content readable banana hota hai.
8. Nginx reload se pehle `nginx -t` successful hona chahiye.
9. Manual deployment se samajh aata hai ke Ansible modules kya automate karte hain.
10. Cleanup mein sirf lab ke banaye hue resources remove hone chahiye.
