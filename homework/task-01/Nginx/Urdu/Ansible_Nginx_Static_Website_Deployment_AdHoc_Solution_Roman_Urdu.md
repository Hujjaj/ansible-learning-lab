# Ansible Ad-Hoc Lab: Nginx par Static Website Deploy Karein — Roman Urdu

Yeh revised solution un commands aur paths ke mutabiq hai jo Rocky Linux lab mein successfully kaam kar chuke hain. Pehle `node1` (`192.168.1.154`) par test karein, phir tamam managed nodes par deploy karein.

## Fehrist

1. [Lab Environment](#1-lab-environment)
2. [Sahi Document Root Ka Faisla](#2-sahi-document-root-ka-faisla)
3. [Prechecks](#3-prechecks)
4. [Nginx Install Aur Start Karein](#4-nginx-install-aur-start-karein)
5. [Shuru Ka 403 Error Diagnose Karein](#5-shuru-ka-403-error-diagnose-karein)
6. [Website Archive Inspect Karein](#6-website-archive-inspect-karein)
7. [Website Deploy Karein](#7-website-deploy-karein)
8. [Deployment Verify Karein](#8-deployment-verify-karein)
9. [Teenon Nodes Par Deployment](#9-teenon-nodes-par-deployment)
10. [Idempotency](#10-idempotency)
11. [Troubleshooting](#11-troubleshooting)
12. [Complete Cleanup](#12-complete-cleanup)
13. [Ahm Learning Points](#13-ahm-learning-points)

---

## 1. Lab Environment

| Item | Value |
|---|---|
| Control user | `ansibleadmin` |
| Project directory | `/home/ansibleadmin/automation` |
| Inventory group | `three_tier_app` |
| Pilot host | `node1` |
| node1 IP | `192.168.1.154` |
| Operating system | Rocky Linux 9 |
| Active virtual host | `lawfirm.com` |
| Active document root | `/var/www/lawfirm.com/html` |
| Website archive | `/tmp/space-science.zip` |
| Extraction directory | `/tmp/space-science-extracted` |

```bash
cd /home/ansibleadmin/automation
```

---

## 2. Sahi Document Root Ka Faisla

Rocky Linux mein aam default Nginx root `/usr/share/nginx/html` hota hai. Lekin is lab ka active `lawfirm.com` server block yeh root use karta hai:

```text
/var/www/lawfirm.com/html
```

Active configuration ko priority milti hai, is liye website isi path mein deploy hogi. Agar root maloom na ho:

```bash
ansible node1 -b -m shell -a \
"nginx -T 2>/dev/null | grep -E '^[[:space:]]*root[[:space:]]'"
```

---

## 3. Prechecks

```bash
ansible --version
ansible-config dump --only-changed
ansible-inventory --graph
ansible three_tier_app --list-hosts
ansible three_tier_app -m ping
```

Expected hosts `node1`, `node2`, aur `node3` hain. Har host se `SUCCESS` aur `pong` milna chahiye.

---

## 4. Nginx Install Aur Start Karein

### Step 1: Pilot host par Nginx aur unzip install karein

```bash
ansible node1 -b -m dnf -a "name=nginx,unzip state=present"
```

`state=present` missing package install karta hai. Repository mein naya build ho to DNF purane build ko current build se replace bhi kar sakta hai.

### Step 2: Nginx start aur enable karein

```bash
ansible node1 -b -m service -a "name=nginx state=started enabled=yes"
```

- `state=started`: service abhi running ho.
- `enabled=yes`: reboot ke baad automatically start ho.

### Step 3: Current state verify karein

```bash
ansible node1 -m command -a "systemctl is-active nginx"
```

Expected result `active` hai. `service` output mein kabhi purana status snapshot bhi hota hai; yeh command current state seedha check karti hai.

### Step 4: Firewall mein HTTP allow karein

```bash
ansible node1 -b -m firewalld -a \
"service=http permanent=yes immediate=yes state=enabled"
```

`changed=false` ka matlab HTTP pehle se allowed tha. Yeh sahi idempotent behavior hai.

### Step 5: Website root test karein

```bash
ansible node1 -m uri -a "url=http://localhost status_code=200"
```

Pehle test mein `403 Forbidden` mila. Nginx running tha lekin active document root mein index page nahi tha.

---

## 5. Shuru Ka 403 Error Diagnose Karein

```bash
ansible node1 -b -m command -a "ls -laZ /usr/share/nginx/html"
ansible node1 -b -m command -a "tail -n 20 /var/log/nginx/error.log"
```

Important error yeh tha:

```text
directory index of "/var/www/lawfirm.com/html/" is forbidden
```

Is se asal active document root maloom hua. Woh khaali tha:

```bash
ansible node1 -b -m command -a "ls -laZ /var/www/lawfirm.com/html"
```

Temporary test page banayein:

```bash
ansible node1 -b -m copy -a \
'content="<h1>Welcome to lawfirm.com</h1>\n<p>Deployed using Ansible.</p>\n" dest=/var/www/lawfirm.com/html/index.html owner=root group=root mode=0644'
```

SELinux context restore karke dobara test karein:

```bash
ansible node1 -b -m command -a "restorecon -Rv /var/www/lawfirm.com/html"
ansible node1 -m uri -a \
"url=http://localhost status_code=200 return_content=yes"
```

Status `200` prove karta hai ke Nginx, active root, permissions aur SELinux access sahi hain.

---

## 6. Website Archive Inspect Karein

Archive pehle se na ho to download karein:

```bash
ansible node1 -m get_url -a \
"url=https://freewebsitetemplates.com/download/space-science/ dest=/tmp/space-science.zip mode=0644"
```

Extract karne se pehle inspect karein:

```bash
ansible node1 -m stat -a "path=/tmp/space-science.zip"
ansible node1 -m command -a "file /tmp/space-science.zip"
ansible node1 -m command -a "unzip -l /tmp/space-science.zip"
```

Successful lab mein valid ZIP taqreeban 26 MB thi. Asal website yahan thi:

```text
space-science/upload/
```

Main page `space-science/upload/index.html` thi. License aur design-source folders Nginx ko required nahi hain.

---

## 7. Website Deploy Karein

### Step 1: Temporary extraction directory banayein

```bash
ansible node1 -b -m file -a \
"path=/tmp/space-science-extracted state=directory mode=0755"
```

### Step 2: Archive node1 par extract karein

```bash
ansible node1 -b -m unarchive -a \
"src=/tmp/space-science.zip dest=/tmp/space-science-extracted remote_src=yes creates=/tmp/space-science-extracted/space-science/upload/index.html"
```

- `remote_src=yes`: archive managed node par pehle se hai.
- `creates=`: marker file ho to unnecessary extraction repeat nahi hogi.

### Step 3: Sirf website contents copy karein

```bash
ansible node1 -b -m copy -a \
"src=/tmp/space-science-extracted/space-science/upload/ dest=/var/www/lawfirm.com/html/ remote_src=yes owner=root group=root mode=preserve"
```

`upload/` ke aakhir ka trailing slash bohat important hai. Yeh directory banane ke bajaye us ke **contents** document root mein copy karta hai. Temporary test page bhi real template page se replace ho jata hai.

### Step 4: SELinux contexts restore karein

```bash
ansible node1 -b -m command -a \
"restorecon -Rv /var/www/lawfirm.com/html"
```

Lab mein copied files ka `user_tmp_t` context web-readable `httpd_sys_content_t` mein badla.

### Step 5: Index verify karein

```bash
ansible node1 -m stat -a "path=/var/www/lawfirm.com/html/index.html"
ansible node1 -b -m command -a "ls -laZ /var/www/lawfirm.com/html"
```

---

## 8. Deployment Verify Karein

```bash
ansible node1 -m uri -a "url=http://localhost status_code=200"
```

Successful response mein `status: 200`, `server: nginx/1.20.1`, `content_length: 3538`, aur `changed: false` mila.

[URI Explanation]

Browser mein kholein:

```text
http://192.168.1.154/
```

Is deployment mein `/space-science/` use na karein. Contents seedha document root mein copy hue hain, is liye website `/` se serve hoti hai. Browser ka **Not secure** message expected hai kyun ke yeh HTTP lab hai, HTTPS nahi.

```bash
ansible node1 -m command -a "systemctl is-active nginx"
ansible node1 -m shell -a "ss -tln | grep ':80 '"
ansible node1 -m uri -a "url=http://localhost status_code=200"
```

---

## 9. Teenon Nodes Par Deployment

Pehle confirm karein ke sab nodes `/var/www/lawfirm.com/html` use karte hain. Phir:

```bash
ansible three_tier_app -b -m dnf -a "name=nginx,unzip state=present"
ansible three_tier_app -b -m service -a "name=nginx state=started enabled=yes"
ansible three_tier_app -b -m firewalld -a "service=http permanent=yes immediate=yes state=enabled"
ansible three_tier_app -m get_url -a "url=https://freewebsitetemplates.com/download/space-science/ dest=/tmp/space-science.zip mode=0644"
ansible three_tier_app -b -m file -a "path=/tmp/space-science-extracted state=directory mode=0755"
ansible three_tier_app -b -m unarchive -a "src=/tmp/space-science.zip dest=/tmp/space-science-extracted remote_src=yes creates=/tmp/space-science-extracted/space-science/upload/index.html"
ansible three_tier_app -b -m copy -a "src=/tmp/space-science-extracted/space-science/upload/ dest=/var/www/lawfirm.com/html/ remote_src=yes owner=root group=root mode=preserve"
ansible three_tier_app -b -m command -a "restorecon -Rv /var/www/lawfirm.com/html"
ansible three_tier_app -m uri -a "url=http://localhost status_code=200"
```

Browser URLs:

```text
http://192.168.1.154/
http://192.168.1.185/
http://192.168.1.190/
```

---

## 10. Idempotency

| Operation | Second run par expected result |
|---|---|
| `dnf state=present` | Packages sahi hon to `changed=false` |
| `service state=started enabled=yes` | Already running aur enabled ho to `changed=false` |
| `firewalld state=enabled` | HTTP pehle se allowed ho to `changed=false` |
| `get_url` | Remote file same ho to aam tor par `changed=false` |
| `unarchive creates=...` | Marker ho to extraction skip |
| `copy` | Contents match hon to `changed=false` |
| `uri` | Verification; aam tor par `changed=false` |

`command` module read-only command chala kar bhi aksar `CHANGED` dikhata hai. Is ka matlab zaroori nahi ke host modify hua hai.

---

## 11. Troubleshooting

### HTTP 403 Forbidden

```bash
ansible node1 -b -m command -a "tail -n 20 /var/log/nginx/error.log"
ansible node1 -b -m command -a "ls -laZ /var/www/lawfirm.com/html"
ansible node1 -b -m shell -a "nginx -T 2>/dev/null | grep -E '^[[:space:]]*root[[:space:]]'"
```

403 aam tor par missing index, directory permissions, ya ghalat SELinux context ki wajah se aata hai.

### `/space-science/` par 404

`http://192.168.1.154/` use karein. Site document root par install hui hai.

### Port 80 pehle se use ho

```bash
ansible node1 -b -m shell -a "ss -tlnp | grep ':80 '"
```

### ZIP asal mein HTML ho

```bash
ansible node1 -m command -a "file /tmp/space-science.zip"
```

Output ZIP archive data hona chahiye.

### Nginx configuration validate karein

```bash
ansible node1 -b -m command -a "nginx -t"
```

---

## 12. Complete Cleanup

Yeh commands pilot host ko completely reset karte hain. Custom site aur Nginx configuration delete hogi, is liye sirf full cleanup ke waqt use karein.

```bash
# Nginx stop aur disable karein
ansible all -b -m service -a "name=nginx state=stopped enabled=no"

# Nginx packages remove karein
ansible all -b -m dnf -a "name=nginx,nginx-core,nginx-filesystem state=absent"

# Optional: unzip sirf is lab ke liye tha to remove karein
ansible all -b -m dnf -a "name=unzip state=absent"

# Custom site aur remaining Nginx configuration remove karein
ansible all -b -m file -a "path=/var/www/lawfirm.com state=absent"
ansible all -b -m file -a "path=/etc/nginx state=absent"

# Temporary deployment files remove karein
ansible all -b -m file -a "path=/tmp/space-science-extracted state=absent"
ansible all -b -m file -a "path=/tmp/space-science.zip state=absent"

# HTTP firewall rule sirf tab remove karein jab doosri site ko port 80 na chahiye
ansible all -b -m firewalld -a "service=http permanent=yes immediate=yes state=disabled"
```

Cleanup verify karein:

```bash
ansible all -m command -a "rpm -q nginx nginx-core nginx-filesystem"
ansible all -m stat -a "path=/var/www/lawfirm.com"
ansible all -m stat -a "path=/tmp/space-science.zip"
ansible all -m stat -a "path=/tmp/space-science-extracted"
ansible all -b -m shell -a "ss -tlnp | grep ':80 ' || true"
```

Expected: packages `not installed`, removed paths par `exists: false`, aur port 80 par Nginx listen na kare.

`node1` ko `three_tier_app` se sirf tab replace karein jab kisi managed node par doosri required website na chal rahi ho.

---

## 13. Ahm Learning Points

1. Default root assume karne ke bajaye active Nginx configuration inspect karein.
2. `403` ka matlab server ne jawab diya lekin content serve nahi kar saka.
3. Unknown archive extract karne se pehle inspect karein.
4. Sirf asal website wali directory deploy karein.
5. `copy` source ka trailing slash directory aur us ke contents mein farq karta hai.
6. SELinux system par `/tmp` se copy ke baad `restorecon` chalayein.
7. Tamam nodes se pehle `node1` par pilot test karein.
8. Service status ke saath `uri` se HTTP bhi verify karein.
9. Cleanup ko exact lab paths tak limit rakhein.
10. Tested ad-hoc workflow ko repeatable automation ke liye playbook mein convert karein.
