# Ansible Study Notes — Roman Urdu

## Part 2: Project Directory aur `ansible.cfg`

Yeh notes `automation` working directory ke andar banayi gayi Ansible project configuration ko explain karte hain.

## 1. Project Directory

Project directory banayein aur us ke andar jayein:

```bash
mkdir -p ~/automation
cd ~/automation
```

Apni current location confirm karein:

```bash
pwd
```

Expected path:

```text
/home/ansibleadmin/automation
```

Planned directory structure yeh hai:

```text
automation/
├── ansible.cfg
├── inventory
└── playbooks/
```

- `ansible.cfg` mein project-specific Ansible settings hoti hain.
- `inventory` mein managed nodes ke names, IP addresses aur groups honge.
- `playbooks/` mein YAML automation playbooks honge.

## 2. `ansible.cfg` Banayein

`automation` directory ke andar se:

```bash
vim ansible.cfg
```

Neeche wali configuration add karein:

```ini
[defaults]
inventory = ./inventory
host_key_checking = False
remote_user = ansibleadmin
ask_pass = False
private_key_file = /home/ansibleadmin/.ssh/ansible-key

[privilege_escalation]
become_method = sudo
become_user = root
become_ask_pass = False
```

```ini
[defaults]

# Inventory file containing the managed nodes
inventory = ./inventory

# Verify managed nodes' SSH host keys for secure connections
host_key_checking = True

# Default user Ansible will use to connect to managed nodes through SSH
remote_user = ansibleadmin

# Do not ask for the SSH login password; use SSH key authentication
ask_pass = False


[privilege_escalation]

# Use sudo when privilege escalation is enabled
become_method = sudo

# Escalate privileges to the root user
become_user = root

# Do not ask for the sudo password; requires NOPASSWD sudo configuration
become_ask_pass = False
```

## 3. Configuration Syntax

`ansible.cfg` file INI-style syntax use karti hai.

### Section Headers

Section ka naam square brackets ke andar likha jata hai:

```ini
[defaults]
```

```ini
[privilege_escalation]
```

### Key-Value Settings

Har setting aam tor par is structure ko follow karti hai:

```ini
key = value
```

Example:

```ini
remote_user = ansibleadmin
```

### Comments

Jo line `#` se start ho woh comment hoti hai:

```ini
# This is a comment
```

Comments configuration ko explain karte hain, lekin Ansible unhein execute nahi karta.

## 4. `[defaults]` Section

`[defaults]` section Ansible ke general behavior ko control karta hai.

### Inventory Location

```ini
inventory = ./inventory
```

Yeh Ansible ko batata hai ke isi project directory mein `inventory` naam ki file use karo.

Kyun ke inventory path configuration mein set hai, is liye yeh short command use ki ja sakti hai:

```bash
ansible all -m ping
```

Is setting ke baghair inventory ko manually specify karna padega:

```bash
ansible all -i inventory -m ping
```

### SSH Host-Key Checking

```ini
host_key_checking = True
```

Yeh Ansible ko managed node ki SSH identity verify karne ko kehta hai. Is se kisi fake ya unexpectedly changed host se chup-chaap connection hone ka risk kam hota hai.

Jab pehli dafa kisi naye managed node se connect karein, trust manually establish karein:

```bash
ssh ansibleadmin@MANAGED_NODE_IP
```

Fingerprint ko verify karein aur sirf tab `yes` type karein jab server sahi ho.

Temporary private lab mein host-key checking disable ki ja sakti hai:

```ini
host_key_checking = False
```

Lekin is ko enabled rakhna zyada secure aur real-world practice ke qareeb hai.

### Remote SSH User

```ini
remote_user = ansibleadmin
```

Ansible har managed node se `ansibleadmin` user ke taur par connect karega. Is liye yeh account har managed node par mojood hona chahiye.

Conceptually yeh is ke barabar hai:

```bash
ssh ansibleadmin@MANAGED_NODE_IP
```

Command line se user ko override bhi kiya ja sakta hai:

```bash
ansible all -m ping -u another_user
```

### SSH Password Prompt

```ini
ask_pass = False
```

Ansible SSH login password nahi poochega. Is configuration mein control node aur managed nodes ke darmiyan SSH key authentication expected hai.

Agar temporary tor par password authentication use karni ho:

```bash
ansible all -m ping --ask-pass
```

## 5. `[privilege_escalation]` Section

Ansible pehle managed node se `ansibleadmin` ke taur par connect hota hai. Phir privileged task `sudo` use karke `root` ke taur par execute ho sakta hai.

```text
Control node
     |
     | SSH
     v
ansibleadmin on managed node
     |
     | sudo / become
     v
root privileges
```

### Become Method

```ini
become_method = sudo
```

Yeh privilege escalation ke liye `sudo` method select karta hai.

### Become User

```ini
become_user = root
```

Jab privilege escalation request ki jaye, Ansible task ko `root` ke taur par execute karega.

Yeh setting target user select karti hai, lekin apne aap privilege escalation enable nahi karti.

### Become Password Prompt

```ini
become_ask_pass = False
```

Ansible sudo password request nahi karega. Is ke liye har managed node par is tarah ka `NOPASSWD` rule chahiye:

```text
ansibleadmin ALL=(ALL) NOPASSWD: ALL
```

Agar passwordless sudo configured nahi hai to `-K` se become password request karein:

```bash
ansible all -b -K -m command -a "whoami"
```

## 6. SSH Password vs Become Password

Yeh settings authentication ke do mukhtalif stages ko control karti hain:

| Setting | Maqsad |
| --- | --- |
| `ask_pass` | Managed node se connect hone ke liye SSH login password |
| `become_ask_pass` | Connect hone ke baad privilege escalation ke liye sudo password |

Is lab mein:

```text
ask_pass = False
```

kyun ke SSH keys use hongi, aur:

```text
become_ask_pass = False
```

kyun ke `ansibleadmin` ke paas `NOPASSWD` sudo access hoga.

## 7. `become = True` Global Kyun Nahi Hai

Configuration mein jaan-boojh kar yeh setting nahi rakhi gayi:

```ini
become = True
```

Is se har task automatically root privileges ke saath run nahi hota. Privilege escalation sirf jab zaroorat ho tab request karni chahiye.

Ad-hoc command mein is ko `-b` ke saath enable karein:

```bash
ansible all -b -m command -a "whoami"
```

Ya playbook mein enable karein:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present
```

Is tareeqe se clearly pata chalta hai ke kin commands ya playbooks ko administrative access chahiye.

## 8. Configuration Validate Karein

`~/automation` ke andar se check karein ke Ansible ne konsi configuration file select ki:

```bash
ansible --version
```

Expected line:

```text
config file = /home/ansibleadmin/automation/ansible.cfg
```

Selected configuration file display karein:

```bash
ansible-config view
```

Woh settings dikhayein jo Ansible defaults se different hain:

```bash
ansible-config dump --only-changed
```

Inventory banane ke baad us ka structure display karein:

```bash
ansible-inventory --graph
```

Tamam managed nodes ko test karein:

```bash
ansible all -m ping
```

## 9. Important Configuration-File Behavior

Ansible **sirf ek** configuration file select karta hai. Yeh neeche di gayi locations ko order mein check karta hai aur pehli valid file milte hi ruk jata hai:

| Priority | Location | Wazahat |
| --- | --- | --- |
| 1 — Highest | `$ANSIBLE_CONFIG` | Configuration path jo environment variable ke taur par explicitly export ki gayi ho |
| 2 | `./ansible.cfg` | Present working directory (`pwd`) mein configuration file |
| 3 | `~/.ansible.cfg` | Current user ki home directory mein hidden configuration file |
| 4 — Lowest | `/etc/ansible/ansible.cfg` | System-wide default configuration file |

Is liye preference order yeh hai:

```text
$ANSIBLE_CONFIG
       ↓
./ansible.cfg in PWD
       ↓
~/.ansible.cfg
       ↓
/etc/ansible/ansible.cfg
```

### Priority 1: Export ki Hui `ANSIBLE_CONFIG` Value

Aap explicitly Ansible ko bata sakte hain ke konsi configuration file use karni hai:

```bash
export ANSIBLE_CONFIG=/home/ansibleadmin/automation/ansible.cfg
```

Exported value check karein:

```bash
echo "$ANSIBLE_CONFIG"
```

Is ki priority sab se high hai. Jab tak yeh variable set hai, Ansible doosri locations search karne ke bajaye isi file ko use karta hai.

Jab zaroorat na rahe to current shell se exported value remove karein:

```bash
unset ANSIBLE_CONFIG
```

### Priority 2: Present Working Directory Mein Configuration

`pwd` ka matlab **present working directory** hai. Is se check karein:

```bash
pwd
```

Agar result yeh ho:

```text
/home/ansibleadmin/automation
```

Ansible phir yahan dekhta hai:

```text
/home/ansibleadmin/automation/ansible.cfg
```

Hamari lab mein yehi method use ho raha hai. Project-level `ansible.cfg` har project ko apni inventory aur apna behavior rakhne deta hai.

Security note: Agar current directory world-writable ho to Ansible wahan ki `ansible.cfg` ko ignore kar sakta hai, kyun ke koi doosra user malicious configuration rakh sakta hai.

### Priority 3: User ki Home Directory Mein Configuration

Tilde `~` current user ki home directory ko represent karta hai:

```bash
echo "$HOME"
```

`ansibleadmin` ke liye aam tor par yeh output aata hai:

```text
/home/ansibleadmin
```

Is liye:

```text
~/.ansible.cfg
```

ka matlab hai:

```text
/home/ansibleadmin/.ansible.cfg
```

Yeh configuration user par tab apply hoti hai jab koi higher-priority configuration file select na ho.

### Priority 4: System-Wide Configuration

Aakhri location hai:

```text
/etc/ansible/ansible.cfg
```

Yeh system-wide configuration hai. Ansible is ko sirf tab use karta hai jab pehli teen locations mein se kisi se configuration file na mile.

### Selected Configuration Confirm Karein

Jab bhi confirm karna ho ke konsi configuration file active hai, `ansible --version` run karein.

```bash
ansible --version
```

Is line ko dekhein:

```text
config file = /home/ansibleadmin/automation/ansible.cfg
```

Aap yeh bhi use kar sakte hain:

```bash
ansible-config view
ansible-config dump --only-changed
```

Agar aap `automation` directory se bahar chale jayein aur Ansible ko project configuration na mile, to wapas is directory mein aa jayein:

```bash
cd ~/automation
```

## Final Configuration

```ini
[defaults]
inventory = ./inventory
host_key_checking = True
remote_user = ansibleadmin
ask_pass = False
private_key_file = /home/ansibleadmin/.ssh/ansible-key

[privilege_escalation]
become_method = sudo
become_user = root
become_ask_pass = False
```

## Agla Step

Agla step managed nodes ko prepare karna aur `~/automation` ke andar `inventory` file banana hai.
