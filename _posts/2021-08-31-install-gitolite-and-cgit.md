---
title: "Install GitoLite and cgit"
date: 2021-08-31
categories: [Engineering,Linux]
tags: [gitolite,cgit,tools]
---

# Information
* System - `CentOS Linux release 7.3.1611 (Core)`
* Core - `Linux 3.10.0-123.8.1.el7.x86_64`

# Step 1. Setup gitolite

Create main directory:
```bash
$ mkdir -p /var/vcs
```
Create user:
```bash
$ useradd -U -m -c "Git User" -d "/var/vcs/git" git
$ passwd git
```
Create SSH key:
```bash
$ ssh-keygen -t rsa -f ~/gitolite -C "GitoLite Service Key"
$ cp ~/gitolite.pub /var/vcs/git
$ chown git:git /var/vcs/git/gitolite.pub
$ chmod 600 /var/vcs/git/gitolite.pub
```
Generated above key can be used in several ways as example for access to storage.

**Warning:** Using the same key whatever to access to the server and to the storage is prohibited. Two different keys must be used in that case.

Download and install gitolite:
```bash
$ su - git
$ mkdir -p $HOME/bin
$ git clone git://github.com/sitaramc/gitolite $HOME/gitolite
$ gitolite/install -to $HOME/bin
$ rm -fr $HOME/gitolite
$ gitolite setup -pk $HOME/gitolite.pub
$ rm -f $HOME/gitolite.pub
```

Do initial setup within file `/var/vcs/git/.gitolite.rc`:
```text
# Set default umask
UMASK                           =>  0027,

# Set acceptable config key (look for "git-config" in the documentation)
GIT_CONFIG_KEYS                 =>  '.*',

# Set project list file path
GITWEB_PROJECTS_LIST            => '/var/vcs/git/projects.list',

# Set log configuration
LOG_EXTRA                       =>  1,
LOG_DEST                        =>  "normal",

# show more detailed messages on deny
"expand-deny-messages",
```

# Step 2. Check access to the gitolite

Configure server in `~/.ssh/config` file:
```text
Host <host-alias>
User git
Hostname <host-name>
Port 22
IdentityFile ~/.ssh/gitolite # Path to the private key of <admin> user
```

```bash
$ ssh <host-alias>
```

# Step 3. Configure gitolite storage

To configure gitolite storage you need to clone storage from the server to your workstation. Changing config files and upload changes to the server apply storage configuration.

```bash
$ mkdir -p ~/work/git && cd work/git
$ git clone git:<host-alias>:gitolite-admin
```

Gitolite storage usually contains two folders: `conf` and `keydir`. The `conf` folder contains config file `gitolite.conf` (see [documentation](https://gitolite.com/gitolite/conf.html)). The `keydir` folder contains public keys of users.

By default `gitolite.conf` look like:
```bash
repo gitolite-admin
    RW+     =   gitolite
repo testing
    RW+     =   @all
```

## Example 1 - Create repository
For example we have `project1` project that locates at home directory. To setup git versioning and downloading to the cgit server we need perform certain steps.

* Add to the config file `gitolite.conf` configuration of new repository:
```text
repo project1
    RW+	= @all
    config cgit.owner	= Coders
    config cgit.desc	= Projects Group
    config cgit.section = projects
```
* Upload changes to the server:
```bash
$ git commit -a -m "append project1 configuration"
$ git push origin master
```
* Perform git initialization within project directory and upload it to the server:
```bash
$ cd ~/project1
$ git init
$ git commit -a -m "initial commit"
$ git remote add origin &lt;server-host&gt;:project1.git
$ git push -u origin master
```

## Example 2 - Create group

To create new group of users you need to make next changes to the `gitolite.conf` file:
```text
@admins = gitolite
repo gitolite-admin
    RW+     =   @admins
repo testing
    RW+     =   @all
```
After that you need to upload these changes to the server:
```bash
$ cd ~/work/git/gitolite-admin
$ git commit -a -m "Add group @admins"
$ git push origin master
```

## Example 3 - Create user

To create new user and its personal storage you need first create personal key for the user:
```bash
$ ssh-keygen -t rsa -f ~/user -C "User Key"
```
The name of the file within `keydir` folder must be the same as the user name that you will use later to work as that user.

Add `user.pub` file to the `keydir` folder of the storage at your workstation clone and change `gitolite.conf` file:
```bash
repo user
    RW+     =   user
```

Upload changes:
```bash
$ cd ~/work/git/gitolite-admin
$ git add keydir/user.pub
$ git commit -a -m "Add group new user 'user' and create his personal repo"
$ git push origin master
```

## Example 4 - Create wildcard configration

To create wildcard configration (configuration for several repositories) you need:
```text
repo cpp/[a-zA-Z][a-zA-Z0-9_\-\/]*
	C	= @admins
	RW+     = @admins
```
To understand deeply follow this [documentation](https://gitolite.com/gitolite/wild.html).

# Step 4. Install cgit on the server

Install prerequisites:
```bash
$ yum -y groupinstall "Development Tools"
$ yum -y install gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel
```

Download the last version `cgit`:
```bash
$ git clone https://git.zx2c4.com/cgit && cd cgit
$ git submodule init
$ git submodule update
```

Build and install:
```bash
$ mkdir -p $HOME/cgit-build
$ mkdir -p /var/www/<hostname>/git
$ make
$ make DESTDIR=$HOME/cgit-build install
$ mv -f $HOME/cgit-build/usr/local/lib/cgit /usr/local/lib
$ mv -f $HOME/cgit-build/var/www/htdocs/cgit/* /var/www/<hostname>/git
$ chown -R apache:apache /var/www/<hostname>/git
```

Apply required rights:
```bash
$ chmod 750 /var/vcs/git
$ chmod 750 /var/vcs/git/repositories
$ chmod 644 /var/vcs/git/projects.list
$ find /var/vcs/git/repositories/gitolite-admin.git -type d -exec chmod 0750 {} \;
$ find /var/vcs/git/repositories/gitolite-admin.git -type f -exec chmod 0640 {} \;
```

Add `apache` user to the `git` group:
```bash
$ usermod -a -G git apache
```

Enable syntax highlight:

```bash
$ sed -i "s|^exec highlight --force|exec highlight --force --inline-css|" /usr/local/lib/cgit/filters/syntax-highlighting.sh
```

Configure `gitolite` within `/var/vcs/git/.gitolite.rc` file:

```text
# Enable cgit supporting
'cgit',
```
After enabling cgit supporting additional options become available (see [documentation](https://git.zx2c4.com/cgit/tree/cgitrc.5.txt)).

Configure `cgit` within `/etc/cgitrc` file (see [documentation](https://git.zx2c4.com/cgit/tree/cgitrc.5.txt)):

```text
# Set cache root
cache-root=/var/cache/cgit

# Enable caching of up to 1000 output entries
cache-size=1000

# Specify some default clone urls using macro expansion
clone-url=git@<HOSTNAME>:$CGIT_REPO_URL

# Specify the css url
css=/cgit.css

# Show owner on index page
enable-index-owner=1

# Allow http transport git clone
enable-http-clone=0

# Show extra links for each repository on the index page
enable-index-links=1

# Enable ASCII art commit history graph on the log pages
enable-commit-graph=1

# Show number of affected files per commit on the log pages
enable-log-filecount=1

# Show number of added/removed lines per commit on the log pages
enable-log-linecount=1

# Sort branches by date
branch-sort=age

# Add a cgit favicon
favicon=/favicon.ico

# Use a custom logo
logo=/cgit.png

# Enable statistics per week, month and quarter
max-stats=quarter

# Set the title and heading of the repository index page
root-title=TEST

# Set a subheading for the repository index page
root-desc=Tracking TEST development

# Allow download of tar.gz, tar.bz2 and zip-files
snapshots=tar.gz tar.bz2 zip

##
## List of common mimetypes
##
mimetype.gif=image/gif
mimetype.html=text/html
mimetype.jpg=image/jpeg
mimetype.jpeg=image/jpeg
mimetype.pdf=application/pdf
mimetype.png=image/png
mimetype.svg=image/svg+xml

# Highlight source code with python pygments-based highlighter
source-filter=/usr/libexec/cgit/filters/syntax-highlighting.sh

# Format markdown, restructuredtext, manpages, text files, and html files
# through the right converters
about-filter=/usr/libexec/cgit/filters/about-formatting.sh

##
## Search for these files in the root of the default branch of repositories
## for coming up with the about page:
##
readme=:README.md
readme=:readme.md
readme=:README.mkd
readme=:readme.mkd
readme=:README.rst
readme=:readme.rst
readme=:README.html
readme=:readme.html
readme=:README.htm
readme=:readme.htm
readme=:README.txt
readme=:readme.txt
readme=:README
readme=:readme
readme=:INSTALL.md
readme=:install.md
readme=:INSTALL.mkd
readme=:install.mkd
readme=:INSTALL.rst
readme=:install.rst
readme=:INSTALL.html
readme=:install.html
readme=:INSTALL.htm
readme=:install.htm
readme=:INSTALL.txt
readme=:install.txt
readme=:INSTALL
readme=:install

##
## List of repositories
##

enable-git-config=1
scan-path=/var/vcs/git/repositories/
```
Replace `<HOSTNAME>` by appropriate name of host.

Create folder to store cache:
```bash
$ mkdir -p /var/cache/cgit
$ chmod 700 /var/cache/cgit
$ chown apache:apache /var/cache/cgit
```

Configre SELinux (optional):
```bash
$ setsebool -P git_cgi_enable_homedirs 1
$ semanage fcontext -a -t git_system_content_t "/var/vcs/git/projects.list" 
$ semanage fcontext -a -t git_system_content_t "/var/vcs/git/repositories(/.*)?"
$ restorecon -v -F -R /var/vcs/git
```

# Step 5. Configure virtual host on web server

Configure appropriate rights:
```bash
$ chown -R apache:apache /var/www/<hostname>/git
```

Add virtual host configuration:
```text
<VirtualHost *:80>
    ServerName git.<hostname>
    ServerAdmin admin@<hostname>
    DocumentRoot /var/www/<hostname>/git
    <Directory "/var/www/<hostname>/git">
        AddHandler cgi-script .cgi
        Options ExecCGI
        DirectoryIndex cgit.cgi
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```
Delete `cgit` config file from `/var/www/conf.d` folder that was created after `cgit` was installed.

Restart apache:
```bash
$ systemctl restart httpd
```

## Links

* [Documentation](https://gitolite.com/gitolite/gitolite.html)
* [Wildcard](https://gitolite.com/gitolite/wild.html)
