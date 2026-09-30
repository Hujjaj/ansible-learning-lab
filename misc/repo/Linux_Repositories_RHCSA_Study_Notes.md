# Linux Repositories and RHCSA Repository Configuration

Combined beginner study notes • RHEL 9 / Rocky Linux 9

Includes useful material from the attached `RHCSA_Project_02_software-repositories(1).md`, corrected commands, and a workplace scenario.

## Topic index

1. [What is a package?](#what-is-a-package)
2. [What is a repository?](#what-is-a-repository)
3. [YUM and DNF](#yum-and-dnf)
4. [Dependencies and metadata](#dependencies-and-metadata)
5. [BaseOS and AppStream](#baseos-and-appstream)
6. [Local repositories](#local-repositories)
7. [EPEL](#epel)
8. [The yum.repos.d directory](#the-yumreposd-directory)
9. [Understanding a repo file](#understanding-a-repo-file)
10. [The RHCSA practice question](#the-rhcsa-practice-question)
11. [Step-by-step configuration](#step-by-step-configuration)
12. [Verification and package installation](#verification-and-package-installation)
13. [Troubleshooting](#troubleshooting)
14. [Quick revision](#quick-revision)
15. [Practice questions](#practice-questions)
16. [Mirrorlist versus baseurl](#mirrorlist-versus-baseurl)
17. [Rocky repository settings in detail](#rocky-repository-settings-in-detail)
18. [Finding a package repository](#finding-a-package-repository)
19. [Eight repository lab example](#eight-repository-lab-example)
20. [DNF cache commands](#dnf-cache-commands)
21. [Network checks in order](#network-checks-in-order)
22. [Real job scenario and rollback](#real-job-scenario-and-rollback)
23. [Completion checklist and review](#completion-checklist-and-review)
24. [References](#references)

## What is a package?

A package is software prepared for installation. It contains program files and installation information. On RHEL and Rocky Linux, package files usually end in `.rpm`.

Examples: `nginx` is a web server; `git` manages source-code history; `htop` displays system activity.

Installing a package puts its files in the appropriate system locations. Downloading an RPM alone does not install it.

## What is a repository?

A repository, or repo, is a collection of software packages plus metadata describing them. It gives your package manager a source from which to install and update software.

| Linux concept | Grocery store analogy |
|---|---|
| Repository | Store |
| Package | Item |
| Metadata | Catalog |
| DNF/YUM | Helper who finds and collects items |
| Repo configuration | Store address and instructions |

Here, “repository” means a package repository. A Git repository stores project files and version history; that is a different use of the word.

## YUM and DNF

YUM and DNF are package managers. They install, update, and remove packages. DNF is YUM's successor; on RHEL 9, the `yum` command is provided for compatibility with DNF.

```bash
sudo dnf install nginx
sudo dnf update nginx
sudo dnf remove nginx
```

These commands change the system. Run installation/removal commands only when needed for your lab task.

## Dependencies and metadata

**Dependencies** are other packages a program needs. DNF resolves them and includes the required packages in the installation transaction.

**Metadata** is the repository catalog: package names, versions, dependencies, and other details. It is not the application itself.

For a typical RPM repository, this file is an important metadata entry point:

```text
repodata/repomd.xml
```

Conceptually, DNF reads configuration, checks repository metadata, resolves packages and dependencies, retrieves packages, verifies them as configured, and installs them. Cached metadata or packages may be reused.

## BaseOS and AppStream

| Repository | Main purpose |
|---|---|
| BaseOS | Core operating system functionality |
| AppStream | Additional applications, runtimes, and tools |

Both are standard parts of RHEL's software distribution. A label in a `.repo` file does not determine its contents; the URL points to the actual content.

## Local repositories

A local repository provides packages from your own machine or an internal network. Organizations use them for controlled package distribution and environments with restricted internet access.

| Source | Illustrative address |
|---|---|
| Same-machine directory | `file:///home/student/myrepo` |
| Mounted installation media | `file:///mnt/rhel9/BaseOS` |
| Internal web server | `http://repo.company.local/rhel9/BaseOS/` |

These are examples, not confirmed lab paths. The directory must have valid repository metadata; a folder of RPMs alone is insufficient. Installation media commonly already supplies that metadata. Building a new repository may require a tool such as `createrepo_c`, but that is not the task in this question.

## EPEL

**EPEL = Extra Packages for Enterprise Linux.**

EPEL is a Fedora community project providing additional packages for enterprise Linux distributions. Think of it as another software store alongside the operating system repositories.

On Rocky Linux, you may encounter:

```bash
sudo dnf install epel-release
```

`epel-release` provides repository configuration and signing keys. It does not install every EPEL application. Setup prerequisites depend on your distribution and release.

**EPEL is not required for the BaseOS/AppStream practice question.**

## The yum.repos.d directory

```text
/etc/yum.repos.d/
```

This directory contains repository configuration files ending in `.repo`. DNF reads these files to learn where repositories are and how to use them.

```bash
ls /etc/yum.repos.d/
```

The directory stores settings, not the complete package collection. One `.repo` file can define multiple repositories.

## Understanding a repo file

Illustrative configuration:

```ini
[practice-baseos]
name=Practice BaseOS
baseurl=http://repo.example.com/rhel9/BaseOS/
enabled=1
gpgcheck=1
gpgkey=file:///path/to/trusted-signing-key
```

| Field | Meaning |
|---|---|
| `[practice-baseos]` | Unique repository ID used in commands |
| `name=` | Human-readable description |
| `baseurl=` | Repository location, normally above `repodata/` |
| `enabled=1` | Enable the repository |
| `enabled=0` | Disable it by default |
| `gpgcheck=1` | Check package signatures |
| `gpgcheck=0` | Disable package signature checking |
| `gpgkey=` | Location of a trusted signing key |

The URL and key path above are placeholders. Use the lab's actual values. Some repositories use `mirrorlist=` or `metalink=` to locate mirrors instead of a fixed `baseurl=`.

## The RHCSA practice question

The supplied question asks you to configure repositories on `servera` so packages are available through YUM/DNF.

Both supplied URLs are:

```text
http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
```

**Important: the BaseOS and AppStream URLs are identical.** They may be incomplete or copied incorrectly. Two differently named entries using the same URL still access the same source. Do not assume that adding `/BaseOS` or `/AppStream` will fix it; confirm the paths in the lab instructions or inspect the server's directory listing if one is available.

This is the user's supplied practice question, not a verified official exam question. The lab endpoint has not been tested from these notes.

## Step-by-step configuration

### Step 1: Work on the correct machine

Use the console or SSH access provided for `servera`. Check:

```bash
hostname
```

### Step 2: Check the supplied repository metadata

Run inside the lab environment:

```bash
curl -fL http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht/repodata/repomd.xml
```

XML output suggests the metadata entry point is accessible. It does not prove that every referenced metadata file or package is accessible. A 404 suggests an incorrect or incomplete path. A DNS error means the hostname could not be resolved.

The hostname may only resolve inside the lab network. Failure from your home computer does not establish that the lab URL is invalid.

### Step 3: Create a configuration file

```bash
sudo vi /etc/yum.repos.d/practice.repo
```

In `vi`: press `i` to insert text; after editing, press `Esc`, type `:wq`, and press Enter to save and exit.

If the lab confirms the supplied identical URLs, the following reflects them exactly:

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

**This defines two IDs pointing to one source. It does not establish separate BaseOS and AppStream content.** If the question provides corrected URLs, put the correct URL in each corresponding entry.

Here, `gpgcheck=0` assumes the lab permits disabling package signature checking. If a signing key is supplied or signature checks are required, use `gpgcheck=1` with the correct trusted key configuration.

### Step 4: Inspect your saved configuration

```bash
cat /etc/yum.repos.d/practice.repo
```

Check spelling, brackets, unique IDs, URLs, and the `.repo` filename extension.

## Verification and package installation

List enabled repositories:

```bash
dnf repolist
```

Seeing a repository listed shows configuration is recognized; it does not prove the URL works.

Refresh metadata only for the two configured IDs:

```bash
sudo dnf --disablerepo='*' \
  --enablerepo=practice-baseos \
  --enablerepo=practice-appstream \
  makecache --refresh
```

`--disablerepo='*'` excludes other repositories for this command only. The two `--enablerepo` options select your configured IDs. These options do not permanently disable the other repositories.

To test each source separately:

```bash
sudo dnf --disablerepo='*' --enablerepo=practice-baseos makecache --refresh
sudo dnf --disablerepo='*' --enablerepo=practice-appstream makecache --refresh
```

If a package is requested, install it using the same repository selection as needed:

```bash
sudo dnf --disablerepo='*' \
  --enablerepo=practice-baseos \
  --enablerepo=practice-appstream \
  install PACKAGE_NAME
```

Replace `PACKAGE_NAME` with the requested package. A successful installation is stronger evidence than `repolist` alone. Do not install an arbitrary package if the question only asks for configuration.

## Troubleshooting

| Symptom | Likely area to investigate | Useful check |
|---|---|---|
| Could not resolve host | DNS or incorrect hostname | `getent hosts content.example.com` |
| HTTP 404 | Incorrect repository path | Check `baseurl` and `repodata/repomd.xml` |
| Connection timed out | Routing, firewall, or server availability | `ip route`; test URL with `curl` |
| Connection refused | Service unavailable at destination | Check server/service in the lab |
| Repo missing from enabled list | Disabled entry, wrong file extension, or configuration error | `dnf repolist --all` |
| Cannot download repomd.xml | URL, metadata, DNS, or network issue | Test metadata URL and read DNF error |
| No match for argument | Package unavailable in selected repos or wrong name | `dnf search PACKAGE_NAME` |
| GPG/signature error | Missing or incorrect trusted key, or invalid signature | Review lab key instructions |
| Duplicate repo ID warning | Same bracketed ID in multiple entries | Inspect `.repo` files |

Do not delete existing repository files to solve a new configuration task. Do not disable signature checks merely to hide an unexplained signature failure.

## Quick revision

| Term | One-line meaning |
|---|---|
| RPM package | Software prepared for installation |
| Repository | Packages plus metadata |
| DNF/YUM | Package manager |
| Local repo | Repository on your machine or internal network |
| EPEL | Additional enterprise Linux packages |
| `/etc/yum.repos.d/` | Repository configuration directory |
| `.repo` | Repository configuration filename extension |
| `baseurl` | Address of repository content |
| `repodata` | Repository metadata directory |

Exam workflow: **correct machine → correct URLs → create `.repo` file → inspect settings → refresh metadata → install requested package if required.**

## Practice questions

1. Does `/etc/yum.repos.d/` hold all available RPM packages?
2. What is the difference between DNF and a repository?
3. What does `enabled=1` mean?
4. Does installing `epel-release` install all EPEL software?
5. Why is `dnf repolist` alone insufficient to validate a URL?
6. Do two repository labels using the same URL create two different sources?
7. What file can you request to check the metadata entry point?

### Answers

1. No. It normally holds repository configuration files.
2. DNF manages installation; a repository supplies packages and metadata.
3. The repository is enabled by default.
4. No. It installs repository configuration and signing keys.
5. A configured repository can be listed even when its server is unreachable.
6. No. They point to the same location.
7. `repodata/repomd.xml`, relative to the repository base URL.

## Mirrorlist versus baseurl

The attached project explains the difference between Rocky's public mirrors and a direct lab repository server.

| Setting | What DNF does |
|---|---|
| `baseurl=` | Retrieves metadata and packages directly from the supplied repository address |
| `mirrorlist=` | Requests mirror addresses from a service, then retrieves packages from a mirror |
| `#baseurl=` | Treats the line as a comment; it is inactive |

The mirrorlist service acts as an address directory. Mirror servers hold the package content. A `baseurl` can point to an internet server, an internal server, or a local `file://` location.

## Rocky repository settings in detail

Inspect the actual configuration on your VM:

```bash
ls -l /etc/yum.repos.d/
cat /etc/yum.repos.d/rocky.repo
```

The filename may differ on your installation. A shortened example from the attached project:

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

This is an explanatory example. Do not overwrite a working `rocky.repo` with it.

| Setting | Meaning |
|---|---|
| `$releasever` | DNF substitutes the distribution release value, such as `9` |
| `$basearch` | System architecture, such as `x86_64` or `aarch64` |
| `$rltype` | Rocky variable used in repository naming |
| `$contentdir` | Rocky variable used in its content path |
| `countme=1` | Helps estimate the number of systems using mirrors through repository requests |
| `metadata_expire=6h` | Cached metadata expires after six hours and is refreshed when needed |
| `gpgkey=file:///...` | Location of the trusted public signing key on this machine |

**`metadata_expire=6h` does not mean installed software expires after six hours.** DNF does not necessarily run a background refresh when that time passes; it checks freshness when metadata is needed.

DNF substitutes the variables. You normally do not need to replace them manually with fixed values.

## Finding a package repository

```bash
dnf info bash
dnf info python3
dnf info --available httpd
```

Inspect the `Repository` field. An installed package may show `@System` and may also have a `From repo` field. Use `--available` to inspect available package sources.

```bash
dnf repolist
dnf repolist --all
```

The first command lists enabled repositories; the second includes disabled entries. Output depends on your VM's configuration. The attached sample is not an exact expected output for every VM.

## Eight repository lab example

The attached file provides a second practice example with separate URLs:

```text
http://repo.eight.example.com/BaseOS
http://repo.eight.example.com/AppStream
```

These are different from the earlier `content.example.com` URLs. They are not a confirmed correction to that first question. Use the URLs supplied for each particular lab.

### Create the configuration

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

The signature-checking assumption explained earlier also applies here. A question omitting a key does not by itself establish that `gpgcheck=0` is required.

**Keep repository IDs unique across all `.repo` files.** If Rocky already uses `[baseos]`, use a distinct ID such as `[eight-baseos]` in the new file. Changing `name=` alone does not change the repository ID.

### Verify metadata and configuration

```bash
curl -fL --max-time 10 http://repo.eight.example.com/BaseOS/repodata/repomd.xml
curl -fL --max-time 10 http://repo.eight.example.com/AppStream/repodata/repomd.xml
sudo dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream makecache --refresh
dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream repolist
```

The `curl` commands test metadata entry points. A successful DNF metadata refresh provides a more complete check. A lab hostname may be unavailable from your home network.

### Install Apache if the task requires it

```bash
dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream info --available httpd
sudo dnf --disablerepo='*' --enablerepo=eight-baseos --enablerepo=eight-appstream install httpd
rpm -q httpd
```

`rpm -q httpd` checks the local RPM database for installation. It does not by itself prove that Apache is running or identify the installation source.

**Correction to the attached project:** some commands used `Eightbaseos,Eightappstream`, while the configured IDs were `eight-baseos` and `eight-appstream`. All commands above use IDs matching the configuration. Match spelling and case exactly.

## DNF cache commands

Think of the cache as a saved copy of a store catalog.

| Command | Purpose |
|---|---|
| `sudo dnf clean all` | Removes cached metadata and cached packages; does not remove installed software |
| `sudo dnf makecache` | Prepares metadata caches for enabled repositories; fresh caches may be reused |
| `sudo dnf makecache --refresh` | Expires metadata and checks freshness again |
| `sudo dnf install httpd` | Installs the package and its dependencies |

You do not need `clean all` before every installation. A targeted `makecache --refresh` is usually sufficient to validate a newly configured repository. Select the intended repository IDs to avoid unrelated repository failures.

## Network checks in order

Access to a repository may depend on a working interface, IP address, route, name resolution, and HTTP service.

### Interface and IP address

```bash
ip -br link
ip -br address
nmcli device status
nmcli connection show
```

`nmcli connection show` lists connection profiles; it does not by itself prove a physical cable is healthy. A VM uses a virtual NIC, so virtual links and hypervisor networking also matter.

If `ethtool` is available:

```bash
sudo ethtool INTERFACE_NAME
```

Replace `INTERFACE_NAME` with your actual interface, such as `enX0`. `Link detected: yes` indicates a link, not complete connectivity.

### Routing and gateway

```bash
ip route
```

A default gateway sends traffic to other networks. A repository on the same local subnet normally uses an on-link route and does not require the default gateway for that connection.

### Name resolution

```bash
cat /etc/resolv.conf
getent hosts repo.eight.example.com
```

`/etc/resolv.conf` shows DNS resolver configuration. The DNS server is not necessarily your home router.

`getent` means “get entries.” With `hosts`, it uses the system's host lookup rules in `/etc/nsswitch.conf`, which may include `/etc/hosts`, DNS, and other sources. Receiving an IP address proves successful system name resolution, but not necessarily that DNS supplied the answer.

If `dig` and `nslookup` are already available:

```bash
dig repo.eight.example.com
nslookup repo.eight.example.com
```

These tools query DNS. On Rocky Linux 9 they are provided by `bind-utils`:

```bash
sudo dnf install bind-utils
```

When repositories are already failing, installing a troubleshooting tool may also fail. Start with tools available on the system.

### Ping and HTTP

```bash
ping -c 3 repo.eight.example.com
curl -fL --max-time 10 http://repo.eight.example.com/BaseOS/repodata/repomd.xml
```

Ping uses ICMP rather than TCP or UDP ports. Successful ping does not prove HTTP works. Failed ping does not necessarily prove HTTP is unavailable because ICMP may be blocked.

`curl -I` requests HTTP headers using HEAD. A successful directory response does not prove repository metadata is available, and some servers reject HEAD requests. Fetching the metadata file with GET and refreshing it through DNF are more useful repository checks.

## Real job scenario and rollback

The attached project's business scenario describes NEXUS, a company requiring software installation and updates from approved repositories. Another team has prepared the repository server. Your job is to configure clients to use it and validate patching.

Suggested sequence:

1. Record the current state on a test VM.
2. Back up repository configuration.
3. Configure approved URLs and trusted keys.
4. Test metadata and the requested package.
5. Apply the change to a pilot group with Ansible.
6. After validation, use a playbook and Automation Controller for a wider rollout.
7. Record evidence and rollback steps.

### Back up configuration in your home directory

```bash
backup_dir="$HOME/repo-backups/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$backup_dir"
sudo cp -a /etc/yum.repos.d "$backup_dir/"
dnf repolist --all > "$backup_dir/repolist-before.txt"
rpm -qa | sort > "$backup_dir/packages-before.txt"
```

This backs up repository configuration and records package inventory. **It does not back up installed package contents, application data, or the entire VM.** Patching rollback requires a suitable snapshot/backup and a tested recovery plan separately.

### Roll back only the new eight.repo configuration

```bash
sudo mv /etc/yum.repos.d/eight.repo "$backup_dir/eight.repo.disabled"
dnf repolist
```

This example assumes the same shell still has `backup_dir` set. In a new session, supply the actual backup path. Moving the file out of the repository directory prevents DNF from loading its entries. It does not downgrade or uninstall packages. If you modified an existing file, restore that particular file from its backup rather than blindly overwriting the entire directory.

Repository files persist on disk across reboots. Rebooting is not required every time merely to establish that a file is saved; verify after reboot when the task or maintenance plan requires it.

## Completion checklist and review

- [ ] Confirmed the correct VM.
- [ ] Checked URLs and repository IDs.
- [ ] Backed up configuration when needed.
- [ ] Saved and inspected the `.repo` file.
- [ ] Successfully refreshed metadata for the intended repositories.
- [ ] Installed and verified the requested package, if required.
- [ ] Kept existing security settings unless a justified change was required.
- [ ] Recorded evidence and understood configuration rollback.
- [ ] Distinguished patching rollback from repository rollback.

Review questions:

1. What business problem was solved? **Controlled software distribution and patching from approved sources.**
2. Which command lists enabled configuration? **`dnf repolist`.**
3. Which command verifies metadata access? **Targeted `dnf makecache --refresh`.**
4. Why is the configuration persistent? **It is saved in a `.repo` file on disk.**
5. How do repository rollback and patch rollback differ? **Repository rollback restores source settings; patch rollback restores software/data state.**

The attached Markdown references image files using relative paths, but those images were not attached. This combined document uses text explanations and tables in their place to avoid broken images.

## References

- User-provided project: `RHCSA_Project_02_software-repositories(1).md`.
- [DNF configuration reference](https://dnf.readthedocs.io/en/latest/conf_ref.html)
- [Red Hat: RHEL 9 repositories](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/considerations_in_adopting_rhel_9/ref_repositories_considerations-in-adopting-rhel-9)

- [Red Hat: Managing custom software repositories, RHEL 9](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/assembly_managing-custom-software-repositories_managing-software-with-the-dnf-tool)
- [DNF command reference](https://dnf.readthedocs.io/en/latest/command_ref.html)
- [Fedora: EPEL FAQ](https://fedoraproject.org/wiki/EPEL/FAQ)
- [Fedora: epel-release package](https://packages.fedoraproject.org/pkgs/epel-release/epel-release/)
- [User's YouTube reference](https://www.youtube.com/live/HRKvoIqJvbc?si=am7_lUEKMiw77OqJ) — video contents could not be retrieved; these notes do not claim to summarize it.

[Back to topic index](#topic-index)
