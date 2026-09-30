# Linux Repositories aur RHCSA Repository Configuration

Shuru se samajhne ke liye combined study notes — RHEL 9 / Rocky Linux 9

Attached `RHCSA_Project_02_software-repositories(1).md` ke useful topics, corrected commands aur real-job scenario shamil hain.

## Topic index

1. [Package kya hai?](#package-kya-hai)
2. [Repository kya hai?](#repository-kya-hai)
3. [YUM aur DNF](#yum-aur-dnf)
4. [Dependencies aur metadata](#dependencies-aur-metadata)
5. [BaseOS aur AppStream](#baseos-aur-appstream)
6. [Local repository](#local-repository)
7. [EPEL repository](#epel-repository)
8. [yum.repos.d directory](#yumreposd-directory)
9. [Repo file ko samajhna](#repo-file-ko-samajhna)
10. [RHCSA practice sawal](#rhcsa-practice-sawal)
11. [Configuration ke qadam](#configuration-ke-qadam)
12. [Verification aur package installation](#verification-aur-package-installation)
13. [Troubleshooting](#troubleshooting)
14. [Jaldi revision](#jaldi-revision)
15. [Practice sawalat aur jawab](#practice-sawalat-aur-jawab)
16. [Mirrorlist aur baseurl ka farq](#mirrorlist-aur-baseurl-ka-farq)
17. [Rocky repo settings ki tafseel](#rocky-repo-settings-ki-tafseel)
18. [Package kis repo mein hai](#package-kis-repo-mein-hai)
19. [Eight repo ka doosra lab example](#eight-repo-ka-doosra-lab-example)
20. [DNF cache commands](#dnf-cache-commands)
21. [Network checks ki tartib](#network-checks-ki-tartib)
22. [Real job scenario aur rollback](#real-job-scenario-aur-rollback)
23. [Completion checklist aur review](#completion-checklist-aur-review)
24. [References](#references)

## Package kya hai?

**Package aik software hota hai jo installation ke liye tayyar kiya gaya ho.** Is mein program ki files aur installation ki maloomat hoti hain. RHEL aur Rocky Linux mein package files aam tor par `.rpm` par khatam hoti hain.

Misalein:

- `nginx`: web server.
- `git`: source code ki tabdeeliyon ka record rakhta hai.
- `htop`: system ki activity dikhata hai.

Package install karne se uski files system ki munasib jagahon par rakhi jati hain. Sirf RPM download karna installation nahi hota.

## Repository kya hai?

**Repository, ya repo, aik jagah hai jahan software packages aur unki maloomat yani metadata mojood hota hai.** Package manager yahan se software hasil karke install ya update karta hai.

Isay kiryana store ki misaal se samjhein:

| Linux ka lafz | Kiryana store ki misaal |
|---|---|
| Repository | Store |
| Package | Store mein rakhi hui cheez |
| Metadata | Cheezon ki fehrist aur tafseel |
| DNF/YUM | Madadgar jo cheez dhoond kar lata hai |
| Repo configuration | Store ka pata aur istemal ki hidayat |

Yahan repo se murad **software package repository** hai. Git repository mein project files aur unki version history hoti hai; woh is lafz ka aik doosra istemal hai.

## YUM aur DNF

**YUM aur DNF package managers hain.** Inka kaam packages install, update aur remove karna hai.

DNF, YUM ke baad aane wala package manager hai. RHEL 9 mein `yum` command compatibility ke liye DNF ke sath di jati hai.

```bash
sudo dnf install nginx
sudo dnf update nginx
sudo dnf remove nginx
```

| Command | Kaam |
|---|---|
| `install` | Software install karna |
| `update` | Installed software update karna |
| `remove` | Software hatana |

Yeh commands system mein tabdeeli karti hain. Lab task ke mutabiq zaroorat par chalayein.

**Yaad rakhein: repo software rakhta hai; DNF usay install karta hai.**

## Dependencies aur metadata

**Dependencies:** woh doosre packages jin ki kisi program ko chalne ke liye zaroorat hoti hai. DNF inhein dhoondta hai aur installation mein shamil karta hai.

**Metadata:** repository ki fehrist aur tafseel, jaise package names, versions aur dependencies. Metadata khud application nahi hota.

Aam RPM repository mein yeh file metadata ka aik aham shuruati point hoti hai:

```text
repodata/repomd.xml
```

Jab aap package install karte hain, DNF aam tor par:

1. Repository configuration parhta hai.
2. Metadata check karta hai.
3. Package aur uski dependencies dhoondta hai.
4. Zaroori packages hasil karta hai.
5. Configuration ke mutabiq verification karta hai.
6. Packages install karta hai.

Pehle se cache mein mojood metadata ya packages dobara istemal bhi ho sakte hain.

## BaseOS aur AppStream

| Repository | Bunyadi kaam |
|---|---|
| BaseOS | Operating system ke bunyadi packages |
| AppStream | Mazeed applications, runtimes aur tools |

Dono RHEL ki software distribution ke standard hissay hain. `.repo` file mein naam likhne se packages ki qisam tay nahi hoti. **Asal packages us URL par hotay hain jis ki taraf entry ishara karti hai.**

## Local repository

**Local repo aapki apni machine ya organization ke internal network par mojood software store hota hai.**

Companies isay packages ki controlled distribution aur limited internet walay environments ke liye istemal karti hain.

| Jagah | Sirf misaal ke liye address |
|---|---|
| Isi machine ki directory | `file:///home/student/myrepo` |
| Mount ki hui installation media | `file:///mnt/rhel9/BaseOS` |
| Company ka internal web server | `http://repo.company.local/rhel9/BaseOS/` |

Yeh addresses misaalein hain; aapke lab ke confirmed paths nahi hain.

**Sirf RPM files wali directory kaafi nahi: repository metadata bhi hona chahiye.** Installation media mein metadata aam tor par pehle se hota hai. Naya repo banane ke liye `createrepo_c` jaisa tool istemal ho sakta hai, lekin is practice sawal mein repo server banana aapka task nahi hai.

## EPEL repository

**EPEL = Extra Packages for Enterprise Linux.**

EPEL Fedora community ka project hai jo enterprise Linux distributions ke liye mazeed packages deta hai. Isay operating system ke stores ke sath aik extra software store samjhein.

Rocky Linux mein aapko yeh command nazar aa sakti hai:

```bash
sudo dnf install epel-release
```

`epel-release` repository configuration aur signing keys deta hai. **Yeh EPEL ke tamam software install nahi karta.** Setup ki zarooriyat distribution aur release par depend karti hain.

Is BaseOS/AppStream practice sawal mein EPEL ki zaroorat nahi hai.

## yum.repos.d directory

```text
/etc/yum.repos.d/
```

Is directory mein repository ki configuration files hoti hain. In files ka extension `.repo` hota hai.

DNF in files ko parh kar samajhta hai ke repository kahan hai aur usay kaise istemal karna hai.

```bash
ls /etc/yum.repos.d/
```

Yeh command configuration filenames dikhati hai.

**Is directory mein settings hoti hain; tamam available packages nahi hotay.** Aik `.repo` file mein multiple repositories define ho sakti hain.

## Repo file ko samajhna

Sirf samjhane ke liye configuration:

```ini
[practice-baseos]
name=Practice BaseOS
baseurl=http://repo.example.com/rhel9/BaseOS/
enabled=1
gpgcheck=1
gpgkey=file:///path/to/trusted-signing-key
```

| Setting | Matlab |
|---|---|
| `[practice-baseos]` | Repository ki unique ID jo commands mein istemal hoti hai |
| `name=` | Insaan ke parhne ke liye naam ya description |
| `baseurl=` | Repo ka address; aam tor par `repodata/` ki parent location |
| `enabled=1` | Repository enabled hai |
| `enabled=0` | Repository default tor par disabled hai |
| `gpgcheck=1` | Package signatures check karna |
| `gpgcheck=0` | Package signature checking band karna |
| `gpgkey=` | Bharosaymand signing key ki location |

Upar diya URL aur key path placeholders hain. Lab ki asal values istemal karein. Kuch repositories fixed `baseurl=` ke bajaye mirrors dhoondne ke liye `mirrorlist=` ya `metalink=` istemal karti hain.

## RHCSA practice sawal

Aapke diye hue sawal ka matlab hai:

> `servera` par repositories configure karein taa-ke YUM/DNF ke zariye packages install kiye ja sakein.

BaseOS aur AppStream ke liye dono supplied URLs yeh hain:

```text
http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
```

**Aham baat: dono URLs aik jaisay hain.** Mumkin hai path adhoora ho ya copy karte waqt ghalti hui ho.

Do alag naam wali entries agar aik hi URL istemal karein, to dono aik hi source se packages hasil karti hain. Khud se `/BaseOS` ya `/AppStream` lagane ko sahi hal na samjhein. Lab instructions mein paths confirm karein, ya agar server directory listing deta ho to usay dekhein.

Yeh aapka diya hua practice sawal hai; isay official exam question ke tor par verify nahi kiya gaya. In notes ki tayyari ke dauran lab URL test nahi hua.

## Configuration ke qadam

### Qadam 1: Sahi machine par kaam karein

`servera` ke liye diya gaya console ya SSH access istemal karein. Check karein:

```bash
hostname
```

Maqsad: tasdeeq karna ke aap sahi server par configuration kar rahe hain.

### Qadam 2: Repository metadata check karein

Lab environment ke andar chalayein:

```bash
curl -fL http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht/repodata/repomd.xml
```

| Result | Matlab |
|---|---|
| XML output | Metadata ka shuruati point accessible lagta hai |
| HTTP 404 | Path ghalat ya adhoora ho sakta hai |
| Could not resolve host | Hostname resolve nahi ho raha |

XML milna yeh sabit nahi karta ke har referenced metadata file aur har package bhi accessible hai.

Yeh hostname sirf lab network ke andar resolve ho sakta hai. Ghar ke computer se failure ka matlab zaroori nahi ke lab URL ghalat hai.

### Qadam 3: Repo file banayein

```bash
sudo vi /etc/yum.repos.d/practice.repo
```

`vi` mein:

1. `i` dabayein taa-ke text likh sakein.
2. Configuration enter karein.
3. `Esc` dabayein.
4. `:wq` likhein aur Enter dabayein; file save ho kar editor band ho jayega.

Agar lab dono identical URLs ko confirm kare, to unke mutabiq configuration yeh hai:

```ini
[practice-baseos]
name=Practice BaseOS
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
enabled=1
gpgcheck=0

[practice-appstream]
name=Practice AppStream
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
enabled=1
gpgcheck=0
```

**Yeh do IDs banati hai jo aik hi source ki taraf ishara karti hain. Is se BaseOS aur AppStream ka content alag nahi ho jata.** Corrected URLs milne par har entry mein uska sahi URL likhein.

Yahan `gpgcheck=0` is assumption par hai ke practice lab signature checking band karne ki ijazat deta hai. Agar key di gayi ho ya signature checking required ho, to `gpgcheck=1` aur sahi trusted key configuration istemal karein.

### Qadam 4: Saved file check karein

```bash
cat /etc/yum.repos.d/practice.repo
```

Spelling, brackets, unique IDs, URLs aur `.repo` extension check karein.

## Verification aur package installation

Enabled repositories dekhne ke liye:

```bash
dnf repolist
```

Repo ka list mein nazar aana batata hai ke configuration pehchani gayi hai. **Is se URL ka kaam karna sabit nahi hota.**

Sirf apni dono repository IDs ka metadata refresh karein:

```bash
sudo dnf --disablerepo='*' \
  --enablerepo=practice-baseos \
  --enablerepo=practice-appstream \
  makecache --refresh
```

| Hissa | Kaam |
|---|---|
| `--disablerepo='*'` | Sirf is command ke liye baqi repos ko exclude karta hai |
| `--enablerepo=practice-baseos` | Is command mein BaseOS ID select karta hai |
| `--enablerepo=practice-appstream` | Is command mein AppStream ID select karta hai |
| `makecache --refresh` | Metadata dobara check karke cache banata hai |

Yeh options baqi repositories ko permanently disable nahi karte.

Har entry ko alag test karne ke liye:

```bash
sudo dnf --disablerepo='*' --enablerepo=practice-baseos makecache --refresh
sudo dnf --disablerepo='*' --enablerepo=practice-appstream makecache --refresh
```

Agar sawal kisi package ki installation bhi mange:

```bash
sudo dnf --disablerepo='*' \
  --enablerepo=practice-baseos \
  --enablerepo=practice-appstream \
  install PACKAGE_NAME
```

`PACKAGE_NAME` ki jagah manga gaya package name likhein. Successful installation, sirf `repolist` se zyada mazboot verification hai. Agar sawal sirf configuration ka ho, to bila zaroorat koi package install na karein.

## Troubleshooting

| Masla ya error | Kya check karein? | Madadgar check |
|---|---|---|
| Could not resolve host | DNS ya hostname | `getent hosts content.example.com` |
| HTTP 404 | Repository path | `baseurl` aur `repodata/repomd.xml` check karein |
| Connection timed out | Routing, firewall ya server availability | `ip route`; URL ko `curl` se test karein |
| Connection refused | Destination par service available nahi | Lab server aur service check karein |
| Repo enabled list mein nahi | Disabled entry, extension ya config error | `dnf repolist --all` |
| Cannot download repomd.xml | URL, metadata, DNS ya network | Metadata URL aur DNF error dekhein |
| No match for argument | Package name ya selected repos | `dnf search PACKAGE_NAME` |
| GPG/signature error | Missing/ghalat key ya invalid signature | Lab ki signing-key instructions dekhein |
| Duplicate repo ID | Multiple entries mein aik hi bracketed ID | `.repo` files inspect karein |

Nayi configuration ke liye existing repo files delete na karein. Kisi samajh na aane wali signature error ko chhupane ke liye signature checking band na karein.

## Jaldi revision

| Lafz | Aik line mein matlab |
|---|---|
| RPM package | Installation ke liye tayyar software |
| Repository | Packages aur unka metadata |
| DNF/YUM | Packages manage karne wala tool |
| Local repo | Apni machine ya internal network ka repo |
| EPEL | Enterprise Linux ke liye extra packages |
| `/etc/yum.repos.d/` | Repository configuration ki directory |
| `.repo` | Repository configuration file ka extension |
| `baseurl` | Repository content ka address |
| `repodata` | Repository metadata ki directory |

Exam mein tartib:

1. Sahi machine check karein.
2. Sahi URLs confirm karein.
3. `.repo` file banayein.
4. Settings inspect karein.
5. Metadata refresh karein.
6. Agar manga gaya ho to package install karein.

## Practice sawalat aur jawab

### Sawalat

1. Kya `/etc/yum.repos.d/` mein tamam available RPM packages hotay hain?
2. DNF aur repository mein kya farq hai?
3. `enabled=1` ka kya matlab hai?
4. Kya `epel-release` install karne se tamam EPEL software install ho jata hai?
5. Sirf `dnf repolist` URL verify karne ke liye kaafi kyun nahi?
6. Kya aik URL par do repo labels dene se do alag sources ban jate hain?
7. Metadata ka shuruati point check karne ke liye kaunsi file request karenge?

### Jawab

1. Nahi. Is mein aam tor par repository configuration files hoti hain.
2. DNF installation manage karta hai; repository packages aur metadata deti hai.
3. Repository default tor par enabled hai.
4. Nahi. Repository configuration aur signing keys install hoti hain.
5. Repo list mein aa sakta hai chahe uska server reachable na ho.
6. Nahi. Dono aik hi location ki taraf ishara karte hain.
7. Repository base URL ke andar `repodata/repomd.xml`.

## Mirrorlist aur baseurl ka farq

Attached project mein sab se useful baat yeh hai ke **Rocky ke public mirrors aur lab ke direct server mein farq** samjhaya gaya hai.

| Setting | DNF kya karta hai? |
|---|---|
| `baseurl=` | Seedha diye hue repository address se metadata aur packages leta hai |
| `mirrorlist=` | Service se mirror servers ke addresses leta hai, phir mirror se packages leta hai |
| `#baseurl=` | Line comment hai; active configuration nahi |

Mirrorlist service aik address directory ki tarah hai; mirror servers asal package stores hain. `baseurl` internet, internal network ya local `file://` path ka bhi ho sakta hai.

## Rocky repo settings ki tafseel

Apni VM par asal configuration dekhein:

```bash
ls -l /etc/yum.repos.d/
cat /etc/yum.repos.d/rocky.repo
```

Filename aapki installation mein mukhtalif ho sakta hai. Attached file se aik chhoti misaal:

```ini
[baseos]
name=Rocky Linux $releasever - BaseOS
mirrorlist=https://mirrors.rockylinux.org/mirrorlist?arch=$basearch&repo=BaseOS-$releasever$rltype
#baseurl=http://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
countme=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-9
```

Yeh samjhane ke liye hai; working `rocky.repo` ko is example se overwrite na karein.

| Setting | Roman Urdu mein matlab |
|---|---|
| `$releasever` | DNF distribution ki release value bhar deta hai, jaise `9` |
| `$basearch` | System architecture, jaise `x86_64` ya `aarch64` |
| `$rltype` | Rocky ki repo naming mein istemal hone wali variable |
| `$contentdir` | Rocky ke content path ki variable |
| `countme=1` | Mirror requests ke zariye systems ki tadaad ka andaza lagane mein madad |
| `metadata_expire=6h` | Chhay ghantay baad cached metadata expired samjha jata hai aur zaroorat par refresh hota hai |
| `gpgkey=file:///...` | Isi machine par trusted public signing key ki location |

**`metadata_expire=6h` ka matlab installed software chhay ghantay baad expire hona nahi.** DNF sirf ghantay guzarne par background mein lazmi refresh nahi karta; metadata istemal karte waqt freshness dekhta hai.

Variables ko DNF substitute karta hai. Inke liye khud fixed values likhna aam tor par zaroori nahi.

## Package kis repo mein hai

```bash
dnf info bash
dnf info python3
dnf info --available httpd
```

Output mein `Repository` field dekhein. Installed package ke liye `@System` aa sakta hai; `From repo` bhi dikh sakta hai. Available package ka source dekhne ke liye `--available` useful hai.

```bash
dnf repolist
dnf repolist --all
```

Pehli command enabled repositories dikhati hai. Doosri disabled entries bhi dikhati hai. Output aapki VM ke installed repositories ke mutabiq hoga; attached sample ko apna expected exact output na samjhein.

## Eight repo ka doosra lab example

Attached file mein pehle sawal se **alag URLs** diye gaye hain:

```text
http://repo.eight.example.com/BaseOS
http://repo.eight.example.com/AppStream
```

Yeh pehle identical `content.example.com` URLs ka confirmed correction nahi; doosra practice example hai. Har lab mein usi sawal ke URLs istemal karein.

### File banayein

```bash
sudo vi /etc/yum.repos.d/eight.repo
```

```ini
[eight-baseos]
name=Eight BaseOS
baseurl=http://repo.eight.example.com/BaseOS
enabled=1
gpgcheck=0

[eight-appstream]
name=Eight AppStream
baseurl=http://repo.eight.example.com/AppStream
enabled=1
gpgcheck=0
```

Signature checking ke liye wahi lab assumption lagu hai jo pehle samjhayi gayi hai. Sawal mein key ka zikr na hona akela saboot nahi ke `gpgcheck=0` lazmi hai.

**Repo IDs tamam `.repo` files mein unique rakhein.** Agar Rocky mein `[baseos]` pehle mojood hai, nayi file mein `[eight-baseos]` jaisi distinct ID istemal karein. Sirf `name=` badalne se ID nahi badalti.

### Metadata aur configuration verify karein

```bash
curl -fL --max-time 10 http://repo.eight.example.com/BaseOS/repodata/repomd.xml
curl -fL --max-time 10 http://repo.eight.example.com/AppStream/repodata/repomd.xml
sudo dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream makecache --refresh
dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream repolist
```

`curl` metadata entry point test karta hai; DNF ka successful metadata refresh zyada mukammal check hai. Lab hostname ghar ke network par reachable hona zaroori nahi.

### Agar Apache install karna task ka hissa ho

```bash
dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream info --available httpd
sudo dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream install httpd
rpm -q httpd
```

`rpm -q httpd` local RPM database se check karta hai ke package installed hai. Yeh Apache service ke running hone ya installation ke source ka akela saboot nahi.

**Attached file ki correction:** kuch commands mein `Eightbaseos,Eightappstream` likha tha, jabke configured IDs `eight-baseos` aur `eight-appstream` hain. Upar tamam commands mein IDs configuration ke sath match karti hain. ID ka spelling aur case exact rakhein.

## DNF cache commands

Cache ko store ki saved catalog copy samjhein.

| Command | Kaam |
|---|---|
| `sudo dnf clean all` | Cached metadata aur cached packages waghera saaf karta hai; installed software remove nahi karta |
| `sudo dnf makecache` | Enabled repositories ka metadata cache tayyar karta hai; fresh cache reuse ho sakta hai |
| `sudo dnf makecache --refresh` | Metadata expire karke dobara freshness check karta hai |
| `sudo dnf install httpd` | Package aur dependencies install karta hai |

Har installation se pehle `clean all` chalana zaroori nahi. Naye repo ki verification ke liye targeted `makecache --refresh` aam tor par kaafi hai. Unrelated repo ki failure se bachne ke liye selected IDs use karein.

## Network checks ki tartib

Repository tak access ke liye interface, IP address, route, name resolution aur HTTP service sab ka kaam karna zaroori ho sakta hai.

### Interface aur IP

```bash
ip -br link
ip -br address
nmcli device status
nmcli connection show
```

`nmcli connection show` connection profiles dikhata hai; akela cable ki sehat prove nahi karta. VM ka NIC virtual hota hai; virtual link aur hypervisor/network setup bhi check hota hai.

Agar `ethtool` available ho:

```bash
sudo ethtool INTERFACE_NAME
```

`INTERFACE_NAME` ko apne interface, jaise `enX0`, se replace karein. `Link detected: yes` link ki indication hai, poori connectivity ka saboot nahi.

### Routing aur gateway

```bash
ip route
```

Default gateway doosre networks tak traffic bhejne ke liye istemal hota hai. Isi local subnet ka repo aam tor par direct on-link route se reachable hota hai; uske liye default gateway lazmi nahi.

### Name resolution

```bash
cat /etc/resolv.conf
getent hosts repo.eight.example.com
```

`/etc/resolv.conf` configured DNS resolver information dikhata hai; DNS server zaroori nahi ke home router hi ho.

`getent` ka matlab “get entries” hai. `getent hosts` system ke host lookup rules istemal karta hai, jo `/etc/nsswitch.conf` ke mutabiq `/etc/hosts`, DNS aur doosre sources ko shamil kar sakte hain. IP milna system name resolution ki kamyabi hai; yeh DNS se hi jawab milne ka lazmi saboot nahi.

Agar `dig` aur `nslookup` pehle se available hon:

```bash
dig repo.eight.example.com
nslookup repo.eight.example.com
```

Yeh DNS queries karte hain. Rocky 9 mein in tools ke liye `bind-utils` package istemal hota hai:

```bash
sudo dnf install bind-utils
```

Agar repository hi fail ho rahi ho, to naya tool install karna bhi fail ho sakta hai. Pehle available commands se diagnosis karein.

### Ping aur HTTP

```bash
ping -c 3 repo.eight.example.com
curl -fL --max-time 10 http://repo.eight.example.com/BaseOS/repodata/repomd.xml
```

Ping ICMP use karta hai; TCP/UDP ports use nahi karta. Successful ping HTTP service ko prove nahi karta; failed ping bhi HTTP failure ko lazmi prove nahi karta kyun ke ICMP blocked ho sakta hai.

`curl -I` sirf HTTP HEAD response leta hai. Directory ka 200 response package metadata ko prove nahi karta; kuch servers HEAD ko reject bhi karte hain. Is liye metadata file ka GET aur DNF refresh behtar checks hain.

## Real job scenario aur rollback

Attached file ka business scenario: company NEXUS sirf approved repositories se software install/update karna chahti hai. Doosri team repository server tayyar kar chuki hai; aap clients ko us server ka address configure karte hain aur patching validate karte hain.

Suggested tartib:

1. Aik test VM par current state record karein.
2. Repo configuration ki backup banayein.
3. Approved URLs aur trusted keys configure karein.
4. Metadata aur requested package test karein.
5. Pilot group par Ansible se change karein.
6. Validation ke baad wider rollout ke liye playbook/Automation Controller use karein.
7. Evidence aur rollback steps record karein.

### Home directory mein backup

```bash
backup_dir="$HOME/repo-backups/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$backup_dir"
sudo cp -a /etc/yum.repos.d "$backup_dir/"
dnf repolist --all > "$backup_dir/repolist-before.txt"
rpm -qa | sort > "$backup_dir/packages-before.txt"
```

Yeh repo configuration aur package list ki backup/evidence hai. **Installed packages, data ya poori VM ki backup nahi.** Package patching rollback ke liye munasib snapshot/backup aur tested recovery plan alag se chahiye.

### Sirf nayi eight.repo configuration ko wapas lena

```bash
sudo mv /etc/yum.repos.d/eight.repo "$backup_dir/eight.repo.disabled"
dnf repolist
```

Yeh same shell mein saved `backup_dir` ki misaal hai. Naye session mein asal backup path use karein. File repo directory se hatne par uski entries load nahi hoti. Is se installed/updated packages downgrade ya uninstall nahi hotay. Agar purani file modify ki thi, usi file ki backup restore karein; poori directory ko andha-dhund overwrite na karein.

Repo files disk par save hoti hain aur reboot ke baad bhi rehti hain. Sirf persistence check karne ke liye har dafa reboot zaroori nahi; task ya maintenance plan maange to reboot ke baad verify karein.

## Completion checklist aur review

- [ ] Sahi VM confirm ki.
- [ ] URL aur repo IDs check kiye.
- [ ] Zaroorat ke mutabiq configuration backup banayi.
- [ ] `.repo` file save aur inspect ki.
- [ ] Targeted metadata refresh successful hua.
- [ ] Manga gaya package install aur verify kiya, agar task mein tha.
- [ ] Existing security settings bila wajah change nahi ki.
- [ ] Evidence aur configuration rollback samajh liya.
- [ ] Patching rollback ko repo rollback se alag samjha.

Review sawalat:

1. Business masla kya tha? **Approved sources se controlled software distribution aur patching.**
2. Kaunsi command enabled configuration dikhati hai? **`dnf repolist`.**
3. Kaunsi command metadata access verify karti hai? **Targeted `dnf makecache --refresh`.**
4. Persistence kis wajah se hai? **Configuration disk par `.repo` file mein save hai.**
5. Repo rollback aur patch rollback mein farq? **Repo rollback source settings wapas karta hai; patch rollback software/data state restore karta hai.**

Attached Markdown mein images ke relative links thay, lekin image files attach nahi hui theen. Is combined file mein unki jagah text explanations aur tables hain taa-ke broken images na dikhain.

## References

- User ka attached project: `RHCSA_Project_02_software-repositories(1).md`.
- [DNF configuration reference](https://dnf.readthedocs.io/en/latest/conf_ref.html)
- [Red Hat: RHEL 9 repositories](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/considerations_in_adopting_rhel_9/ref_repositories_considerations-in-adopting-rhel-9)

- [Red Hat: Managing custom software repositories, RHEL 9](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/assembly_managing-custom-software-repositories_managing-software-with-the-dnf-tool)
- [DNF command reference](https://dnf.readthedocs.io/en/latest/command_ref.html)
- [Fedora: EPEL FAQ](https://fedoraproject.org/wiki/EPEL/FAQ)
- [Fedora: epel-release package](https://packages.fedoraproject.org/pkgs/epel-release/epel-release/)
- [Aapka diya hua YouTube link](https://www.youtube.com/live/HRKvoIqJvbc?si=am7_lUEKMiw77OqJ) — video ka content retrieve nahi ho saka; yeh notes video ka verified khulasa nahi hain.

[Topic index par wapas jayein](#topic-index)
