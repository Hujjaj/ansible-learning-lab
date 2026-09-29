# Linux Repositories and RHCSA Repository Configuration

Beginner study notes • RHEL 9 / Rocky Linux 9

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
16. [References](#references)

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

## References

- [Red Hat: Managing custom software repositories, RHEL 9](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_software_with_the_dnf_tool/assembly_managing-custom-software-repositories_managing-software-with-the-dnf-tool)
- [DNF command reference](https://dnf.readthedocs.io/en/latest/command_ref.html)
- [Fedora: EPEL FAQ](https://fedoraproject.org/wiki/EPEL/FAQ)
- [Fedora: epel-release package](https://packages.fedoraproject.org/pkgs/epel-release/epel-release/)

[Back to topic index](#topic-index)
